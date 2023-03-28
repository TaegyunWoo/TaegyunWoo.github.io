---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 검증 - 오류코드와 메시지처리"
date:   2021-08-13 19:00:00 
lastmod : 2021-08-13 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
  1. [검증 - 기초](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)

<br/><br/>

# 오류코드와 메시지처리

## 개요

[이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)에서 `BindingResult` 를 활용해서 검증처리하는 방법을 알아보았다. 해당 글에서 검증처리를 할 때, 오류 메시지를 직접 입력하였다.

이번 글에서는 오류 메시지를 보다 편리하고 통일성있게 처리하고, 검증처리를 더욱 간편하게 하는 방법에 대해 설명할 것이다.

이전에 다룬 예시 코드를 개선하는 방식으로 설명할 예정이니, 이전 글을 참고하기 바란다.

<br/>

### 목표

- `FieldError` , `ObjectError` 를 직접 사용하지 않고, 보다 간편한 검증 처리
- 오류 코드 자동화

<br/><br/>

## `rejectValue()` 와 `reject()`

- `BindingResult` 가 제공하는 `rejectValue()` 와 `reject()` 를 사용하여, `FieldError` 와 `ObjectError` 를 직접 생성하지 않고 검증 오류를 다룰 수 있다.

<br/>

### `rejectValue()` 란?

- 필드오류 처리를 담당하는 메서드이다.
- **`FieldError` 객체를 생성하여 `BindingResult` 에 추가한다.**

> "해당 필드(값)를 거절할거야!" 라고 생각하면 된다.

<br/>

### `rejectValue()` 의 매개변수

- 메서드 형태

    ```java
    void rejectValue(@Nullable String field, String errorCode);

    void rejectValue(@Nullable String field, String errorCode,
    			@Nullable Object[] errorArgs, @Nullable String defaultMessage);
    ```

- 매개변수 설명
    - **field**
        - 오류 필드명
    - **errorCode**
        - 오류 코드
    - **errorArgs**
        - 오류 메시지에서 `{0}` 나 `{1}` 을 치환하기 위한 값
    - **defaultMessage**
        - 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지

<br/>

### `reject()` 란?

- 오브젝트오류 처리를 담당하는 메서드이다.
- **`ObjectError` 객체를 생성하여 `BindingResult` 에 추가한다.**

> "해당 오브젝트를 거절할거야!" 라고 생각하면 된다.

<br/>

### `reject()` 의 매개변수

- 메서드 형태

    ```java
    void reject(String errorCode, String defaultMessage);

    void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
    ```

- 매개변수 설명
    - **errorCode**
        - 오류 코드
    - **errorArgs**
        - 오류 메시지에서 `{0}` 나 `{1}` 을 치환하기 위한 값
    - **defaultMessage**
        - 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지

<br/><br/>

## 예시 코드

### `application.properties` 파일 설정

```html
spring.messages.basename=messages,errors
```

이처럼 `application.properties` 파일에 작성해주어야, **스프링이 메시지 소스로 사용될 파일의 이름을 알 수 있다.**

- 메시지 소스 파일
    - `messages.properties`
    - `errors.properties`

> 메시지 관련 내용은 [이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_MessageAndInternationalization)을 참고하자.

<br/>

### `errors.properties` 파일

해당 파일에 오류 메시지로 사용될 정보를 작성해야한다. 여기에 작성된 메시지를 `rejectValue()` 와 `reject()` 가 검증 오류를 처리할 때 사용한다.

```html
required.item.itemName=아이템 이름은 필수로 입력해야합니다.
colored.item.itemColor=아이템 색상은 {0} 이어야 합니다.
appleAndQuantity=사과는 수량이 {0}보다 커야합니다. 현재 수량은 {1} 입니다.
```

> {0}, {1} 은 인자로 넘어온 값으로 치환될 곳이다. 자세한 설명은 아래에서 하겠다.

<br/>

### 컨트롤러

검증 조건은 다음과 같다.

- **itemName의 값은 빈 문자열이 올 수 없다.**
- **itemColor의 값은 “GREEN” 이어야 한다.**
- **만약 itemName이 “Apple”이라면, itemQuantity의 값은 5보다 커야한다**

```java
@PostMapping("/item")
public String validationWithRejectValue(@ModelAttribute("item") Item item, BindingResult bindingResult) {

    //첫 번째 검증 조건 (FieldError)
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.rejectValue("itemName", "required");
    }

    //두 번째 검증 조건 (FieldError)
    if (!item.getItemColor().equals("GREEN")) {
        Object[] param = {"GREEN"}; //인자로 전달할 값
        bindingResult.rejectValue("itemColor", "colored", param, null);
    }

    //세 번째 검증 조건 (ObjectError)
    if (!isAppleOver5(item)) {
        Object[] param = {5, item.getItemQuantity()}; //인자로 전달할 값
        bindingResult.reject("appleAndQuantity", param, null);
    }

    //검증 실패시
    if (bindingResult.hasErrors()) {
        return "itemform"; //다시 Item 입력 폼으로
    }

    //검증 성공시 로직...

}
```

<br/>

### 뷰 템플릿 ( `itemform.html` )

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <style>
        .error-class {
            background-color: red;
        }
    </style>
</head>
<body>
<h1>아이템 입력 폼</h1>
<form th:object="${item}" method="post">

    <div th:if="${#fields.hasGlobalErrors()}">
        <p th:each="error : ${#fields.globalErrors()}" th:text="${error}">
            글로벌 오류 (오브젝트 오류)시, 출력되는 태그
        </p>
    </div>

    <input type="text" th:field="*{itemName}" placeholder="아이템이름" th:errorclass="error-class">
    <div th:errors="*{itemName}">
        아이템 이름 오류시, 출력되는 태그
    </div> <br/>

    <input type="text" th:field="*{itemColor}" placeholder="아이템색상" th:errorclass="error-class">
    <div th:errors="*{itemColor}">
        아이템 색상 오류시, 출력되는 태그
    </div> <br/>

    <input type="text" th:field="*{itemQuantity}" placeholder="아이템개수" th:errorclass="error-class">
    <input type="submit"/>
</form>
</body>
</html>
```

<br/>

### 요청과 결과 - `rejectValue()` 관련 검증 오류 (itemName)

- 요청

    ![요청1](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%206.png)

- 결과

    ![결과1](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%207.png)

<br/>

### 요청과 결과 - `rejectValue()` 관련 검증 오류 (itemColor)

- 요청

    ![요청2](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%208.png)

- 결과

    ![결과2](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%209.png)

<br/>

- **메시지 치환**
    - `errors.properties` 내용

        ```html
        colored.item.itemColor=아이템 색상은 {0} 이어야 합니다.
        ```

    - 컨트롤러에서 메시지에 전달해준 값

        ```java
        Object[] param = {"GREEN"}; //인자로 전달할 값
        bindingResult.rejectValue("itemColor", "colored", param, null);
        ```

    - 결과

        ```html
        아이템 색상은 GREEN 이어야 합니다.
        ```

    - `rejectValue()` 에서 전달한 `Object[] param` 의 0번째 인덱스 요소가 `{0}` 와 치환된다.

<br/>

### 요청과 결과 - `reject()` 관련 검증 오류

- 요청

    ![요청3](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%2010.png)

- 결과

    ![결과3](/assets/img/2021-08-13-SPRING_MVC_ValidationAndMessage/Untitled%2011.png)

<br/>

- **메시지 치환**
    - `errors.properties` 내용

        ```html
        appleAndQuantity=사과는 수량이 {0}보다 커야합니다. 현재 수량은 {1} 입니다.
        ```

    - 컨트롤러에서 메시지에 전달해준 값

        ```java
        Object[] param = {5, item.getItemQuantity()}; //인자로 전달할 값
        bindingResult.reject("appleAndQuantity", param, null);
        ```

    - 결과

        ```html
        사과는 수량이 5보다 커야합니다. 현재 수량은 3 입니다.
        ```

    - `rejectValue()` 에서 전달한 `Object[] param` 의 0번째와 1번째 인덱스 요소가 각각 `{0}` 와  `{1}` 에 대해 치환된다.

<br/>

### 출력된 결과에 대한 의문점?

- 위처럼, 테스트를 해보았는데 `errors.properties` 에 작성한 내용이 정상적으로 출력되었다.
- **하지만 우리는 `rejectValue("itemName", "required")` 에서 `"required"` 같이, 존재하지 않는 오류 코드를 사용했다.**
    - `errors.properties` 에 작성된 내용

        ```html
        required.item.itemName=아이템 이름은 필수로 입력해야합니다.
        ```

    - `rejectValue()` 에 넘겨준 매개변수

        ```java
        **rejectValue("itemName", "required")**
        ```

- **위 코드가 정상적으로 작동하는 것에는 `MessageCodesResolver` 덕분이다.**

<br/><br/>

## `MessageCodesResolver`

- 검증 오류 코드로 메시지 코드들을 생성한다.
- **즉, 오류 코드를 기반으로 다양한 메시지 코드들을 자동으로 생성해준다.**
- 구현체는 `DefaultMessageCodesResolver` 이다.

<br/>

### `DefaultMessageCodesResolver` 의 기본 메시지 생성 규칙

- **객체 오류**

    ```html
    1순위: code + "." + object_name
    2순위: code
    ```

    - 메시지코드 생성 예시
        - 오류코드: `required` , object_name: `item`
        1. `required.item`
        2. `required`

<br/>

- **필드 오류**

    ```html
    1순위: code + "." + object_name + "." + field_name
    2순위: code + "." + field_name
    3순위: code + "." + field_type
    4순위: code
    ```

    - 메시지코드 생성 예시
        - 오류코드: `required` , object_name: `item` , field_name: `itemColor` , field_type: `String`
        1. `required.item.itemColor`
        2. `required.itemColor`
        3. `required.String`
        4. `required`

<br/>

### 동작 방식

1. `rejectValue()` , `reject()` 는 내부에서 `MessageCodesResolver` 를 사용한다. 여기에서 메시지 코드들을 생성한다.
2. **`MessageCodesResolver` 를 통해 생성 순위대로 오류 코드를 배열에 담고, 이것을 `FieldError` 나 `ObjectError` 생성자에게 전달해준다.**
3. `FieldError` , `ObjectError` 의 생성자는 매개변수로 여러 오류 코드를 배열로 받는다.
4. 생성자를 통해 생성된 `FieldError` 나 `ObjectError` 를 `BindingResult` 에 추가한다.
5. `BindingResult` 가 뷰 템플릿 (타임리프)에 전달된다.
6. **가장 먼저 생성된 (생성순위가 높은) 메시지코드를 최우선으로 선택하여 메시지를 출력한다.**

<br/>

### 참고사항

- `FieldError` 나 `ObjectError` 객체는 메시지 코드 (KEY) 만을 가지고 있다. 그리고, 메시지 코드 (KEY) 만을 뷰 템플릿에 전달한다.
- 타임리프의 `th:errors` 를 통해 메시지(VALUE)를 출력한다.

<br/><br/>

## 스프링이 스스로 추가한 검증 오류

`@ModelAttribute` 를 통해 바인딩할 때 발생하는 타입 오류는 **스프링이 스스로 `BindingResult` 에 `FieldError` 를 추가한다.** 이 경우, 출력되는 메시지를 어떻게 변경해야할까?

매우 간단하다. 그저 `errors.properties` 에 작성하여 변경할 수 있다.

<br/>

### 타입오류의 오류코드

스프링은 타입 오류가 발생하면, `rejectValue()` 의 매개변수로 `typeMismatch` 를 넘겨서 오류 코드를 지정한다. 따라서, `MessageCodesResolver` 를 통해 생성되는 메시지 코드는 다음과 같다.

```html
1순위: typeMismatch.오브젝트이름.필드이름
2순위: typeMismatch.필드이름
3순위: typeMismatch.필드타입
4순위: typeMismatch
```

<br/>

### `errors.properties` 파일 수정

`errors.properties` 파일에 원하는 메시지를 작성하면 된다.

```html
typeMismatch.오브젝트이름.필드이름=변경된 메시지 1
typeMismatch.필드이름=변경된 메시지 2
typeMismatch.필드타입=변경된 메시지 3
typeMismatch=변경된 메시지 4
```

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*