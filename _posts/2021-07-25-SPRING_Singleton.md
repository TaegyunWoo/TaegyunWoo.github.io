---
category: Spring-Core
tags: [스프링, 핵심원리, 싱글톤, 스프링컨테이너]
title: "[스프링 - 핵심원리] 싱글톤과 스프링 컨테이너"
date:   2021-07-25 00:30:00 
lastmod : 2021-07-25 00:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 웹 애플리케이션의 기본 동작

## 웹 애플리케이션의 특징

- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.

![싱글톤 적용 X](/assets/img/2021-07-25-SPRING_Singleton/Untitled%2010.png)

- 각 클라이언트와 1대1로 대응되어 총 3개의 객체가 생성된다.
    - 각 고객마다 객체를 생성하게 된다면, **메모리 낭비가 발생** 한다.
- **따라서 해당 객체가 딱 1개만 생성되고, 클라이언트들이 해당 객체를 공유하도록 설계해야한다.**
    - 이것을 **싱글톤 패턴** 이라고 한다.

<br><br>

## 싱글톤이 적용되지 않은 컨테이너 테스트코드

> `AppConfig` , `MemberService` 관련 내용은 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고

```java
import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

public class SingletonTest {
	
	@Test
	@DisplayName("스프링 없는 순수한 DI 컨테이너")
	void pureContainer() {
		AppConfig appConfig = new AppConfig();
		
		//1. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService1 = appConfig.memberService();

		//2. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService2 = appConfig.memberService();
		
		//참조값이 다른 것을 확인
		System.out.println("memberService1 = " + memberService1);
		System.out.println("memberService2 = " + memberService2);

		//memberService1 != memberService2
		// 서로 다른 객체이다.
		assertThat(memberService1).isNotSameAs(memberService2);
	}

}
```

<br><br><br>

# 싱글톤 패턴

## 개요

### 싱글톤 패턴이란?

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.

<br>

### 싱글톤 패턴 적용하기

- 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
- 따라서 생성자를 `private` 으로 설정하여 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

<br><br>

## 싱글톤이 적용된 컨테이너 테스트코드

### SingletonService 클래스

```java
public class SingletonService {
	
	//1. static 영역에 객체를 딱 1개만 생성해둔다.
	private static final SingletonService instance = new SingletonService();
	
	//2. public으로 열어서 객체 인스터스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
	public static SingletonService getInstance() {
		return instance;
	}

	//3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
	private SingletonService() {}

	public void logic() {
		System.out.println("싱글톤 객체 로직 호출");
	}

}
```

1. static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
2. 이 객체 인스턴스가 필요하면 오직 getInstance() 메서드를 통해서만 조회할 수 있다. 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.
3. 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 `private` 으로 막아둔다.

<br>

### 테스트 코드

```java
import hello.core.AppConfig;
import hello.core.SingletonService;
import hello.core.member.MemberService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

public class SingletonTest {
	
	// 싱글톤이 적용되지 않은 기존의 테스트코드
	@Test
	@DisplayName("스프링 없는 순수한 DI 컨테이너")
	void pureContainer() {
		AppConfig appConfig = new AppConfig();
		
		//1. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService1 = appConfig.memberService();

		//2. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService2 = appConfig.memberService();
		
		//참조값이 다른 것을 확인
		System.out.println("memberService1 = " + memberService1);
		System.out.println("memberService2 = " + memberService2);

		//memberService1 != memberService2
		// 서로 다른 객체이다.
		assertThat(memberService1).isNotSameAs(memberService2);
	}

	//싱글톤이 적용된 싱글톤패턴 테스트코드
	@Test
	@DisplayName("싱글톤 패턴을 적용한 객체 사용")
	public void singletonServiceTest() {
		//private으로 생성자를 막아두었다. 컴파일 오류가 발생한다.
		//new SingletonService();
		
		//1. 조회: 호출할 때 마다 같은 객체를 반환
    SingletonService singletonService1 = SingletonService.getInstance();
		
		//2. 조회: 호출할 때 마다 같은 객체를 반환
		SingletonService singletonService2 = SingletonService.getInstance();
		
		//참조값이 같은 것을 확인
		System.out.println("singletonService1 = " + singletonService1);
		System.out.println("singletonService2 = " + singletonService2);
		
		// singletonService1 == singletonService2
		// 서로 같은 객체이다.
		assertThat(singletonService1).isSameAs(singletonService2);
	}
}
```

<br><br>

## 싱글톤 패턴 문제점

- 싱글톤 패턴을 구현하는 코드가 많이 들어간다.
- **의존관계상 클라이언트(** `singletonServiceTest` 테스트 메서드 **)가 구체 클래스(** `SingletonService` 클래스 **)에 의존한다.
⇒ DIP를 위반한다.**
    - `구체클래스.getInstance()` 를 사용해야하기 때문이다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- `private` 생성자로 인해 자식 클래스를 만들기 어렵다.
- 즉, 결론적으로 **유연성이 떨어진다**.

<br><br><br>

# 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하고, 객체 인스턴스를 싱글톤으로 관리한다.

<br><br>

## 스프링 컨테이너의 싱글톤 기능

- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다.
    - 싱글톤 객체를 생성하고 관리하는 기능을 **싱글톤 레지스트리**라고 한다.
- 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
- DIP, OCP, 테스트, `private` 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.

<br><br>

## 스프링 컨테이너 테스트 코드

```java
import hello.core.AppConfig;
import hello.core.SingletonService;
import hello.core.member.MemberService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

public class SingletonTest {
	
	// 싱글톤이 적용되지 않은 기존의 테스트코드
	@Test
	@DisplayName("스프링 없는 순수한 DI 컨테이너")
	void pureContainer() {
		AppConfig appConfig = new AppConfig();
		
		//1. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService1 = appConfig.memberService();

		//2. 조회: 호출할 때 마다 객체를 생성
		MemberService memberService2 = appConfig.memberService();
		
		//참조값이 다른 것을 확인
		System.out.println("memberService1 = " + memberService1);
		System.out.println("memberService2 = " + memberService2);

		//memberService1 != memberService2
		// 서로 다른 객체이다.
		assertThat(memberService1).isNotSameAs(memberService2);
	}

	//싱글톤이 적용된 싱글톤패턴 테스트코드
	@Test
	@DisplayName("싱글톤 패턴을 적용한 객체 사용")
	public void singletonServiceTest() {
		//private으로 생성자를 막아두었다. 컴파일 오류가 발생한다.
		//new SingletonService();
		
		//1. 조회: 호출할 때 마다 같은 객체를 반환
    SingletonService singletonService1 = SingletonService.getInstance();
		
		//2. 조회: 호출할 때 마다 같은 객체를 반환
		SingletonService singletonService2 = SingletonService.getInstance();
		
		//참조값이 같은 것을 확인
		System.out.println("singletonService1 = " + singletonService1);
		System.out.println("singletonService2 = " + singletonService2);
		
		// singletonService1 == singletonService2
		// 서로 같은 객체이다.
		assertThat(singletonService1).isSameAs(singletonService2);
	}

	// 스프링 컨테이너를 사용하는 테스트 코드
	@Test
	@DisplayName("스프링 컨테이너와 싱글톤")
	void springContainer() {
		ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
		
		//1. 조회: 호출할 때 마다 같은 객체를 반환
		MemberService memberService1 = ac.getBean("memberService", MemberService.class);

		//2. 조회: 호출할 때 마다 같은 객체를 반환
		MemberService memberService2 = ac.getBean("memberService", MemberService.class);

		//참조값이 같은 것을 확인
		System.out.println("memberService1 = " + memberService1);     System.out.println("memberService2 = " + memberService2);

		//memberService1 == memberService2
		//서로 같은 객체이다.
		assertThat(memberService1).isSameAs(memberService2);
	}

}
```

<br><br>

## 싱글톤 컨테이너 (스프링 컨테이너) 적용 후

![싱글톤 적용 O](/assets/img/2021-07-25-SPRING_Singleton/Untitled%2011.png)

- 각 클라이언트들이 하나의 객체를 공유한다.
- 스프링 컨테이너는 바이트코드를 조작하여 싱글톤을 유지할 수 있다.

    > 자세한 내용은 [다음 게시글](https://taegyunwoo.github.io/spring/SPRING_Configuration)에서 다룬다.

> 참고로 요청마다 새로운 객체를 생성해서 반환하는 기능 역시 스프링이 지원한다.

<br><br>

## 싱글톤 방식의 주의점

- 여러 클라이언트(요청자, 고객)이 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 **무상태(stateless)** 로 설계해야 한다.
    - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
    - 가급적 읽기만 가능해야 한다.
    - 필드 대신 지역변수, 파라미터 등을 사용해야 한다.
        - 필드: 클래스 레벨에서 선언된 변수
        - 지역변수: 메서드 내에서 선언된 변수

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*