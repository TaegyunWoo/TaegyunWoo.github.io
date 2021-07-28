---
category: Spring-Core
tags: [스프링, 핵심원리, 컨테이너, 컨테이너생성]
title: "[스프링 - 핵심원리] 스프링 컨테이너 생성"
date:   2021-07-24 18:30:00 
lastmod : 2021-07-24 18:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 개요

## 스프링 컨테이너 생성 코드

### 생성 코드

```java
//스프링 컨테이너 생성
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

### 설명

- `ApplicationContext`
    - 이것을 스프링 컨테이너라 한다.
    - 인터페이스이다.
    - 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.

- `AppConfig`
    - 자바 설정 클래스를 의미한다.

    > 자세한 것은 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고

- `AnnotationConfigApplicationContext(AppConfig.class)`
    - `AnnotationConfigApplicationContext` 는 `ApplicationContext` 의 구현체이다.
    - 즉, `AppConfig` 클래스가 자바 설정 클래스의 역할을 수행하기 때문에, 자바 설정 클래스를 기반으로 스프링 컨테이너를 만들었다고 할 수 있다.

# 스프링 컨테이너의 생성 과정

## 스프링 컨테이너 생성

![스프링 컨테이너 생성](/assets/img/2021-07-24-SPRING_Contatiner_Construct/Untitled%204.png)

- `new AnnotationConfigApplicationContext(AppConfig.class)`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.
- `AppConfig.class` 를 구성 정보로 지정했다.

## 스프링 빈 등록

![스프링 빈 등록](/assets/img/2021-07-24-SPRING_Contatiner_Construct/Untitled%205.png)

- 스프링 컨테이너는 파라미터로 넘어온 **설정 클래스 정보를 사용해서 스프링 빈을 등록** 한다.
- 빈 이름
    - 빈 이름은 메서드 이름을 사용한다.
        - `@Bean` 을 사용하면 빈이름이 자동으로 메서드 이름으로 설정된다.
    - 빈 이름을 직접 부여할 수 도 있다.
        - `@Bean(name="memberService")` 를 사용하면 빈이름을 직접 설정할 수 있다.
    - **빈 이름은 항상 다른 이름을 부여해야 한다.**
        - 같은 이름 부여시, 다른 빈이 무시되거나, 기존 빈을 덮어버리는 등의 문제가 발생한다.

## 스프링 빈 의존관계 설정

- 준비

    ![준비](/assets/img/2021-07-24-SPRING_Contatiner_Construct/Untitled%206.png)

- 완료

    ![완료](/assets/img/2021-07-24-SPRING_Contatiner_Construct/Untitled%207.png)

    - 스프링 컨테이너는 설정 정보 (`AppConfig.class` ) 를 참고해서 의존관계를 주입한다. (**DI**)

    > 스프링 자체는 원래 '빈생성', '의존관계 주입' 단계가 나누어져 있지만,  
    자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다.  
    ( `return new MemberServiceImpl( memberRepository() )` 에서 `MemberServiceImpl` 생성자와 `memberRepository()` 메서드가 같이 호출되기 때문에 )

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*