---
category: Spring-MVC
tags: [스프링, MVC, 검증]
title: "[스프링 - MVC] 검증 - Bean Validation"
date:   2021-08-14 15:30:00 
lastmod : 2021-08-14 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
  - [검증 - 기초](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)
  - [검증 - 오류코드와 메시지처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)
  - [검증 - 검증 로직과 컨트롤러 분리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndController)

<br/><br/>

# Bean Validation

## 개요

검증에 대해 다룬 이전 게시글들에서 작성한 코드는 너무 복잡하다. 사실 따지고 보면, 특정 필드에 대한 검증 로직은 대부분 비어있는 값인지, 특정 크기를 넘는지와 같은 일반적인 로직이다. **이런 일반적인 로직을 굳이 복잡하게 직접 구현해야할까?**

Bean Validation을 통해 이러한 문제를 쉽게 해결할 수 있다.

<br/>

### Bean Validation 이란?

- Bean Validation 은 특정한 구현체가 아닌, Bean Vaildation 2.0(JSR-380) 이라는 기술 표준이다.
- 즉, 검증 애너테이션과 여러 인터페이스의 모음이다.
- Bean Validation 을 구현한 기술 중, 일반적으로 **하이버네이트 Validator** 를 사용한다. (ORM과는 관련이 없다.)
- **도메인 모델에서 애너테이션을 통한 필드값 검증을 가능하게 해준다.**

> [하이버네이트 Validator의 검증 애너테이션 모음](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/)

<br/><br/>

## Bean Validation 사용하기

### 검증 조건

- **`productName` 은 비어있으면 안된다. (필드에러)**
- **`productPrice` 는 null이면 안되고, 1000이상 10000이하 이다. (필드에러)**
- **`productQuantity` 는 null이면 안되고, 최대 100이다. (필드에러)**
- **`productQuantity` * `productPrice` 의 결과 값이 10000 이상여야 한다. (오브젝트에러)**

<br/>

### 스프링에서의 Bean Validation

- 스프링부트는 `spring-boot-starter-validation` 라이브러리를 넣으면 **자동으로 Bean Validator를 인지하고 스프링에 통합**한다.
- `LocalValidatorFactoryBean` 은 해당 라이브러리에서 사용되는 검증기 객체이다. **`@NotNull` 등과 같은 애너테이션을 보고 검증을 수행한다.**
- 스프링부트는 `LocalValidatorFactoryBean` 을 **글로벌 검증기**로 등록한다. 그렇기 때문에 모든 컨트롤러에 적용된다.

    > [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndController#13)에서, `@InitBinder` 를 통해 특정 컨트롤러에만 검증기를 적용하는 것과는 상반된다.

- 글로벌 검증기로 등록되기 때문에 **`@Valid` 나 `@Validated` 만 적용하면 된다.**

<br/>

### 의존관계(라이브러리) 추가: `build.gradle` 파일

```html
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

> [https://start.spring.io/](https://start.spring.io/) 에서, Dependencies 에 Validation을 추가해도 된다. (새 프로젝트 시작시)

<br/>

### 에러 코드와 메시지 작성: `errors.properties`

```html
priceAndQuantity=가격 * 수량은 {0}이상이여야 합니다.
```

<br/>

### Bean Validator 적용: `Product` 클래스

- Bean Validator 는 필드에러를 처리하므로, 도메인 모델 클래스에 애너테이션을 적용해야한다.

```java
package prac.myPrac.domain;

import org.hibernate.validator.constraints.Range;
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

/**
 * 상품도메인
 */
public class Product {

    @NotBlank
    private String productName;

    @NotNull
    @Range(min = 1000, max = 10000)
    private Integer productPrice;

    @NotNull
    @Max(100)
    private Integer productQuantity;

	/*
	 * 생성자, getter, setter 생략
	 */
}
```

- `@NotBlank`
    - 빈값 + 공백만 있는 경우를 허용하지 않는다.
- `@NotNull`
    - null을 허용하지 않는다.
- `@Range(min = 1000, max = 10000)`
    - 범위 안의 값이어야 한다.
- `@Max(100)`
    - 최대 100까지만 허용한다.

<br/>

### 컨트롤러: `ProductController` 클래스

```java
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import prac.myPrac.domain.Product;

@Controller
public class ProductController {

	/**
	 * 상품 폼 템플릿을 호출하는 메서드
	 */
    @GetMapping("/product")
    public String viewForm(@ModelAttribute("product") Product product) {
        return "productform";
    }

	/**
	 * 검증 메서드
	 * @Validated 를 통해 검증
	 */
    @PostMapping("/product")
    public String validate(@Validated @ModelAttribute("product") Product product, BindingResult bindingResult) {

		//오브젝트에러 처리
        int priceAndQuantity = product.getProductPrice() * product.getProductQuantity();
        if (priceAndQuantity < 10000) {
            bindingResult.reject("priceAndQuantity", new Object[] {10000}, null);
        }

        if (bindingResult.hasErrors()) {
            return "productform";
        }

		/*
		 * 검증 성공시, 수행할 로직
		 */
    }
}
```

<br/>

### 뷰 템플릿: `productform.html`

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
<h1>상품 입력 폼</h1>
<form th:object="${product}" method="post">

    <div th:if="${#fields.hasGlobalErrors()}">
        <p th:each="error : ${#fields.globalErrors()}" th:text="${error}">
            글로벌 오류 (오브젝트 오류)시, 출력되는 태그
        </p>
    </div>

    상품이름: <input type="text" th:field="*{productName}" placeholder="상품이름" th:errorclass="error-class">
    <div th:errors="*{productName}">
        상품 이름 오류시, 출력되는 태그
    </div> <br/>

    상품가격: <input type="text" th:field="*{productPrice}" placeholder="상품가격" th:errorclass="error-class">
    <div th:errors="*{productPrice}">
        상품 가격 오류시, 출력되는 태그
    </div> <br/>

    상품개수: <input type="text" th:field="*{productQuantity}" placeholder="상품개수" th:errorclass="error-class">
    <div th:errors="*{productQuantity}">
        상품 개수 오류시, 출력되는 태그
    </div> <br/>

    <input type="submit"/>
</form>
</body>
</html>
```

<br/>

### 입력과 결과: 필드에러 1

- 검증할 조건: `productName` 은 비어있으면 안된다. (필드에러)
- 입력

    ![필드에러1 입력](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2012.png)

- 결과

    ![필드에러1 결과](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2013.png)

<br/>

### 입력과 결과: 필드에러 2

- 검증할 조건: `productPrice` 는 null이면 안되고, 1000이상 10000이하 이다. (필드에러)
- 입력

    ![필드에러2 입력](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2014.png)

- 결과

    ![필드에러2 결과](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2015.png)

<br/>

### 입력과 결과: 필드에러 3

- 검증할 조건: `productQuantity` 는 null이면 안되고, 최대 100이다. (필드에러)
- 입력

    ![필드에러3 입력](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2016.png)

- 결과

    ![필드에러3 결과](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2017.png)

<br/>

### 입력과 결과: 오브젝트에러

- 검증할 조건: `productQuantity` * `productPrice` 의 결과 값이 10000 이상여야 한다. (오브젝트에러)
- 입력

    ![오브젝트에러 입력](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2018.png)

- 결과

    ![오브젝트에러 결과](/assets/img/2021-08-14-SPRING_MVC_BeanValidation/Untitled%2019.png)

> 오브젝트 오류는 컨트롤러에서 자체적으로 처리했다. 그 이유에 대해 따로 설명하도록 하겠다.

<br/><br/>

## 검증 순서

1. `@ModelAttribute` : 각각의 필드에 타입 변환을 시도한다.
    - 성공하면 다음으로
    - 실패하면 `typeMismatch` 발생 후, 스프링이 `FieldError` 를 `BindingResult` 에 추가
2. Validator 적용
    - 위 예시코드에서는, `LocalValidatorFactoryBean` 이 적용됐다.

<br/>

### Bean Validation 적용 조건

- (타입오류 없이) 바인딩에 성공한 필드만 Bean Validation 가 적용된다.
- 왜냐하면, 일단 값이 정상적으로 들어와야 해당 값을 검증할 수 있기 때문이다.
- 예시
    - `productName` 에 문자 "A" 입력 ⇒ 타입 변환 성공 ⇒ `productName` 필드에 beanValidation 적용
    - `productPrice` 에 문자 "A" 입력 ⇒ (`@ModelAttribute` 에서) 타입 변환 실패 ⇒ `typeMismatch FieldError` 추가 ⇒ `price` 필드는 BeanValidation 적용 안됨

<br/><br/>

## Bean Validation 메시지 변경

- Bean Validation을 적용하여 검증시, 기본적으로 제공되는 오류 메시지가 출력된다.
- 물론 변경할 수 있다.

<br/>

### MessageCodesResolver

- Bean Validation도 MessageCodesResolver 를 통해 메시지 코드를 생성한다.
- 애너테이션 이름을 가지고 메시지 코드를 생성한다.
- 생성 예시: `@NotBlank`
    1. NotBlank.객체이름.필드이름
    2. NotBlank.필드이름
    3. NotBlank.필드타입
    4. NotBlank
- 변경 예시

    ```html
    #errors.properties 파일
    NotBlank.product.productName=상품명은 필수입니다!
    ```

> [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage#17)에서 MessageCodesResolver에 대해 자세히 다뤘으니 참고하자.

<br/>

### BeanValidation가 메시지를 찾는 순서

1. 생성된 메시지 코드 순서대로 `messageSource` 에서 찾기
    - `messages.properties`
    - `errors.properties` 등
2. 애너테이션의 `message` 속성 사용
    - `@NotBlank(message = "공백일 수 없습니다!")`
3. 라이브러리가 제공하는 기본 값 사용

<br/><br/>

## 오브젝트 에러 처리

위 예시 코드의 컨트롤러 부분을 보자. `validate()` 메서드가 오브젝트 에러를 처리하는 로직이 들어있는 것을 확인할 수 있다. 보통 오브젝트 에러는 해당 예시처럼 처리한다. 왜냐하면, BeanValidation은 필드에러에 한정되어 검증하기 때문이다.

- 필드에러 처리: BeanValidation
- 오브젝트에러 처리: 컨트롤러

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*