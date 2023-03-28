---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 검증 - API 응답"
date:   2021-08-14 19:30:00 
lastmod : 2021-08-14 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
  1. [검증 - 기초](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)
  2. [검증 - 오류코드와 메시지처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)
  3. [검증 - 검증 로직과 컨트롤러 분리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndController)
  4. [검증 - Bean Validation](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_BeanValidation)
  5. [검증 - Form 전송 객체의 분리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_FormAndValidation)

<br/><br/>

# 검증과 API 응답

## 개요

이전 게시글에서 뷰 템플릿을 통해 검증 정보를 넘겨주는 방법에 대해 설명했다. 이번에는 API 응답을 통해 검증 정보를 사용자에게 제공하는 방법을 알아보자.

<br/><br/>

## API 응답에서의 검증

API의 경우 아래 3가지 경우를 나누어 생각해야 한다.

- **성공 요청**: 성공
- **실패 요청**: JSON을 객체로 생성하는 것 자체가 실패하는 경우 (예. TypeMismatch)
- **검증 오류 요청**: JSON을 객체로 생성하는 것은 성공했지만, 검증에서 실패하는 경우

<br/><br/>

## 예시 코드: 컨트롤러

```java
import ...

@RestController
public class ApiController {

    @PostMapping("/api")
    public Object returnToJson(@RequestBody @Validated ProductAddForm form,
                               BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return bindingResult.getAllErrors();
        }
        return form;
    }
}
```

Postman 프로그램을 통해서, 요청을 해보자.

> `ProductAddForm` 객체는 [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_FormAndValidation#6)을 참고하자.

<br/><br/>

## 성공

### 요청

![성공-요청](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2020.png)

<br/>

### 응답

![성공-응답](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2021.png)

<br/><br/>

## 실패

### 요청

![실패-요청](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2022.png)

<br/>

### 응답

![실패-응답](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2023.png)

<br/>

### 설명

- `HttpMessageConverter` 에서 요청 JSON을 `ProductAddForm` 객체로 생성하는데 실패한다.
- 이 경우, `ProductAddForm` 객체를 만들지 못하기 때문에 컨트롤러 자체가 호출되지 않고 그전에 예외가 발생한다.
- Validator 도 실행되지 않는다.

<br/><br/>

## 검증 실패

### 요청

![검증실패-요청](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2024.png)

<br/>

### 응답

![검증실패-응답](/assets/img/2021-08-14-SPRING_MVC_ValidationAndJson/Untitled%2025.png)

<br/>

### 설명

- `ProductAddForm` 의 `private Integer productPrice` 필드에 대해 `@Min(1000)` 검증에 걸린다.

<br/><br/>

## `@ModelAttribute` vs `@RequestBody`

### `@ModelAttribute`

- 각각의 필드 단위로 세밀하게 적용된다.
- **특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있다. 따라서 나머지 필드에 대해 Validator 검증도 적용할 수 있다.**
    - 스프링이 `TypeMismatch FieldError` 를 생성하여 `BindingResult` 에 추가하여 처리한다.

<br/>

### `@RequestBody`

- `HttpMessageConverter` 단계에서 JSON 데이터를 객체로 변경하지 못하면, 이후 단계 자체가 진행되지 않고 예외가 발생한다.
- **컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.**

> `HttpMessageConverter` 관련 정보는 [게시글1](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_HTTPMessageConverter), [게시글2](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_HandlerAdapterAndHttpMessageConverter#4) 을 참고하자.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*