# Purity

pure (side-effect-free) 함수를 찾기 위해서 사용한다. 반대된 표현은 “impurity”라고 한다.

- `--purity` 를 사용한다.



## pure, side-effect-free

*Java 기준으로 찾아봄*

[Java Functional Programming](https://jenkov.com/tutorials/java-functional-programming/index.html)

Functional Programming(함수형 프로그래밍)은 자바에서 이전부터 쓰인 개념은 아니다. 함수형 프로그래밍은 명령형 프로그래밍과 비교하여 설명된다. 명령형 프로그래밍은 C, C++, Java, C# 처럼 무엇(What)을 할 것인지 나타내기보다 어떻게(How)할 것인지 설명하는 방식이다. 함수형 프로그래밍은 모든 것을 순수 함수로 나누어 문제를 해결하는 방법으로 가독성을 높이고 유지보수를 쉽게 할 수 있다는 장점이 있다.

**명령형 프로그래밍과 함수형 프로그래밍 비교**

- **명령형 프로그래밍**

  1~10까지의 값을 i에 할당하여 출력한다.

  ```bash
  for(int i = 1; i < 10; i++){
  	System.out.println(i);
  }
  ```

- **함수형 프로그래밍**

  대입문을 사용하지 않고, 작은 문제를 해결하기 위한 함수를 작성한다.

  ```bash
  process(10, print(num));
  ```



### Functional Programming Basics

함수형 프로그래밍 주요 컨셉

- Functions as first class objects
- Pure functions
- Higher order functions

Pure functional programming

- No state
- No side effects
- Immutable variables
- Favour recusion over looping



### Functions as First Class Objects

함수형 프로그래밍 패러다임에서 함수는 일급객체이다. 문자열, 맵 또는 다른 개체 참조처럼 함수 인스턴스의 “인스턴스"를 생성할 수 있으며 다른 함수에 매개변수로 전달되기도 한다.



### Pure Functions

- side effects가 없어야 한다.
- 반환값이 함수에 지난 입력값 파라미터에 의존한다.

> **Pure function(method) in Java**

```java
public class ObjectWithPureFunction{
	public int sum(int a, int b) {
		return a + b;
	}
}
```

이 함수는 Pure 함수이다. 합을 의미하는 a + b가 입력값 a,b에 영향을 받기 때문이다.

또한 sum() 은 side effects가 없다. 함수 밖에 있는 다른 값들을 건들이지 않기 때문이다.

> **non-pure functions**

```java
public class ObjectWithNonPureFunction{
	private int value = 0;

	public int add(int nextValue) {
		this.value += nextValue;
		return this.value;
	}
}
```

add() 함수는 memberVariable을 사용하기 때문에 value를 반환한다. 또 value 값을 바꾸기 때문에 side effects가 있다.



### Higher Order Functions

아래 설명 상황 중 하나라도 만족하면 고차함수라고 할 수 있다.

- 파라미터로 하나 혹은 그 이상의 함수를 갖는 함수
- 결과로 다른 함수를 반환하는 함수

다음 코드는 하나 이상의 람다 식을 매개 변수로 사용하고 다른 람다식을 반환한다.

```java
public class HigherOrderFunctionClass {
	public <T> IFactory<T> createFactory(IProducer<T> producer, IConfigurator<T> configurator) {
		return () -> {
			T instance = producer.produce();
			Configurator.configure(instance);
			return instance;
		}
	}
}	
```

createFactor()는 lambda 표현식을 반환하기 때문에 Higher Order Functions이라 할 수 있다.

자세히보면

```java
public interface IFactory<T> {
	T create();
}
```

```java
public interface IProducer<T> {
	T produce();
}
```

```java
public interface IConfigurator<T> {
	void configure(T t);
}
```

모든 인터페이스가 함수형 인터페이스이다. 따라서 람다 표현식에 의해 실행될 수 있다. createFactory() 메서드는 고차함수이다.



### No State

“no state”는 함수의 외부 상태를 의미한다. 함수는 내부에서 일시적인 상태를 잠시 갖고 있는 내부 변수가 있다. 하지만 함수는 함수가 속한 클래스 또는 객체의 멤버 변수를 참조할 수 없다.

외부 상태를 사용하지 않는 함수의 예시이다.

```java
public class Calculator {
	public int sum(int a, int b) {
		return a + b;
	}
} 
```

반대로 외부 변수를 사용한 예시이다. no state 규칙을 지키지 않았다.

```java
public class Calculator {
	private int initVal = 5;
	public int sum(int a) {
		return initVal + a;
	}
} 
```



### No Side Effects

no side effects의 의미는 함수 밖에 있는 것을 바꿀 수 없다는 것이다. 함수 밖에 있는 것을 수정하는 것은 side effect로 여겨진다.

함수 외부 상태(Side Effect)는 클래스 또는 객체의 멤버 변수, 함수의 매개 변수 내용의 멤버 변수를 나타내거나 파일 시스템이나 데이터 베이스같은 것이다.



### Immutable Variable

side effects를 줄일 수 있는 방법이다. (Immutable이 의미하는 것 → 불변(변하지 않음), 값을 바꿀 수 없기 때문에 side effects를 줄이게 된다.)



### Favour Recursion Over Looping

loop에서 재귀형을 사용하는 것이다. 재귀는 loop를 돌면서 함수를 사용하기 때문에 코드가 더 함수형을 띄게 된다.

다른 루프 방법으로는 Strema API가 있다.



### Functional Interfaces

자바에서 함수형 인터페이스는 추상 메서드만을 갖는 인터페이스이다. 추상 메서드는 실행되지 않는 유일한 메서드를 의미한다. 하나의 인터페이스는 여러 메서드를 지닌다. 인터페이스는 기본 방법과 정적 방법과 같은 여러 방법을 가질 수 있지만, 인터페이스가 구현되지 않는다면 기능적인 인터페이스로 간주된다.

함수형 인터페이스 예시이다.

```java
public interface MyInterface {
	public void run();
}
```

default method와 static method를 가진 함수형 인터페이스 예시이다.

```java
public interface MyInterface2 {
	public void run();
	
	public default void doIt() {
		System.out.println("doing it");
	}

	public static void doItStatically() {
		System.out.println("doing it statically");
	}
}
```

run()만 구현(추상)되지 않았기 때문에 기능적인 인터페이스이다. 만약 구현되지 않은 메서드가 많다면 인터페이스는 더 이상 기능적인 인터페이스가 아니므로 자바 람다식으로 구현될 수 없다.


pure 함수를 파악할 수 있는 실험적인 절차 분석이다. 각각의 함수는 purity 분석이 계속해서 purity 함수 확인 뿐만 아니라 전역 변수를 수정하거나 파라미터를 수정하여 글로벌 변수를 수정한다.

만약 함수가 pure한 상태(global 변수 또는 파라미터를 수정하거나 알수없는 함수들을 호출하는지)라면, `PURE_FUNCTION` 이슈를 발생시킨다.



# Weaknesses

purity 분석에는 두 가지 문제점이 있다.

- 어떤 파라미터가 수정되었는지 찾기 위해서 정확한 값 찾기 위해 분석이 필요하다.
- 수정된 argument를 찾는 것이 쉽지 않다.