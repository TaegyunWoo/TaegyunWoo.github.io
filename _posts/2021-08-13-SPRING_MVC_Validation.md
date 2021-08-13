---
category: Spring-MVC
tags: [스프링, MVC, 검증]
title: "[스프링 - MVC] 검증 - 기초"
date:   2021-08-13 16:00:00 
lastmod : 2021-08-13 16:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 검증: 기본

## 개요

- 검증이란, 서버에 들어온 데이터(HTTP 요청)가 적절한 값인지 검사하는 것이다.
- 검증은 컨트롤러의 중요한 역할 중 하나이다.
- 검증 방법
    - **클라이언트에서 검증하기**
        - 사용자의 사용성이 좋다.
        - 조작이 가능하여 보안에 취약하다.
    - **서버에서 검증하기**
        - 보안성이 좋고, 확실하다.
        - 즉각적인 고객 사용성이 떨어진다.
    - 따라서, 둘을 적절히 섞어서 사용해야하고, 최종적으로 서버 검증은 필수이다.
- API 방식으로 서버가 응답을 한다면, API 스펙을 잘 정의하여 검증 오류를 API 응답 결과에 잘 남겨주어야한다.

<br/><br/>

## 검증 직접 처리 예시

본 예시는 상품을 저장하는 기능에 대한 예시이다.

- 상품 저장 성공

    ![상품 저장 성공](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled.png)

- 검증에 실패하여 상품 저장에 실패

    ![상품 저장 실패](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled%201.png)

<br/><br/>

## BindingResult

### `BindingResult` 란?

- `BindingResult` 는 스프링이 제공하는 객체이며, 검증 오류 처리 방법에 활용된다.

<br/>

### 사용 예시

- 사용자로부터 전달받은 Form 데이터를 담는 객체는 다음과 같다.

    ```java
    public class Item {
    	private String itemName;
    	private String itemColor;
    	private int itemQuantity;
    	
    	//getter, setter, 생성자 생략
    }
    ```

<br/>

- 사용자가 HTML Form 을 통해 다음과 같은 값을 서버에 전송했다고 가정하자.

    ```html
    itemName = "Apple"
    itemColor = "RED"
    itemQuantity = 3
    ```

<br/>

- 이때, 검증 조건은 다음과 같다.
    - **itemName의 값은 빈 문자열이 올 수 없다.**
    - **itemColor의 값은 "GREEN" 이어야 한다.**
    - **만약 itemName이 "Apple"이라면, itemQuantity의 값은 5보다 커야한다.**

<br/>

- 이러한 검증을 조건에 맞추어 하려면 컨트롤러를 다음과 같이 작성해야한다.

    ```java
    @Controller
    public class ItemController {

    		//사용자에게 view를 제공
        @GetMapping("/item")
        public String viewItemForm(@ModelAttribute Item item) {
            return "itemform"; //아이템 입력 폼 호출
        }

    		/**
    		 * 검증 메서드 (with BindingResult)
    		 * 검증하고자 하는 객체: item
    		 * 매개변수 BindingResult는 item 뒤에 위치한다.
    		**/
        @PostMapping("/item")
        public String validation(@ModelAttribute Item item, BindingResult bindingResult) {
    			
    			//첫 번째 검증 조건 (FieldError)
    			if (!StringUtils.hasText(item.getItemName())) {
    				bindingResult.addError(new FieldError("item", "itemName", "아이템 이름은 필수입니다!"));
    			}
    			
    			//두 번째 검증 조건 (FieldError)
    			if (!item.getItemColor().equals("GREEN")) {
    				bindingResult.addError(new FieldError("item", "itemColor", "아이템 색상은 초록색이어야 합니다!"));
    			}

    			//세 번째 검증 조건 (ObjectError)
    			if (!isAppleOver5(item)) {
    				bindingResult.addError(new ObjectError("item", "사과는 5개보다 많아야합니다!"));
    			}

    			//검증 실패시
    			if (bindingResult.hasErrors()) {
    				return "itemform"; //다시 아이템 입력 폼으로
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

계속해서 설명하도록 하겠다.

<br/>

### `BindingResult` 의 특징

- `BindingResult` 은 반드시 검증하고자 하는 객체의 뒤에 작성해야한다.
    - `BindingResult` 는 바로 앞에 작성된 매개변수를 처리한다.
- `BindingResult` 객체는 `Model` 객체에 자동으로 바인딩된다.

<br/>

### FieldError

- 검증하고자 하는 객체의 필드에 오류가 있으면, `FieldError` 객체를 생성해서 `bindingResult` 객체에 담아두면 된다.
- 위에서 작성한 예시

    `bindingResult.addError(new FieldError("item", "itemColor", "아이템 색상은 초록색이어야 합니다!"));`

- `FieldError` 생성자

    ```java
    public FieldError(String objectName, String fieldName, String defaultMessage) {...}
    ```

    - `objectName` : 검증할 객체 이름
    - `fieldName` : 검증할 객체의 필드 이름
    - `defaultMessage` : 오류 기본 메시지

<br/>

### ObjectError

- 특정 필드를 넘어서는 범위의 오류가 있으면, `ObjectError` 객체를 생성해서 `bindingResult` 객체에 담아두면 된다.
- 위에서 작성한 예시
    - `bindingResult.addError(new ObjectError("item", "사과는 5개보다 많아야합니다!"));`
    - 위 경우, `itemName` 필드와 `itemQuantity` 필드를 복잡적으로 고려한 검증이므로, 특정 필드 범주를 벗어난다. 따라서, `ObjectError` 객체를 사용한다.
- `ObjectError` 생성자

    ```java
    public ObjectError(String objectName, String defaultMessage) {...}
    ```

    - `objectName` : 검증할 객체 이름
    - `defaultMessage` : 오류 기본 메시지

<br/>

### 뷰 템플릿 ( `itemform.html` )

위에서 검증을 처리하는 컨트롤러를 작성해보았다. 검증 오류가 발생한다면 `itemForm` 라는 뷰 템플릿을 **다시 호출**하는 로직을 구현하였다. 그렇다면, `itemForm` 템플릿을 어떻게 작성해야 사용자에게 검증오류에 대한 안내를 할 수 있을까.

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
        </div>

        <input type="text" th:field="*{itemColor}" placeholder="아이템색상" th:errorclass="error-class">
        <div th:errors="*{itemColor}">
            아이템 색상 오류시, 출력되는 태그
        </div>

        <input type="text" th:field="*{itemQuantity}" placeholder="아이템개수" th:errorclass="error-class">
				<input type="submit"/>
    </form>
</body>
</html>
```

- `#fields`
    - 컨트롤러에서 `BindingResult` 객체가 자동으로 `model` 객체에 담겨서 뷰 템플릿으로 넘어온다. 여기서 넘어온 `BindingResult` 에 접근할 수 있다.

<br/>

- `th:errors`
    - 해당 필드에 오류가 있는 경우에 태그를 출력한다.
    - 또한, `BindingResult` 객체에 추가된 에러객체 (`FieldError` , `ObjectError`)의 오류 메시지를 출력한다.
    - 즉, `th:if` 와 `th:text` 를 합친 것과 같다.

<br/>

- `th:errorclass`
    - `th:field` 에서 지정한 필드에 오류가 있으면 class 정보를 추가한다.

<br/>

> 검증 오류로 다시 `itemForm` 템플릿을 컨트롤러가 호출한다고 해도, **오류가 발생하지 않은 태그의 value만 그대로 다시 나타난다.** (이후에 자세히 설명한다.)

<br/><br/>

## `@ModelAttribute` 와 `BindingResult`

만약 사용자가 `itemForm` 페이지의 `<input type="text" th:field="*{itemQuantity}" placeholder="아이템개수" ... />` 에 문자를 입력하면 어떻게 될까?

사용자가 `아이템개수` 태그에 문자를 입력한 뒤, 요청을 하게되면 `@ModelAttribute` 를 통해 `item` 객체의 필드에 값을 담는 과정에서 **`int itemQuantity` 필드에 대해 타입오류가 발생한다.**

<br/>

### 타입 오류와 `BindingResult`

- `bindingResult` 를 컨트롤러(의 메서드)에서 사용하지 않는 경우
    - 400 오류가 발생하면서 **컨트롤러(의 메서드)가 호출되지 않고, 오류 페이지로 이동한다.**

<br/>

- `bindingResult` 를 컨트롤러(의 메서드)에서 사용하는 경우
    - 오류 정보( `FieldError` )를 `BindingResult` 에 담아서 **컨트롤러(의 메서드)를 정상 호출한다.**

<br/><br/><br/>

# 검증: 잘못된 입력값 유지

위에서 `BindingResult` 를 활용하여 검증하는 방법을 살펴보았다. **하지만 사용자가 입력한 값에 대해 검증 오류가 발생하여 다시 입력 폼 (`itemform.html`)을 호출했을 때, 문제가 발생한 부분의 값이 출력되지 않는다.** 이처럼, 사용자가 잘못 입력한 값을 다시 출력해주지 않는다면, 사용자에게 혼란을 가져올 수 있다. 따라서, 해당 부분을 개선해보자.

<br/><br/>

## FieldError

먼저 `FieldError` 가 제공하는 생성자들에 대해 알아보자.

<br/>

### 기본 생성자

```java
public FieldError(String objectName, String field, String defaultMessage) {..}
```

<br/>

### 다양한 기능을 포함하는 생성자

```java
public FieldError(String objectName, String field, @Nullable Object rejectedValue
			, boolean bindingFailure, @Nullable String[] codes
			, @Nullable Object[] arguments, @Nullable String defaultMessage) {..}
```

파라미터 목록

- `objectName`
    - 오류가 발생한 객체 이름
- `field`
    - 오류 필드
- `rejectedValue`
    - 사용자가 입력한 값 (거절된 값)
- `bindingFailure`
    - 바인딩 실패 (타입오류 등)인지, 검증 실패인지 구분하는 값
    - 바인딩 자체 실패 (데이터 자체가 안들어오는 경우)⇒ true
    - 단순 검증 실패 ⇒ false
- `codes`
    - 메시지 코드
- `arguments`
    - 메시지에서 사용하는 인자에 넘겨줄 값
- `defaultMessage`
    - 기본 오류 메시지

<br/><br/>

## ObjectError

위 FieldError의 생성자와 유사하다.

<br/><br/>

## 오류 발생시 사용자 입력 값 유지

- 사용자의 입력 데이터가 컨트롤러의 `@ModelAttribute` 에 바인딩되는 시점에 오류 발생시, 모델 객체에 사용자 입력 값을 유지하기 어렵다.
    - (int형 필드에 사용자가 문자를 입력하는 경우)
- 따라서, 입력된 값을 따로 보관할 곳이 필요하다.
- **생성자의 매개변수 `rejectedValue` 에 거절된 값이 들어간다.**
- 위 예시에선, 해당 매개변수를 사용하지 않았으므로 값이 유지가 안되었다.

<br/>

### 기본 생성자 사용시

```java
@PostMapping("/item")
public String validation(@ModelAttribute Item item, BindingResult bindingResult) {

    //첫 번째 검증 조건 (FieldError)
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", "아이템 이름은 필수입니다!"));
    }

    //두 번째 검증 조건 (FieldError)
    if (!item.getItemColor().equals("GREEN")) {
        bindingResult.addError(new FieldError("item", "itemColor", "아이템 색상은 초록색이어야 합니다!"));
    }

    //세 번째 검증 조건 (ObjectError)
    if (!isAppleOver5(item)) {
        bindingResult.addError(new ObjectError("item", "사과는 5개보다 많아야합니다!"));
    }

    //검증 실패시
    if (bindingResult.hasErrors()) {
        return "itemform"; //다시 Item 입력 폼으로
    }
}
```

- 입력

    ![기본 생성자 입력](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled%202.png)

<br/>

- 결과

    ![기본 생성자 결과](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled%203.png)

    - **잘못 입력한 값 "RED"가 유지되지 않는다.**

<br/>

### 다양한 기능의 생성자 사용시

```java
@PostMapping("/item")
public String validationWithWrongValue(@ModelAttribute Item item, BindingResult bindingResult) {

    //첫 번째 검증 조건 (FieldError)
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "아이템 이름은 필수입니다!"));
    }

    //두 번째 검증 조건 (FieldError)
    if (!item.getItemColor().equals("GREEN")) {
        bindingResult.addError(new FieldError("item", "itemColor", item.getItemColor(), false, null, null,"아이템 색상은 초록색이어야 합니다!"));
    }

    //세 번째 검증 조건 (ObjectError)
    if (!isAppleOver5(item)) {
        bindingResult.addError(new ObjectError("item", "사과는 5개보다 많아야합니다!"));
    }

    //검증 실패시
    if (bindingResult.hasErrors()) {
        return "itemform"; //다시 Item 입력 폼으로
    }

    //검증 성공시 로직...
}
```

- `new FieldError("item", "itemColor", item.getItemColor(), false, null, null,"아이템 색상은 초록색이어야 합니다!")`
    - **사용자에게 전달받아서 `item` 객체의 `itemColor` 필드에 바인딩된 값이 검증 실패된 것이므로, `rejectedValue` 로써 `item.getItemColor()` 가 들어갔다.**

    > 타입 오류의 경우, `item` 객체에 바인딩될 수 없는 값은 어떻게 처리될까? 마지막 부분에서 설명하겠다.

<br/>

- 입력

    ![기능 생성자 입력](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled%204.png)

<br/>

- 결과

    ![기능 생성자 결과](/assets/img/2021-08-13-SPRING_MVC_Validation/Untitled%205.png)

    - **잘못 입력한 값 "RED"가 유지된다. "RED" 라는 값이 `rejectedValue` 에 전달되어, 타임리프가 해당 값을 출력한 것이다.**

<br/>

### 타임리프의 사용자 입력 값 유지

- 타임리프의 `th:field` 동작 방식
    - 정상 상황
        - 모델 객체의 필드 값을 사용한다.
    - 오류 상황
        - `FieldError` 에서 보관한 값 (`rejectedValue`)를 사용해서 값을 출력한다.
        - 따라서, 기본 생성자 (`rejectedValue` 사용 X)를 통해 `FieldError` 객체 생성시 아무 값도 출력되지 않았다.

<br/>

### 스프링의 바인딩 오류 처리

- **타입 오류와 같이 바인딩에 실패하는 상황시, 스프링은 자동으로 `FieldError` 객체를 생성하고 사용자가 입력한 값을 `rejectedValue`에 넣어둔다.**
- **그리고 해당 `FieldError` 객체를 `BindingResult` 에 담아서 컨트롤러(메서드)를 호출한다.**
- 따라서, 타입 오류가 발생하여 바인딩에 실패하더라도, 오류를 정상적으로 해결할 수 있다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*