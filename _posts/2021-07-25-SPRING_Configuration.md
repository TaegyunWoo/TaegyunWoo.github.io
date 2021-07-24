---
category: Spring
tags: [스프링, 핵심원리, 싱글톤, Configuration_애너테이션]
title: "[스프링 - 핵심원리] @Configuration과 싱글톤"
date:   2021-07-25 01:30:00 
lastmod : 2021-07-25 01:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 문제 제기

## AppConfig 클래스의 의문점

`AppConfig` 클래스의 코드에 `orderService()` 메서드를 추가해보자. 아래와 같이 추가하더라도 `AppConfig` 는 정상적으로 동작한다.

> 혹시 이전 글을 보지 않은 분들은 이전 글들을 한번 읽기 바란다.

> 해당 코드가 정상적으로 동작하는 것은 각자 테스트해보길 바란다.

<br>

### `AppConfig` 클래스

```java
@Configuration
public class AppConfig {
	
	@Bean
	public MemberService memberService() {
		return new MemberServiceImpl(memberRepository());
	}

	//이전 글에서 작성한 AppConfig 클래스에서 새로 추가된 메서드
	@Bean
	public OrderService orderService() {
		return new OrderServiceImpl( memberRepository(), discountPolicy());
	}

	@Bean
	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}

	//이하 생략..

}
```

- `orderService()` 메서드가 추가되었지만, 위 코드( `AppConfig` )는 정상적으로 작동하여 싱글톤 패턴을 유지한다.

<br>

### 의문점

- `memberService()` 메서드

    - `memberRepository()` 를 호출하여 `MemoryMemberRepository` 객체를 **생성** 하여 주입한다.
- `orderService()` 메서드
    - 이 메서드 역시 `memberRepository()` 를 호출하여 `MemoryMemberRepository` 객체를 **생성** 하여 주입한다.
- `AppConfig` 는 스프링 컨테이너에 설정 정보를 정상적으로 전달한다. ( 각자 테스트를 통해 확인했을 것이다. )
- 결과적으로 각각 다른 2개의 `MemoryMemberRepository` 가 생성되면서 싱글톤이 깨지는 것처럼 보인다. 하지만 스프링 컨테이너는 싱글톤을 유지하게 된다. 그렇다면 스프링은 이 문제를 어떻게 해결했을까?

<br><br><br>

# @Configuration 애너테이션을 통한 바이트코드 조작

## 문제 해결

위와 같은 문제를 해결하기 위해 스프링은 **클래스의 바이트코드를 조작** 하는 라이브러리를 사용한다.

<br><br>

## 바이트코드 조작

### 테스트코드

아래와 같은 테스트코드를 통해 스프링이 정말로 바이트코드를 조작하는지 확인해보자.

```java
@Test
void configurationDeep() {
	ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	//AppConfig도 스프링 빈으로 등록된다.
	AppConfig bean = ac.getBean(AppConfig.class);
	
	System.out.println("bean = " + bean.getClass());
	//출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```

<br>

### 예상 출력결과

class hello.core.AppConfig

<br>

### 실제 출력결과

bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB\$\$bd479d70

<br>

### 분석

왜 예상과 다르게 결과가 출력되었을까?

- 예상과 다르게 클래스 명에 `xxxCGLIB` 가 붙었다.

- 이것은 우리가 만든 클래스가 아니라 **스프링이 CGLIB라는 바이트코드 조작 라이브러리를 통해서** `AppConfig` **클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것** 이다.
- 그리고 이러한 동작을 `@Configuration` 애너테이션을 통해 수행한다.

<br><br>

## 애너테이션

### @Configuration

![@Configuration 애너테이션](/assets/img/2021-07-25-SPRING_Configuration/Untitled%2012.png)

- 스프링이 'AppConfig를 상속받은 클래스'를 빈으로 등록한다.

- 'AppConfig를 상속받은 클래스' 가 싱글톤이 보장되도록 해준다.
- 'AppConfig를 상속받은 클래스'는 `@Bean` 이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

<br>

### @Bean 만 적용한다면?

위에서 설명했듯이, @Configuration 애너테이션을 통해 스프링은 바이트코드를 조작하여 싱글톤을 유지한다. 그렇다면 @Configuration 애너테이션을 생략하면 어떻게 될까? 아래 코드와 같이 말이다.

```java
// @Configuration 삭제
public class AppConfig {
	
	@Bean
	public MemberService memberService() {
		return new MemberServiceImpl(memberRepository());
	}

	@Bean
	public OrderService orderService() {
		return new OrderServiceImpl( memberRepository(), discountPolicy());
	}

	@Bean
	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}

	//이하 생략..

}
```

모두 예상했다시피, `memberRepository()` 메서드가 `memberService()` 와 `orderService()` 를 통해 중복되어 호출되고 **바이트코드 조작이 되지 않아** 여러 개의 `MemoryMemberRepository` 객체가 생성된다.

따라서, 반드시 설정 정보 클래스에는 `@Configuration` 애너테이션을 붙이도록 하자.

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*