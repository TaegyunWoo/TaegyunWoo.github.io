---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 예외처리 - 스프링 오류 페이지"
date:   2021-08-16 17:00:00 
lastmod : 2021-08-16 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [예외처리 - 서블릿 예외 처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)

<br/><br/>

# 스프링 오류 페이지

## 개요

- [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)에서 서블릿에서 예외를 처리하는 방법에 대해 알아보았다.
- 스프링은 이러한 과정을 모두 기본으로 제공한다.

<br/><br/>

## 스프링이 제공하는 오류 페이지 처리 기능

- `ErrorPage` 를 자동으로 등록한다.
- 기본 오류 페이지 경로: `templates/error` , `static/error`
    - 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다.
    - **서블릿 밖으로 예외가 발생하거나, `response.sendError()` 가 호출되면 모든 오류는 `/error` 를 호출하게 된다.**
    - **그리고 `BasicErrorController` 가 `/error` 요청을 처리한다.**
- **`BasicErrorController` 라는 스프링 컨트롤러를 자동으로 등록한다.**
    - `ErrorPage` 에서 등록한 `/error` 를 매핑해서 처리하는 컨트롤러이다.
- **따라서 개발자는 오류 페이지만 등록하면 된다.**

<br/>

### `BasicErrorController` 의 뷰 선택 우선순위

1. **뷰 템플릿**
    - `resource/templates/error/500.html`
    - `resource/templates/error/5xx.html`
2. **정적 리소스**
    - `resource/static/error/400.html`
    - `resource/static/error/4xx.html`
3. **적용 대상이 없을 때 뷰 이름 ( `error` )**
    - `/resources/templates/error.html`

<br/>

### `BasicErrorController` 의 뷰 선택 규칙

- 해당 경로 위치에 HTTP 상태 코드 이름의 뷰 파일을 넣어두면 된다.
- 뷰 템플릿이 정적 리소스보다 우선순위가 높다.
- 400, 500 처럼 구체적인 것이 5xx처럼 덜 구체적인 것보다 우선순위가 높다.
- 4xx, 5xx 라고 하면 500대, 400대 오류를 처리해준다.

<br/><br/>

## 스프링 오류 처리 예시

> 예시코드 실습을 위해, [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet#14)에서 작성한 `WebServerCustomizer` 클래스에서 `@Component` 애너테이션을 주석처리하자.

<br/>

### 오류 뷰 템플릿 1: `resources/templates/error/4xx.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <h1>BasicErrorController 가 호출해준 오류 페이지</h1>
  <h1>400대 에러를 처리한다.</h1>
</body>
</html>
```

<br/>

### 오류 뷰 템플릿 2: `resources/templates/error/404.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <h1>BasicErrorController 가 호출해준 오류 페이지</h1>
  <h1>404 에러를 처리한다.</h1>
</body>
</html>
```

<br/>

### 오류 뷰 템플릿 3: `resources/templates/error/500.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <h1>BasicErrorController 가 호출해준 오류 페이지</h1>
  <h1>500 에러를 처리한다.</h1>
</body>
</html>
```

<br/>

### 결과

- 호출: `http://localhost:8080/sendError`

    ![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionSpring/Untitled%2010.png)

- 호출: `http://localhost:8080/존재하지않는경로`

    ![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionSpring/Untitled%2011.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*