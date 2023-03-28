---
category: Spring-Core
tags: [Spring-Core]
title: "[스프링 - 핵심원리] 제어의 역전, 의존관계 주입, 컨테이너"
date:   2021-07-24 00:30:00 
lastmod : 2021-07-24 00:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 제어의 역전: IoC(Inversion of Control)

## 기존 프로그램의 동작 방식

- 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 **생성**, **연결**, **실행** 한다.
- 즉, **구현 객체가 프로그램 제어의 주체** 가 된다.

<br>

## AppConfig를 구현한 프로그램의 동작 방식

> `AppConfig` 관련 내용은 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고

- 기존 프로그램과는 다르게 프로그램의 제어 흐름을 `AppConfig`가 가져간다.

- Ex) [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 의 `MemberServiceImpl` 클래스는 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다.
    - 왜냐하면, `AppConfig` 가 구현 객체를 주입해주기 때문이다.
    - 그럼에도 `MemberServiceImpl` 클래스는 자신의 로직을 묵묵히 수행한다.

- 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 **제어의 역전 (IoC)** 라고 한다.

<br><br>

# 의존관계 주입: DI(Dependency Injection)

## 의존관계

- '인터페이스에만 의존하는 클래스'는 실제로 어떤 구현 객체가 사용될지 모른다.

- 의존관계는 **정적인 클래스 의존 관계** 와 **실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계** 둘을 분리해서 생각해야한다.

<br>

## 정적인 클래스 의존관계

- 클래스가 사용하는 `import` 코드만 보고 의존관계를 쉽게 판단할 수 있다.

- 클래스 의존관계 만으로는 실제 어떤 객체가 주입되는지 알 수 없다.
- 예시
  - 클래스 다이어그램이 아래와 같이 주어졌다.
    (클래스 다이어그램은 클래스간의 의존관계를 나타낸다.)

    ![클래스다이어그램](/assets/img/2021-07-24-SPRING_IoC_DI_Container/Untitled%202.png)

  - 여기서 `OrderServiceImpl` 클래스에게 `MemoryMemberRepository` 구현 객체와 `DbMemberRepository` 구체 구현 중 어떤 것이 주입되었는지 알 수 없다.
  
  - `FixDiscountPolicy` 구현 객체와 `RateDiscountPolicy` 구현 객체 역시 어떤 것이 주입되었는지 알 수 없다.

<br>

## 동적인 객체 인스턴스 의존 관계

- 객체 인스턴스 의존 관계란, 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.

- 객체 인스턴스 의존 관계는 객체 다이어그램을 통해 확인할 수 있다.
  
  ![객체다이어그램](/assets/img/2021-07-24-SPRING_IoC_DI_Container/Untitled%203.png)

<br>

## 의존관계 주입 (DI)

- **의존관계 주입** 이란, 애플리케이션 실행 시점에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트(구현 객체를 사용하는 클래스)와 서버(구현 객체의 클래스)의 실제 의존관계가 연결 되는 것을 뜻한다.

  > [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고

<br>

- 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대사의 타입 인스턴스를 변경할 수 있다.

- 의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.

<br><br>

# IoC 컨테이너, DI 컨테이너

- AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것  
⇒ IoC 컨테이너 또는 DI 컨테이너

  > 최근에는 의존관계 주입에 초점을 맞추어 DI 컨테이너라 한다.

  > IoC는 DI를 포함하는 포괄적인 개념이다.

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*