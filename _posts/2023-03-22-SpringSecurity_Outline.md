---
category: Spring-Security
tags: [Spring, SpringSecurity]
title: "[Spring-Security] Spring Security가 무엇인가요?"
date:   2023-03-22 17:15:00 
lastmod : 2023-03-22 17:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

Spring 에서 인증/인가, 보안관리를 안전하고 편리하게 구현할 수 있도록 지원하는 프레임워크 Spring Security에 대해 알아보겠습니다.

> 앞으로 몇 개의 시리즈로 이어서 포스팅할 예정입니다.

이번 포스팅에서는 Spring Security 가 정확히 무엇이고 어떤 기능을 지원하는지 등의 개요적인 내용을 다룹니다.

<br/><br/>

## Spring Security 란 무엇인가요?

Spring Security는 인증(Authentication), 인가(Authorization), 그리고 보안 기능을 제공하는 프레임워크입니다.

Spring 프레임워크 기반의 애플리케이션을 보호하기 위해 사용되며, **거의 표준처럼 사용되기 때문에 잘 알아두는 것이 좋습니다.**

일반적인 `Servlet형 Spring 애플리케이션`뿐만 아니라, `Reactive형 Spring 애플리케이션`에도 적용할 수 있습니다.

> Reactive Spring (WebFlux) 은 [이 링크](https://docs.spring.io/spring-security/reference/reactive/index.html)를 참고하세요!

<br/><br/>

## Spring Security의 기능

Spring Security를 적용하면, 여러 기능을 제공받을 수 있습니다.

가장 먼저 Spring Security를 적용했을 때, 우리의 Spring 애플리케이션이 어떻게 달라지는지 알아봅시다!

<br/>

### Spring Boot의 변화

우리가 Gradle이나 Maven을 통해 Spring Security 의존성을 추가하면, Spring은 아래와 같은 변경사항을 갖게 됩니다.

- Spring Security의 기본 설정(configuration)이 활성화됩니다.
    - 이 기본 설정을 통해, `서블릿 필터` 객체를 Bean으로 등록하는데, 이 객체는 `springSecurityFilterChain` 이라는 이름을 갖습니다.
    - 이 `서블릿 필터` 객체를 통해, 보안과 관련된 모든 작업을 수행하게 됩니다.  
    예를 든다면, 전송된 아이디와 비밀번호를 검증하거나, URL 접근을 제어하는 등의 보안 작업이 있습니다.
- `UserDetailsService` 라는 Bean 객체가 등록됩니다.
    - 이 객체는 기본적으로 로그인할 수 있는 ID와 Password를 갖습니다.
    - ID : user  
    Password : 무작위로 생성된 값으로, Spring 애플리케이션을 실행할 때 콘솔창에 출력됩니다.

<br/>

### 제공받을 수 있는 기능

위에서  Spring Boot에 어떤 변화가 생기는지 알아보았습니다.

그렇다면 이런 변화를 통해, 구체적으로 **어떤 기능을 제공**받을 수 있는 것일까요?

이는 아래와 같습니다.

- 모든 상호 작용에 대해 인증된 사용자를 요구합니다.
- 기본적인 로그인 양식을 생성해줍니다.
- 기본적으로 제공된 사용자 정보(ID, Password)로 인증할 수 있도록 합니다.
    - user라는 아이디와, 콘솔에 자동으로 출력되는 Password를 사용한 인증을 허용합니다.
- `BCrypt` 를 통해, 암호를 안전하게 저장합니다.
- 로그아웃을 지원합니다.
- CSRF 공격을 방지합니다.
    
    > CSRF란 Cross-site Request Forgery 의 약자로, 악의적인 스크립트 코드를 서버에 전송하여 다른 사용자가 해당 스크립트 코드를 실행하도록 만드는 공격 기술입니다.  
    예를 들어, 게시글에 HTML 태그와 JS 코드를 작성해, 해당 게시글을 조회하는 다른 사람들이 해당 JS 코드를 실행하게 만들 수 있습니다.
    > 
- Session Fixation 공격을 방지합니다.
    
    > Session Fixation 이란, 특정 사용자의 세션ID값을 탈취해, 해당 사용자인 척하며 악의적인 행동을 하는 것을 말합니다.
    > 
- 보안과 관련된 여러 헤더들을 통합하여 관리합니다.

<br/><br/>

## 정리

Spring Security가 정확히 무엇이고, Spring에 적용했을 때 일어나는 변화와 기능들에 대해 알아봤습니다.

앞으로 올릴 포스팅을 통해서, 여기서 다룬 내용들의 상세한 부분을 다룰게요!

이런 것들이 있구나~ 정도로만 생각해주시면 되겠습니다.

<br/><br/>

## References

- [https://docs.spring.io/spring-security/reference/servlet/getting-started.html](https://docs.spring.io/spring-security/reference/servlet/getting-started.html)
- [https://en.wikipedia.org/wiki/Cross-site_request_forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery)
- [https://en.wikipedia.org/wiki/Session_fixation](https://en.wikipedia.org/wiki/Session_fixation)