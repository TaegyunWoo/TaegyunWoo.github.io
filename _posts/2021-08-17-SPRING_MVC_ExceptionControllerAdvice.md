---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 예외처리 - @ControllerAdvice"
date:   2021-08-17 14:53:00 
lastmod : 2021-08-17 14:53:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [예외처리 - 서블릿 예외 처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)
    2. [예외처리 - 스프링 오류 페이지](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring)
    3. [예외처리 - API 예외처리와 BasicErrorController](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndBasicController)
    4. [예외처리 - API 예외처리와 HandlerExceptionResolver](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver)
    5. [예외처리 - API 예외처리와 ExceptionHandlerExceptionResolver](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpringAPI)

<br/><br/>

# @ControllerAdvice

## 개요

- [이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpringAPI#10)에서 ExceptionHandlerExceptionResolver 를 활용해서 API 예외처리를 하는 방법에 대해 알아보았다.
- [해당 글에서 작성한 컨트롤러](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpringAPI#14) ( `ExceptionHandlerController` 클래스 )의 문제점은 **정상코드와 예외처리 코드가 하나의 컨트롤러에 섞여있다는 것이다.**
- 따라서, 정상코드와 예외처리코드를 분리해보자.

<br/>

### 목표

- 정상로직를 수행하는 컨트롤러와 예외처리 컨트롤러를 분리한다.

<br/>

## 예시 코드

> [이전에 작성한 코드](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpringAPI#12)를 그대로 사용하여 분리한다.

<br/>

### 예외 처리 코드: `ExControllerAdvice` 클래스

```java
package example.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class ExControllerAdvice {

  @ResponseStatus(HttpStatus.BAD_REQUEST)
  @ExceptionHandler(IllegalArgumentException.class)
  public ErrorItem illegalExHandle(IllegalArgumentException e) {
    return new ErrorItem("BAD", e.getMessage());
  }

  @ExceptionHandler(MyException.class)
  public ResponseEntity<ErrorItem> myExHandler(MyException e) {
    ErrorItem errorItem = new ErrorItem("MY-EX", e.getMessage());
    return new ResponseEntity<>(errorItem, HttpStatus.BAD_REQUEST);
  }

  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  @ExceptionHandler
  public ErrorItem runtimeExHandle(RuntimeException e) {
    return new ErrorItem("EX", "내부오류");
  }

}
```

- `@RestControllerAdvice`
    - 해당 애너테이션을 활용하여 예외처리 전용 클래스를 작성하였다.
    - 요청 및 결과는 이전 게시글의 내용과 동일하게 작동한다.

<br/>

### 예외 발생 컨트롤러: `ExceptionHandlerController` 클래스

```java
package example.exception;

import ...

@RestController
public class ExceptionHandlerController {

    @GetMapping("/ex/{exName}")
    public String throwExceptions(@PathVariable String exName) {
        if (exName.equals("illegal")) {
            throw new IllegalArgumentException("IllegalArgument 예외!");
        }
        if (exName.equals("my")) {
            throw new MyException();
        }
        if (exName.equals("runtime")) {
            throw new RuntimeException();
        }

        return "ok";
    }

}
```

- 예외 처리 코드가 삭제되었다.

<br/><br/>

## `@ControllerAdvice` 상세설명

### 기본 원리

- `@ControllerAdvice` 는 대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler` , `@InitBinder` 기능을 부여해주는 역할을 한다.
- **대상을 지정하지 않으면 모든 컨트롤러에 적용된다. (글로벌 적용)**
- `@RestControllerAdvice` 는 `@ControllerAdvice` 와 같고, `@ResponseBody` 가 추가되어 있다.

<br/>

### 대상 컨트롤러 지정 방법

```java
// @RestController 에만 적용한다.
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// 해당 패키지의 모든 컨트롤러에 적용한다.
@ControllerAdvice("org.example.controllers") public class ExampleAdvice2 {}

// 부모클래스나 자식클래스의 컨트롤러에 적용한다.
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*