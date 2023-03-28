---
category: Tech
tags: [Tech]
title: "Static VS Singleton 패턴"
date:   2023-03-05 17:15:00 
lastmod : 2023-03-05 17:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

최근 모기업 과제전형을 진행하면서, `static 클래스를 사용하는 것`과 `Singleton 패턴을 사용하는 것`의 차이점에 대해 의문이 들었다.

두 방식은 유사한 것 같으면서도, 목적과 사용방법이 다르다.

이번 포스팅에서는 두 프로그래밍 방식에 대해 알아보도록 하겠다.

<br/><br/>

## static 과 Singleton 패턴이란?

먼저 static과 Singleton 패턴이 무엇인지 간략히 알아보자.

<br/>

### static

`static`은 일종의 예약어이다.

`static`을 사용해, 클래스 변수나 클래스 메서드를 만들 수 있다.

클래스 변수와 클래스 메서드는 **인스턴스 생성 없이 클래스명으로 접근** 할 수 있다.

> 사용 예시) `MyClass.staticField` , `MyClass.staticMethod(...)`

<br/>

### Singleton 패턴

Singleton 패턴은 디자인 패턴 중 하나이다.

애플리케이션 라이프사이클 동안, 특정 클래스의 **인스턴스를 오직 하나만 사용**하도록 하는 패턴이다.

보통 아래와 같이 구현한다.

```java
public class MyClass {
	private static final MyClass instance = new MyClass();

	//생성자를 숨겨둬야 한다.
	private MyClass() {}

	public static MyClass getInstance() {
		return instance;
	}
}
```

<br/>

그럼 이제 본격적으로 둘의 차이점에 대해 알아보자.

<br/><br/>

## 런타임 다형성 처리 가능 여부

### 컴파일 타임 다형성과 런타임 다형성

자바에서 다형성을 처리하는 방법에는 두 가지가 있다.

- **컴파일 타임 다형성 (Compile-time Polymorphism)**
    - 컴파일 시점에 결정되는 다형성을 뜻한다.
    - **static 메서드를 오버로딩**하면 컴파일 시점에 다형성이 처리된다.
- **런타임 다형성 (Runtime Polymorphism)**
    - 런타임 시점에 결정되는 다형성을 뜻한다.
    - 오버라이딩을 통해, **런타임 시점에 재정의한 메서드를 호출**한다.

static 을 사용하면 **오직 컴파일 타임 다형성만 활용**할 수 있고, Singleton 패턴을 사용하면 **런타임 다형성까지 사용**할 수 있다.

**즉 static 은 오버라이딩이 불가능하다.**

<br/>

### static 다형성 예시 코드

```java
//부모 클래스
public class SuperClass {
	public static String echo(String data) {
		return "SuperClass : " + data;
	}
}

//자식 클래스
public class SubClass extends SuperClass {
	//static 메서드는 오버라이딩이 되지 않는다.
	public static String echo(String data) {
		return "SubClass : " + data;
	}
}

//테스트
@Test
public void test() {
	//GIVEN
	SuperClass instance = new SubClass();

	//WHEN
	String result = instance.echo("myData");

	//THEN - 오버라이딩 실패
	assertNotEquals("SubClass : myData", result); //Succeed
	assertEquals("SuperClass : myData", result); //Succeed
}
```

<br/>

### Singleton 패턴 예시 코드

```java
//부모 클래스
public class SuperClass {
	public String echo(String data) {
		return "SuperClass : " + data;
	}
}

//자식 싱글톤 클래스
public class SubClass extends SuperClass {
	private static final SubClass instance = new SubClass();

	private SubClass() {}

	public static SubClass getInstance() {
		return instance;
	}

	@Override
	public String echo(String data) {
		return "SubClass : " + data;
	}
}

//테스트
@Test
public void test() {
	//GIVEN
	SuperClass superInstance = new SuperClass();
	SuperClass subInstance = SubClass.getInstance();

	//WHEN
	String superResult = superInstance.echo("myData");
	String subResult = subInstance.echo("myData");

	//THEN - 오버라이딩 성공
	assertEquals("SuperClass : myData", superResult); //Succeed
	assertEquals("SubClass : myData", subResult); //Succeed
}
```

<br/>

### 인터페이스 사용

또한 Singleton 패턴을 사용하면, **인터페이스를 사용해 구현**할 수 있다.

인터페이스에 static 메서드를 선언할 수는 있지만, **재정의는 불가능**하다.

```java
//인터페이스
public interface MyInterface {
	//Java8부터 인터페이스에 static 메서드 지원
  static String myStaticMethod() {
    return "My Interface";
  }

  String myMethod();
}

//인터페이스 구현 싱글톤 클래스
public class MyClass implements MyInterface {
  private static final MyClass instance = new MyClass();

  private MyClass() {}

  //재정의 불가능
  public static String myStaticMethod() {
    return "My Class";
  }

  @Override
  public String myMethod() {
    return "Override Method";
  }

  public static MyClass getInstance() {
    return instance;
  }
}

//테스트
@Test
public void test() {
	//GIVEN
  MyInterface instance = MyClass.getInstance();
	
	//WHEN
	String myMethodResult = instance.myMethod();
	String myInterfaceStaticMethodResult = MyInterface.myStaticMethod();
	String myClassMethodResult = MyClass.myStaticMethod();
	//instance.myStaticMethod(); 호출 불가능

	//THEN
	assertEquals("Override Method", myMethodResult); //Succeed
	assertEquals("My Interface", myInterfaceStaticMethodResult); //Succeed
	assertEquals("My Class", myClassMethodResult); //Succeed
}
```

<br/><br/>

## 인스턴스 Argument 처리 가능 여부

Singleton 패턴은 인스턴스를 사용하기 때문에, **인스턴스를 특정 메서드의 파라미터에 전달**할 수 있다.

<br/>

### Singleton 인스턴스를 메서드에 전달하는 예시 코드
데코레이터 패턴을 사용하는 예시이다.
> 데코레이터 패턴은 [이 포스팅](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_Proxy_Decorator_Pattern_Concept)을 참고하자!

```java
public interface MyInterface {
	public void job()
}

public class MyClass implements MyInterface {
  private static final MyClass instance = new MyClass();

  private MyClass() {}

  public static MyClass getInstance() {
    return instance;
  }

	@Override
	public void job() {
		System.out.println("Do Same Jobs");
	}
}

public class Decorator {
	public void decorate(MyInterface instance) {
		System.out.println("Before invoke");
		instance.job();
		System.out.println("After invoke");
	}
}

@Test
public void test() {
	//GIVEN
	Decorator decorator = new Decorator();

	//WHEN
	decorator.decorate(MyClass.getInstance());

	/* [출력 결과]
	 * Before invoke
	 * Do Same Jobs
	 * After invoke
	 */
}
```

<br/><br/>

## 로딩 메커니즘과 메모리 할당

### Singleton 패턴은…

Singleton 패턴 사용 시, **인스턴스는 JVM 메모리의 Heap 영역에 할당**된다.

> 원래 Heap 영역이 동적으로 생성된 인스턴스가 존재하는 공간이므로

따라서 만약 거대한 로직을 싱글톤 객체로 관리한다면, **다소 오버헤드**가 커질 수 있다.

보통 Heap 영역에 접근하는 것보다, Method 영역에 접근하는 것이 더 빠르다.

또한 Heap 영역에서 **GC가 동작**하기 때문에, 거대한 인스턴스를 대상으로 GC 동작시 오버헤드가 커질 수 있다.

<br/>

### Static 클래스는…

반면에 static 클래스는 **컴파일 시점에 Method 영역에 할당**되기 떄문에, 이 경우 더 유리하다.

static 클래스는 객체 초기화가 필요없기 때문에, 조금 더 효율적이다.

<br/><br/>

## 그럼 어떤 것을 사용해야 할까?

### static 클래스를 사용하는 것이 권장되는 경우

- 런타임 다형성과 같이 상속 등의 객체지향적 솔루션이 필요하지 않은 경우
- 내부 상태값을 갖지 않는 정적 유틸리티 메서드

> 가장 대표적인 예로, Math 클래스가 있다.

<br/>

### Singleton 패턴을 사용하는 것이 권장되는 경우

- 런타임 다형성과 같이 상속 등을 사용해야 하는 경우
- 하나의 인스턴스만 있어도 되는 경우 (인스턴스를 재사용할 수 있는 경우)

<br/><br/>

## 마치며...

이렇게 static 과 Singleton 패턴에 대해 비교해봤다.

이 둘의 목적과 사용 방법이 다르므로, 상황에 맞게 적절한 방식을 사용하자!

<br/><br/>

## References

- [https://www.baeldung.com/java-static-class-vs-singleton](https://www.baeldung.com/java-static-class-vs-singleton)
- [Chat-GPT](https://chat.openai.com/chat)