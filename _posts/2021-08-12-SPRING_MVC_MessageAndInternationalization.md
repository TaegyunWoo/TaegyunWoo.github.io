---
category: Spring-MVC
tags: [스프링, MVC, 메시지, 국제화]
title: "[스프링 - MVC] 메시지와 국제화"
date:   2021-08-12 18:00:00 
lastmod : 2021-08-12 18:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 메시지와 국제화

## 개요

### 메시지란?

- 여러 뷰 템플릿에서 공통적으로 사용되는 단어(메시지)를 **한 곳에서 관리** 할 수 있게 해주는 기능이다.
- 스프링은 기본적인 메시지 기능을 지원한다.

</br>

### 국제화란?

- 메시지 파일 ( `messages.properties` )를 각 나라별로 별도로 관리하면 구현할 수 있다.
- 사용자의 국가 정보는 `accept-langauge` 헤더 값을 통해 얻어올 수 있다. 이러한 정보를 통해, 적절한 메시지 파일을 선택하여 사용자에게 제공할 수 있다.
- 스프링은 기본적인 국제화 기능 역시 지원한다.

</br></br>

## 메시지 소스 설정

메시지 기능은 (key,value) 형식으로 작성된 파일을 기반으로 동작한다. 파일 형식의 예시는 아래와 같다.

```html
#messages.properties 파일

item=상품
item.id=상품 ID
item.itemName=상품명
item.price=가격
item.quantity=수량
```

</br>

### 스프링에서의 설정 방법 (직접 등록)

- 스프링에서 메시지 관리 기능을 사용하려면 `MessageSource` 인터페이스의 구현체를 스프링 빈으로 등록해야한다.
    - `MessageSource` 인터페이스의 구현체 = `ResourceBundleMessageSource` 클래스

</br>

```java
@Bean
public MessageSource messageSource() {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		messageSource.setBasenames("messages", "errors");
		messageSource.setDefaultEncoding("utf-8");
		return messageSource;
}
```

- `messageSource.setBasenames("messages", "errors")`
    - 설정 파일의 이름을 지정한다.
    - 매개변수 `messages` , `errors`
        - `messages.properties` 파일과 `errors.properties` 을 읽어서 사용한다.
        - 여러 파일을 한번에 지정할 수 있다.
    - 국제화 기능을 적용하려면, `messages_en.properties` , `messages_ko.properties` 와 같이 파일명을 설정하면 된다.
        - 만약 찾을 수 있는 국제화 파일이 없으면, `messages.properties` 파일을 기본적으로 사용한다.
    - 메시지 파일의 위치
        - `/main/resources/messages.properties`
        - `/main/resources/messages_en.properties`
        - `/main/resources/messages_ko.properties`

</br>

- `messageSource.setDefaultEncoding("utf-8")`
    - 인코딩 정보를 지정한다.

</br>

### 스프링 부트에서의 설정 방법

- 스프링 부트에서는 `MessageSource` 인터페이스의 구현체인 `ResourceBundleMessageSource` 클래스를 **빈으로 자동 등록** 한다.

- 따라서, 메시지 소스에 대한 세부 설정은 `application.properties` 에 작성하면 된다.
- 스프링 부트에서 `MessageSource` 를 스프링 빈으로 따로 등록하지 않고, 스프링 부트와 관련된 별도의 설정 ( `application.properties` 파일에 설정 )을 하지 않으면, **`messages` 라는 이름이 기본적으로 등록된다.**

```html
#application.properties 파일

spring.messages.basename=myMessages
```

- 위와 같이 설정하면, `myMessages.properties` 파일을 메시지 파일로 처리한다. (필수로 설정해야하는 것은 아니다.)

</br></br>

## 메시지 파일 만들기

- 메시지 파일을 만들고 내용을 작성하여, 메시지 기능을 적용해보자.
- 국제화 테스트를 위해서 `messages_en` 파일도 추가할 것이다.

</br>

### `messages.properties` 메시지 파일

```html
hello=안녕
hello.name=안녕 {0}
```

- `messages.properties` 파일은 기본 값으로 사용된다.
- 파일 경로는 `/main/resources/messages.properties` 이다.
- `{0}` 은 인수로 넘어온 값의 위치이다. 뒤에서 자세하게 설명하도록 하겠다.

</br>

### `messages_en.properties` 메시지 파일

```html
hello=hi
hello.name=hi {0}
```

- `messages_en.properties` 파일은 영어권 사용자에 대해, 적용되는 메시지 파일이다.
- 파일 경로는 `/main/resources/messages_en.properties` 이다.

</br></br>

## 타임리프 템플릿에서 메시지 출력하기

- 타임리프의 메시지 표현식 `#{...}` 을 사용하면 스프링의 메시지를 조회할 수 있다.
- 메시지 파일은 위에서 작성한 `messages.properties` 와 `messages_en.properties` 를 사용한다.

</br>

### 타임리프 템플릿

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
</head>
<body>
	메시지 hello= <p th:text="#{hello}"></p>
	메시지 hello.name= <p th:text="#{hello.name('전달될_값')}"></p>
</body>
</html>
```

</br>

### 결과

```html
메시지 hello= 안녕
메시지 hello.name= 안녕 전달될_값
```

- 이때, 적용되는 것은 `messages.properties` 파일이다.

> `messages_en.properties` 를 적용해보고 싶다면, postman 등을 통해 요청메시지의 `accept-langauge` 헤더 값을 변경하여 확인해보자!

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*