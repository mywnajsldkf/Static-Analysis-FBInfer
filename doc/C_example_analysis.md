
# Infer 예제 분석
예제 코드들은 infer git-hub에서 다운 받을 수 있다.
[ref) Infer git-hub](https://github.com/facebook/infer)

## 1. hello.c
	#include <stdlib.h>
           void test() {
           int* s = NULL;
           *s = 42;
           }
 hello.c의 코드는 위와 같다.
 위 코드를 infer를 통해 분석한 결과는 다음과 같다.


    hello_test.c:12: error: Null Dereference
    
    pointer `s` last assigned on line 11 could be null and is dereferenced at line 12, column 3.
    
    10. void test() {
    11. int* s = NULL;
    12. *s = 42;
        ^
    13. }
    Found 1 issue
	    Issue Type(ISSUED_TYPE_ID): #
    Null Dereference(NULL_DEREFERENCE): 1

위의 결과를 해석하자면,
hello.c 코드의 error issue는 null dereference이고 12번째 줄에서 발생하고 있다는 뜻이다. 

### 1-1. Issue type: Null dereference
Null dereference는 "null 포인터 역참조"로 해석할 수 있다.
null pointer는 어떠한 것도 가르키지 않는 포인터인데 여기에 접근을 하려고 할 때 오류가 발생하는 것이다.

hello.c 코드의 경우 
12번째 줄에서(*s =42;) 포인터를 통해서 값을 접근했는데
11번째 줄을 보면  int *s = null; 이라고 선언되었기 때문에 오류가 발생하는 것이다. 

### 1-2. Solution: Null dereference

 Null dereference issue는 보통 라이브러리 함수를 사용할 때 발생할 가능성이 높다.
 보통 라이브러리 함수는 유저가 의도한 값이 없거나 에러의 발생 위험 등의 이유로 null pointer를 반환하는 경우가 많기 때문이다.

코드를 다음과 같이 수정한 결과

    #include <stdlib.h>
    
    void test() {
      int a = 10;
      int* s = &a;
      *s = 42;
    }
오류가 발생하지 않음을 볼 수 있다.

![hello_c_edit_result](https://user-images.githubusercontent.com/91970346/183456235-9abf653f-af7c-4dcf-8f51-f26a00a772bb.png)

[ref) List of all issue types - nullptr_dereference](https://fbinfer.com/docs/all-issue-types#nullptr_dereference)

 

## 2. example.c
    #include <errno.h>
    ...
    #include <unistd.h> // 자세한 헤더파일은 git-hub 코드 참조
    
    struct Person {
      int age;
      int height;
      int weight;
    };
    int simple_null_pointer() {
      struct Person* max = 0;
      return max->age;
    }
    
    struct Person* Person_create(int age, int height, int weight) {
      struct Person* who = 0;
      return who;
    }
    
    int get_age(struct Person* who) { return who->age; }
    
    int null_pointer_interproc() {
      struct Person* joe = Person_create(32, 64, 140);
      return get_age(joe);
    }
    
    void fileNotClosed() {
      int fd = open("hi.txt", O_WRONLY | O_CREAT | O_TRUNC, 0600);
      if (fd != -1) {
        char buffer[256];
        // We can easily batch that by separating with space
        write(fd, buffer, strlen(buffer));
      }
    }
    
    void simple_leak() {
      int* p;
      p = (int*)malloc(sizeof(int));
    }
    
    void common_realloc_leak() {
      int *p, *q;
      p = (int*)malloc(sizeof(int));
      q = (int*)realloc(p, sizeof(int) * 42);
      // if realloc fails, then p becomes unreachable
      if (q != NULL)
        free(q);
    }

 example.c의 코드는 위와 같다.
 위 코드를 infer을 통해 분석한 결과는 다음과 같다.

    #0
    example_test.c:29: error: Null Dereference
      pointer `max` last assigned on line 28 could be null and is dereferenced at line 29, column 10.
      27. int simple_null_pointer() {
      28.   struct Person* max = 0;
      29.   return max->age;
                   ^
      30. }
      31.
    
    #1
    example_test.c:41: error: Null Dereference
      pointer `joe` last assigned on line 40 could be null and is dereferenced by call to `get_age()` at line 41, column 10.
      39. int null_pointer_interproc() {
      40.   struct Person* joe = Person_create(32, 64, 140);
      41.   return get_age(joe);
                   ^
      42. }
      43.
    
    #2
    example_test.c:49: error: Resource Leak
      resource acquired by call to `open()` at line 45, column 12 is not released after line 49, column 5.
      47.     char buffer[256];
      48.     // We can easily batch that by separating with space
      49.     write(fd, buffer, strlen(buffer));
              ^
      50.   }
      51. }
    
    #3
    example_test.c:55: error: Dead Store
      The value written to `&p` (type `int*`) is never used.
      53. void simple_leak() {
      54.   int* p;
      55.   p = (int*)malloc(sizeof(int));
            ^
      56. }
      57.
    
    Found 4 issues
              Issue Type(ISSUED_TYPE_ID): #
      Null Dereference(NULL_DEREFERENCE): 2
            Resource Leak(RESOURCE_LEAK): 1
                  Dead Store(DEAD_STORE): 1
위의 결과를 해석하자면,
example.c 코드의 error issue는 null dereference 2개, resource leak 1개, dead store 1개가
각각 29,41번째 줄, 49번째 줄 55번째 줄에서 발생한다는 것이다.

### 2-1. Issue type: Null dereference
(#0의 오류는 hello.c와 동일한 에러이기 때문에 생략)
#1의 경우는 3개의 함수를 거쳐서 발생한 오류이다.

    // 1번 함수
    struct Person* Person_create(int age, int height, int weight) {
          struct Person* who = 0;
          return who;
        }
        
    // 2번 함수
    int get_age(struct Person* who) { return who->age; }
    
    // 3번 함수
    int null_pointer_interproc() {
      struct Person* joe = Person_create(32, 64, 140);
      return get_age(joe);
    }

위 코드를 보면 3번 함수에서 1번 함수를 호출하였는데 null ptr를 반환함을 볼 수 있다.
따라서 2번 함수에서는 1번 함수의 who를 반환을 받기 때문에 null ptr을 참조하게 되고
3번 함수에서 2번 함수를 통해서 받은 return값은 null ptr이 된다.
    
### 2-2. Issue solution: Null dereference

    struct Person* Person_create(int age, int height, int weight) {
      struct Person jungwoo; // null ptr 제거
      struct Person* who = &jungwoo;
      return who;
    }
    
    int get_age(struct Person* who) { return who->age; }
    
    int null_pointer_interproc() {
      struct Person* joe = Person_create(32, 64, 140);
      return get_age(joe);
    }
따라서 위와 같이 수정했더니 오류가 사라짐을 볼 수 있다.

### 2-3. Issue type: resource leak


resouce leak은 프로그램이 실행될 때 불필요한 메모리를 계속 점유하고 있을 때 발생하는 issue이다.
다음은 example.c 코드의 reosource leak 발생 부분이다.

     

     47. char buffer[256];
     48. // We can easily batch that by separating with space
     49. write(fd, buffer, strlen(buffer));
         ^
     50.}

write 함수는 file을 열어 문장을 삽입하는 함수이다. 여기에서 주목해야 할 부분은 strlen(buffer)이다.
strlen(buffer)의 의미는  buffer의 크기만큼 공간을 할당하겠다는 의미이다. 여기서 주목해야 할 점은 유저가 항상 256byte의 크기만큼 문장을 작성하지 않는다는 점이다.
따라서, 위 함수를 통해서 **불필요한 공간을 할당한다는 것**이 rsource leak을 발생시킨다.

### 2-4. Solution: resource leak
해결방법은 유저가 문장을 작성하는 만큼 공간을 할당해주면 된다. 
우리는 이러한 방법을 **동적할당**을 통해서 구현할 수 있다.
c에서는 malloc를 c++에서는 malloc와 new를 통해서 구현할 수 있다.

    void fileNotClosed() {
      int fd = open("hi.txt", O_WRONLY | O_CREAT | O_TRUNC, 0600);
      if (fd != -1) {
        char* str; scanf("%s",&str);
        char* buffer;
        buffer = (char*)malloc(strlen(str)*sizeof(char));
        // We can easily batch that by separating with space
        write(fd, str, sizeof(buffer));
        free(buffer);
      }
    }
   
   위와 같이 코드를 수정했더니 오류가 발생하지 않음을 볼 수 있다.

### 2-5. Issue type: Dead store
dead store은 malloc나 new를 통해서 메모리를 할당했지만 free/delete 과정을 거치지 않아 메모리가 회수되지 않을 때 발생하는 issue이다. 메모리를 회수하지 않을 시 컴퓨터에는 불필요한 용량을 차지하게 되고 이는 성능저하로 이어진다.
다음은 example.c 코드에서 issue가 발생한 부분이다.

      53. void simple_leak() {
      54.   int* p;
      55.   p = (int*)malloc(sizeof(int));
            ^
      56. }
55번째 줄에서 malloc를 사용하였지만 이후에 free 과정을 거치지 않는 것이 보인다.

### 2-6. Soultion: Dead store
해결방법은 null dereference와 동일하다. 유저가 free를 진행하였는지 확인하는 방법이다.

    void simple_leak() {
      int* p;
      p = (int*)malloc(sizeof(int));
      free(p); // 할당해제
    }
위와 같이 free를 시켜주니 오류가 발생하지 않았다.

![example_c_edit_result](https://user-images.githubusercontent.com/91970346/183456544-3d63fd3d-4e25-41b4-96ba-7795ddb8171b.png)


## 3. Reference
### 3-1. Pulse 
pulse의 정의는 다음과 같다. "Pulse is an interprocedural memory safety analysis" interprocedural은 절차적인이라는 뜻이다. 즉, pulse는 메모리가 효율적으로 사용이 되는지 분석하는 절차이다. infer에서는 pulse과정을 `--pulse`라는 커맨드로 기능을 제공하고 있다. 
다음은 `--pulse`를 사용하는 예제이다.

    class Person {
        Person emergencyContact;
        String address;
    
        Person getEmergencyContact() {
            return this.emergencyContact;
        }
    }
    
    class Registry {
        void create() {
            Person p = new Person();
            Person c = p.getEmergencyContact();
            // Null dereference here
            System.out.println(c.address);
        }
    
        void printContact(Person p) {
            // No null dereference, as we don't know anything about `p`
            System.out.println(p.getEmergencyContact().address);
        }
    }
    
  위 코드를  `infer run --pulse -- javac Test.java`커맨드를 통해서 분석하면 된다.
  ### 3-2. Dead store causede by only C
  동적할당이라는 개념은 C와 C++에만 존재한다. java나 python과 같은 언어들은 기본적으로 동적할당 기능을 내재하고 있기때문이다.
