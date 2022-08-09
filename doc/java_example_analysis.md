# Java 예제 분석

Infer로 Java로 작성한 예제를 분석할 수 있습니다. 예제 코드는 [Infer github](https://github.com/facebook/infer/tree/main/examples/java_hello) 언어에서 다운받을 수 있습니다.

- [Hello.java](https://github.com/facebook/infer/blob/main/examples/java_hello/Hello.java)
- [Pointers.java](https://github.com/facebook/infer/blob/main/examples/java_hello/Pointers.java)

- [Resources.java](https://github.com/facebook/infer/blob/main/examples/java_hello/Resources.java)



## 1.**예제 실행**

```
infer -- javac Pointers.java Resources.java Hello.java
```

분석 결과는 다음과 같다.

```
Capturing in javac mode...
Found 3 source files to analyze in /Users/mywnajsldkf/Desktop/ongoing/fbinfer/java/java_hello/infer-out
3/3 [############################################################] 100% 118ms

Hello.java:28: error: Null Dereference
  object `a` last assigned on line 26 could be null and is dereferenced at line 28.
  26.       Pointers.A a = Pointers.mayReturnNull(rng.nextInt());
  27.       // FIXME: should check for null before calling method()
  28. >     a.method();
  29.     }
  30.   

Hello.java:38: error: Resource Leak
  resource of type `java.io.FileOutputStream` acquired by call to `allocateResource()` at line 32 is not released after line 38.
  36.   
  37.       try {
  38. >       stream.write(12);
  39.       } finally {
  40.         // FIXME: should close the stream

Hello.java:63: error: Resource Leak
  resource of type `java.io.FileOutputStream` acquired to `fos` by call to `FileOutputStream(...)` at line 53 is not released after line 63.
**Note**: potential exception at line 57
  61.         }
  62.       }
  63. >   }
  64.   }


Found 3 issues
          Issue Type(ISSUED_TYPE_ID): #
        Resource Leak(RESOURCE_LEAK): 2
  Null Dereference(NULL_DEREFERENCE): 1
```

분석 결과는 총 3개의 이슈가 발생했고 2개는 Resource Leak, 1개는 Null Dereference 임을 확인할 수 있다. 



Hello.java 코드를 확인해보면 `Pointers.A`와 `Resources.allocateResource`처럼 Pointer와 Resources 클래스를 호출하는 것을 확인할 수 있다. 따라서 함께 컴파일하는 것이 필요하다. 

만약 `infer -- javac Hello.java` 처럼 하나의 파일만 컴파일 할 경우 다음과 같이 Pointer와 Resources 클래스를 사용하는 함수의 취약점을 파악하지 못하는 것을 확인할 수 있었다.

```
Capturing in javac mode...
Found 1 source file to analyze in /Users/mywnajsldkf/Desktop/ongoing/fbinfer/java/java_hello/infer-out
1/1 [############################################################] 100% 110ms

Hello.java:63: error: Resource Leak
  resource of type `java.io.FileOutputStream` acquired to `fos` by call to `FileOutputStream(...)` at line 53 is not released after line 63.
**Note**: potential exception at line 57
  61.         }
  62.       }
  63. >   }
  64.   }


Found 1 issue
    Issue Type(ISSUED_TYPE_ID): #
  Resource Leak(RESOURCE_LEAK): 1
```



## 2. Issue Type: Null Dereference

### 2-1. 이슈분석

```
Hello.java:28: error: Null Dereference
  object `a` last assigned on line 26 could be null and is dereferenced at line 28.
```

첫번째는 생성된 a가 null 참조가 될 수 있다는 것을 감지했다. 

> 코드

```java
void mayCauseNPE() {
   Random rng = new Random();
   Pointers.A a = Pointers.mayReturnNull(rng.nextInt());
   // FIXME: should check for null before calling method()
    a.method();
}
```

26번째줄을 보면 Pointeres Class에서 `mayReturnNull`이라는 메서드를 호출하고 있다.

Pointers Class를 살펴보자.

```java
package hello;

public class Pointers {

  public static class A {
    public void method() {}
  }

  public static A mayReturnNull(int i) {
    if (i > 0) {
      return new A();
    }
    return null;
  }
}
```

mayReturnNull 함수를 보면 실제로 i가 0이하일 때 null을 반환한다. 이에 따라서 null 참조가 발생하는 것이다. 



### 2-2. 문제 해결

nullPointerException은 객체 생성 후 인스턴스를 생성하지 않은 상태에서 NULL 오브젝트를 사용했을 때 발생한다. 

해결 방법은 다음과 같이 다양하다. 

- Try-catch로 nullPointerException 예외 처리하고 null 인지 비교한다.
- Null 여부를 비교하고 처리한다.
- 문자열 비교 시 equals 메서드를 사용해서 null을 확인하다. (==과 equals의 차이는 각각 주소값 비교, 값 비교이다.)
- toString() 대신 valueOf를 사용하여 null을 감지하면 null이라는 문자를 출력하게 해라.



여기서는 if문을 사용해서 Null 여부를 비교하고 처리하겠다.

```java
void mayCauseNPE() {
  Random rng = new Random();
  Pointers.A a = Pointers.mayReturnNull(rng.nextInt());
  if (a == null) {
    System.out.println("양수의 값을 입력하세요.");
  }
  else {
    a.method();
  }
}
```

이렇게 null을 감지했을 때 에러 메시지를 작성하도록 하면 오류가 발생하지 않는다.



## 3. Issue Type: Resource Leak 

### 3-1. 이슈 분석

```
Hello.java:43: error: Resource Leak
  resource of type `java.io.FileOutputStream` acquired by call to `allocateResource()` at line 37 is not released after line 43.
  41.   
  42.       try {
  43. >       stream.write(12);
  44.       } finally {
  45.         // FIXME: should close the stream
```

다음은 메모리 누수가 발생했다는 것이다. 

메모리 누수는 더 이상 사용되지 않은 객체들이 가지비 컬렉터에 의해 회수되지 않고 계속 누적이 되는 현상을 말한다. 가바지 컬렉션은 동적으로 할당했던 메모리 영역 중에 필요없게 된 영역을 해제하는 것을 말한다.

> 코드

```java
void mayLeakResource() throws IOException {
    OutputStream stream = Resources.allocateResource();
    if (stream == null) {
      return;
    }

    try {
      stream.write(12);
    } finally {
      // FIXME: should close the stream
    }
  }
```

힌트를 보면 알 수 있겠지만 stream을 사용하고 닫아주지 않았다.

Java에서 Stream을 데이터의 흐름이라고 하여 파일 입출력을 도와준다. Java에서 Collectoin을 통해 생성한 Stream은 close를 호출하여 닫을 필요가 없지만 I/O 리소스(ex. Files.lines())를 사용한다면 명시적으로 close()를 호출해야한다. 

실제로 [Stream API doc](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)에서도 해당 내용을 확인할 수 있다.

그럼 왜 같은 Stream이라도 I/O 리소스를 사용했다면 `close()` 를 호출할까? [이유](https://stackoverflow.com/questions/34072035/why-is-files-lines-and-similar-streams-not-automatically-closed)는 의도적으로 설계를 하였다고 한다. I/O 리소스를 받아서 사용했을 때 자동으로 리소스가 닫히지 않아서 명시적으로 `close()` 를  사용하게 설계를 하였다.



### 3-2. 문제 해결

위에서 명시한 것처럼 자바 공식문서에서 제시한 것은 close()를 호출하는 것이다. 

```java
void mayLeakResource() throws IOException {
    OutputStream stream = Resources.allocateResource();
    if (stream == null) {
      return;
    }

    try {
      stream.write(12);
    } finally {
      close();
    }
  }
```

이렇게 `close()` 메서드를 호출하여 리소스를 닫아주면 에러가 발생하지 않는다.



## 4. Issue Type: Resource Leak  - Multiple Resources Bug

### 4-1. 이슈 분석

```
Hello.java:63: error: Resource Leak
  resource of type `java.io.FileOutputStream` acquired to `fos` by call to `FileOutputStream(...)` at line 53 is not released after line 63.
**Note**: potential exception at line 57
  61.         }
  62.       }
  63. >   }
  64.   }
```

다음은 여러개의 리소스가 사용되었을 때 제대로 close되지 않아서 발생하는 문제이다. 

> 코드

```java
  /**
   * This method should be rewritten with nested try { ... } finally { ... } statements, or the
   * possible exception raised by fis.close() should be swallowed.
   */
  void twoResources() throws IOException {
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
      fis = new FileInputStream(new File("whatever.txt"));
      fos = new FileOutputStream(new File("everwhat.txt"));
      fos.write(fis.read());
    } finally {
      if (fis != null) {
        fis.close();
      }
      if (fos != null) {
        fos.close();
      }
    }
  }
```

이 코드에서는 fos.close()까지 코드가 가지 않아 리소스가 닫히지 않기 때문에 Resource Leak 오류가 발생한다. 

힌트를 보면 알 수 있겠지만 try, finally를 중첩해서 사용하거나 exception을 던져서 close를 닫아주라고 하였다. 



### 4-2. 문제 해결

1. 중첩된 try, finally 사용한다.

   ```java
   void twoResources() throws IOException {
       FileInputStream fis = null;
       try {
         FileOutputStream fos = new FileOutputStream(new File("everwhat.txt"));
         try {
           fos.write(fis.read());
         } finally {
           fos.close();
         }
       } finally {
         fis.close();
       }
     }
   ```

   

2. Exception Swallowing을 사용한다.

   ```java
   static void foo() throws IOException {
     FileInputStream fis = null;
     FileOutputStream fos = null;
     try {
       fis = new FileInputStream(new File("whatever.txt"));
       fos = new FileOutputStream(new File("everwhat.txt"));
       fos.write(fis.read());
     } finally {
       try {
         if (fis!=null) fis.close();
       } catch (Exception e) {};  // Exception swallowing
       if (fos!=null) fos.close();
     }
   }
   ```

   Exception을 발생시켜 예외가 있을 경우, 건너 띄게 한다. 

   Exception Swallowing은 error나 exception을 발견했을 때, 소프트웨어 에러를 로그하거나 보고하지 않고 계속 실행시키는 것을 말한다. 이 경우에는 fos.close()까지 진행하도록 수정했다. 하지만 몇몇 개발자들은 이 방법을 추천하지 않는다고 한다. ([링크](https://github.com/google/guava/issues/1118))