---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 예외처리 - API 예외처리와 BasicErrorController"
date:   2021-08-16 18:00:00 
lastmod : 2021-08-16 18:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [예외처리 - 서블릿 예외 처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)
    2. [예외처리 - 스프링 오류 페이지](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring)

<br/><br/>

# 예외 API 응답

## 개요

- 지금까지 예외 발생시, 오류 페이지로 응답하는 방법에 대해 알아보았다.
- 만약 예외발생시 API 스타일로, JSON 응답을 하려면 어떻게 해야할까?
- 그리고 `BasicErrorController` 의 한계는 무엇일까?

<br/><br/>

## `BasicErrorController` 기능

> `BasicErrorController` 의 자세한 내용은 [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring#2)을 참고하자.

`BasicErrorController` 는 아래와 같이 두 가지 기능을 제공한다.

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}

@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

- **`errorHtml(...)`**
    - `@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)`
        - 클라이언트 요청의 Accept 헤더 값이 `text/html` 인 경우에 호출된다.
    - `ModelAndView` 를 반환하며, 뷰를 제공한다.
- **`error(...)`**
    - 그외 경우에 호출된다.
    - **`ResponseEntity` 로 반환하며, HTTP Body에 JSON 데이터(에러정보)를 반환한다.**
- **스프링은 오류 발생시, 기본적으로 `/error` 를 내부요청한다. `BasicErrorController` 는 `/error` 요청을 처리한다.**
    - 이때, 요청의 Accept 헤더가 `text/html` 이면, `errorHtml(...)` 이 매핑된다.
    - 이때, 요청의 Accept 헤더가 `application/json` 이면, `error(...)` 가 매핑된다.

<br/><br/>

## `BasicErrorController` 활용하여 에러 API 응답

> **[이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring)에서 작성한 프로젝트를 그대로 사용한다.**

- 기본적으로 스프링이 `BasicErrorController` 를 통해, 에러 API 응답을 지원하므로, 따로 소스코드를 작성할 필요가 없다.

<br/>

### 요청

- Postman 프로그램으로 테스트해보자.
- 반드시 `Accept` 헤더의 값이 `application/json` 이어야 한다.

![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndBasicController/Untitled%2012.png)

<br/>

### 응답

![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndBasicController/Untitled%2013.png)

<br/><br/>

## `BasicErrorController` 의 한계점

### 에러 HTML 페이지 제공시

- 해당 상황에서는 별다른 문제없이, 편리하게 사용할 수 있다.

<br/>

### 에러 API(JSON) 제공시

- API 마다, 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수도 있다.
- **즉 어떤 종류의 예외, 어떤 컨트롤러 등의 상관없이 모두 일정한 형식의 JSON 데이터로 응답한다.**
- **따라서 `BasicErrorController` 는 HTML 응답시에만 사용하는 것을 권장한다.**

<br/>

## 정리

- **`BasicErrorController` 는 HTML 응답시에만 사용하자!**
- **`BasicErrorController` 으로 API(JSON) 응답을 하기에는, 기능이 너무 단순하다.**

> 다음 게시글에 이어서 API 응답에 대해 설명하도록 하겠다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*