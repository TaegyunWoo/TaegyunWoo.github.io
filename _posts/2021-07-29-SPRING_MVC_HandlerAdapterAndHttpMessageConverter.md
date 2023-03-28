---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 요청 매핑 핸들러 어댑터 구조"
date:   2021-07-29 16:30:00 
lastmod : 2021-07-29 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# RequestMappingHandlerAdapter

## RequestMappingHandlerAdapter 란?

- 핸들러 어댑터의 한 종류로, `@RequestMapping` 을 처리하는 핸들러 어댑터이다.
- [이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_HTTPMessageConverter)에서 다룬 HTTP 메시지 컨버터 기능을 가지고 있는 어댑터이다.

<br><br>

## 동작 방식

![동작 방식](/assets/img/2021-07-29-SPRING_MVC_HandlerAdapterAndHttpMessageConverter/Untitled%205.png)

1. `DispatcherServlet` (프론트컨트롤러)가 핸들러 어댑터 `RequestMappingHandlerAdapter` 를 실행한다.
2. `ArgumentResolver` 를 통해 '핸들러의 매개변수'에 값을 준다.
3. 핸들러가 반환한 값을 `ReturnValueHandler` 가 전달받아서 `ModelAndView` 객체로 만들거나, `@ResponseBody` , `HttpEntity` 로 변환하여 핸들러 어댑터에게 전달해준다.
4. 핸들러 어댑터는 변환된 값을 `DispatcherServlet` 에게 전달해준다.

<br>

### Argument Resolver

- 애노테이션 기반의 컨트롤러가 다양한 매개변수를 사용할 수 있도록 지원한다.
- **컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성하여, 컨트롤러에게 넘겨준다.**
- Argument Resolver에 지정된 타입은 `@ModelAttribute` 로 가져올 수 없다.
- 또한 Argument Resolver로 컨트롤러에게 넘겨줄 타입을 직접 지정해서 넘겨줄 수 있다.

<br>

### HTTP 메시지 컨버터의 위치

![HTTP 메시지 컨버터의 위치](/assets/img/2021-07-29-SPRING_MVC_HandlerAdapterAndHttpMessageConverter/Untitled%206.png)

- HTTP 메시지 컨버터가 작동하는 위치는 위 그림과 같다.
- 컨트롤러가 `@RequestBody` 나 `HttpEntity` 를 통해 매개변수를 제공받고자 하면 **HTTP 메시지 컨버터가 작동** 된다.
- 또한, 컨트롤러가 `@ResponseBody` 나 `HttpEntity` 객체를 반환하고자 하면 **HTTP 메시지 컨버터가 작동** 된다.

<br>

### 요청의 경우

- `@RequestBody` 를 처리하는 `ArgumentResolver` 가 있고,  
`HttpEntity` 를 처리하는 `ArgumentResolver` 가 있다.
- **이때** `ArgumentResolver` **들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.**

<br>

### 응답의 경우

- `@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다.
- `ReturnValueHandler` 가 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*