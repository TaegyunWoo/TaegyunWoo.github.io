---
category: Spring-MVC
tags: [스프링, MVC, 검증]
title: "[스프링 - MVC] 검증 - 검증 로직과 컨트롤러 분리"
date:   2021-08-13 19:50:00 
lastmod : 2021-08-13 19:50:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글: [검증 - 기초](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)
- 이전 게시글: [검증 - 오류코드와 메시지처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)

<br/><br/>

# 검증로직과 컨트롤러 분리

## 개요

[이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)에서 작성한 컨트롤러를 보자. 검증 관련 로직이 상당히 복잡한 것을 알 수 있다. 이번 게시글에서는 이런 복잡한 검증 로직을 따로 떼어내어내는 방법을 설명할 것이다.

<br/>

### 목표

- 복잡한 검증 로직을 별도로 분리한다.

<br/><br/>

## `Validator` 인터페이스

```java
public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```

- 스프링이 제공하는 `Validator` 인터페이스는 검증을 체계적으로 제공하기 위한 것이다.
- `Validator` 인터페이스를 구현하여, 검증로직을 컨트롤러에서 따로 떼어낼 수 있다.

<br/>

### 메서드 설명

- `supports()`
    - 해당 검증기를 지원하는 여부를 확인하는 메서드
- `validator()`
    - 검증 대상 객체와 `BindingResult`

예시를 통해 알아보자.

<br/><br/>

## 기존 코드

기존 코드는 [이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)에서 사용한 것을 이용한다. 참고하자.

<br/>

### `application.properties` 파일

```html
spring.messages.basename=messages,errors
```

<br/>

### `errors.properties` 파일

```html
required.item.itemName=아이템 이름은 필수로 입력해야합니다.
colored.item.itemColor=아이템 색상은 {0} 이어야 합니다.
appleAndQuantity=사과는 수량이 {0}보다 커야합니다. 현재 수량은 {1} 입니다.
```

<br/>

### 컨트롤러

```java
package prac.myPrac.controller;

import ...

@Controller
public class ItemController {

    //사용자에게 view를 제공
    @GetMapping("/item")
    public String viewItemForm(@ModelAttribute Item item) {
        return "itemform";
    }

    @PostMapping("/item")
    public String validationWithRejectValue(@ModelAttribute("item") Item item, BindingResult bindingResult) {

        //첫 번째 검증 조건 (FieldError)
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.rejectValue("itemName", "required");
        }

        //두 번째 검증 조건 (FieldError)
        if (!item.getItemColor().equals("GREEN")) {
            Object[] param = {"GREEN"};
            bindingResult.rejectValue("itemColor", "colored", param, null);
        }

        //세 번째 검증 조건 (ObjectError)
        if (!isAppleOver5(item)) {
            Object[] param = {5, item.getItemQuantity()};
            bindingResult.reject("appleAndQuantity", param, null);
        }

        //검증 실패시
        if (bindingResult.hasErrors()) {
            return "itemform"; //다시 Item 입력 폼으로
        }

        //검증 성공시 로직...

    }

    private boolean isAppleOver5(Item item) {
        String itemName = item.getItemName();
        int itemQuantity = item.getItemQuantity();

        if (!itemName.equals("Apple")) {
            return true;
        }

        return itemQuantity>5;
    }
}
```

<br/>

### 뷰 템플릿 (`itemform.html`)

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

## 검증 로직 분리하기

이제 본격적으로 검증 관련 로직을 떼어내보자.

<br/>

### `Validator` 인터페이스 구현 (`ItemValidator` 클래스)

참고로 검증 조건은 다음과 같다.

- **itemName의 값은 빈 문자열이 올 수 없다.**
- **itemColor의 값은 "GREEN" 이어야 한다.**
- **만약 itemName이 "Apple"이라면, itemQuantity의 값은 5보다 커야한다.**

```java
package prac.myPrac.controller;

import ...

@Component
public class ItemValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz); //Item 객체와 clazz 객체의 타입이 같느냐? (자식포함)
    }

    /**
     * supports() 메서드가 true이면 호출될 메서드
		 * 실제 검증 로직을 작성한다.
     */
    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        //첫 번째 검증 조건 (FieldError)
        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required");
        }

        //두 번째 검증 조건 (FieldError)
        if (!item.getItemColor().equals("GREEN")) {
            Object[] param = {"GREEN"};
            errors.rejectValue("itemColor", "colored", param, null);
        }

        //세 번째 검증 조건 (ObjectError)
        if (!isAppleOver5(item)) {
            Object[] param = {5, item.getItemQuantity()};
            errors.reject("appleAndQuantity", param, null);
        }
    }

    private boolean isAppleOver5(Item item) {
        String itemName = item.getItemName();
        int itemQuantity = item.getItemQuantity();

        if (!itemName.equals("Apple")) {
            return true;
        }

        return itemQuantity>5;
    }
}
```

- **`validate()` 메서드 내부에 작성된 코드가 기존 컨트롤러의 검증부분과 거의 같다는 것을 알 수 있다.**
- `@Component` 을 통해 빈 등록을 해야, 컨트롤러가 주입받을 수 있다.

<br/>

### 검증로직이 분리된 컨트롤러

```java
package prac.myPrac.controller;

import ...

@Controller
public class ItemController {

	private final ItemValidator itemValidator; //의존관계 주입

  @Autowired
  public ItemController(ItemValidator itemValidator) {
      this.itemValidator = itemValidator;
  }

  //사용자에게 view를 제공
  @GetMapping("/item")
  public String viewItemForm(@ModelAttribute Item item) {
      return "itemform";
  }

  @PostMapping("/item")
  public String validationWithRejectValue(@ModelAttribute("item") Item item, BindingResult bindingResult) {

		/*
		 * 검증 로직 삭제
		 */

      itemValidator.validate(item, bindingResult);

      //검증 실패시
      if (bindingResult.hasErrors()) {
          return "itemform"; //다시 Item 입력 폼으로
      }

      //검증 성공시 로직...
  }

}
```

- `ItemValidator` 를 스프링 빈으로 주입 받아서 직접 호출했다.
- `ItemValidator` 를 통해 검증을 수행했다.
- **결과적으로, 검증 로직과 컨트롤러가 분리되었다.**

<br/><br/>

## `WebDataBinder` 와 `@Validator`

위에서 설명한 방법으로 검증기를 충분히 분리할 수 있지만, 좀 더 편리한 방법이 있다.

`WebDataBinder` 와 `@Validator` 를 활용하면 된다.

<br/>

### `WebDataBinder`

- `@InitBinder` 를 통해 검증기 (`ItemValidator` 와 같은) 를 `WebDataBinder` 에 등록하면, 해당 컨트롤러에서 검증기를 자동으로 적용할 수 있다.
- `@InitBinder` 는 해당 컨트롤러에서만 적용된다.

<br/>

### `@Validator`

- `WebDataBinder` 에 등록한 검증기를 편리하게 사용할 수 있게 해주는 애너테이션이다.
- `ItemValidator` 와 같은 검증기를 직접 호출하지 않고, 사용할 수 있게 해준다.
- 구체적인 사용방법은 아래에서 설명한다.

<br/>

### `WebDataBinder` 와 `@Validator` 를 적용한 컨트롤러

```java
package prac.myPrac.controller;

import ...

@Controller
public class ItemController {

    private final ItemValidator itemValidator;

    @Autowired
    public ItemController(ItemValidator itemValidator) {
        this.itemValidator = itemValidator;
    }

    /**
     * dataBinder 는 매 요청마다 생성된다.
     */
    @InitBinder
    public void init(WebDataBinder dataBinder) {
        dataBinder.addValidators(itemValidator); //검증기 등록
    }

    //사용자에게 view를 제공
    @GetMapping("/item")
    public String viewItemForm(@ModelAttribute Item item) {
        return "itemform";
    }

		/**
		 * 매개변수에 @Validator가 추가되었다.
		 */
    @PostMapping("/item")
    public String validationWithRejectValue(@Validator @ModelAttribute("item") Item item, BindingResult bindingResult) {

        //검증 실패시
        if (bindingResult.hasErrors()) {
            return "itemform"; //다시 Item 입력 폼으로
        }

        //검증 성공시 로직...

    }

    private boolean isAppleOver5(Item item) {
        String itemName = item.getItemName();
        int itemQuantity = item.getItemQuantity();

        if (!itemName.equals("Apple")) {
            return true;
        }

        return itemQuantity>5;
    }
}
```

<br/>

### 동작 방식

1. `@Validated` 는 검증기를 실행하라는 애노테이션이다.
2. 어떤 검증기를 실행할지 선택하는 기준이 바로 `supports()` 이다.
3. 위 컨트롤러 예시에서는 `supports(Item.class)` 가 호출되고, 결과가 `true` 이다.
4. 따라서, `ItemValidator` 의 `validate()` 가 호출된다.

<br/>

### 참고

- `@Validated` 와 `@Valid` 모두 사용 가능하다.
- `@Validated` 은 스프링 전용 검증 애노테이션이다.
- `@Valid` 은 자바 표준 검증 애노테이션이다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*