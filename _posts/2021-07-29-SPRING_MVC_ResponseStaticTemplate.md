---
category: Spring-MVC
tags: [스프링, MVC, 정적리소스, 뷰템플릿]
title: "[스프링 - MVC] 정적 리소스와 뷰 템플릿으로 응답하기"
date:   2021-07-29 14:30:00 
lastmod : 2021-07-29 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 개요

## 응답 데이터의 종류

- **정적 리소스**
    - HTML, CSS, JS
- **뷰 템플릿 사용**
    - 동적인 HTML
- **HTTP 메시지 사용**
    - HTTP API를 제공하는 경우, JSON과 같은 형식

서버에서 응답 데이터를 만드는 방법은 위와 같이 크게 3가지가 존재한다. 본 게시글에서는 정적 리소스와 뷰 템플릿으로 응답하는 방법을 살펴본다.

<br><br><br>

# 정적 리소스

## 정적 리소스 디렉터리 종류

- `/static`
- `/public`
- `/resource`
- `/META-INF/resources`

스프링 부트는 위 4개의 디렉터리에 있는 정적 리소스를 제공한다.

<br><br>

## `src/main/resources` 디렉터리

`src/main/resources` 는 리소스를 보관하는 곳이며, 클래스패스의 시작경로이다. 따라서, 해당 디렉터리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

<br>

### 정적 리소스 경로

`src/main/resources` 는 정적 리소스 경로이다. `src/main/resources/static/basic/hello-form.html` 파일을 웹 브라우저에서 호출하려면 `http://localhost:8080/basic/hello-form.html` 으로 요청하면 된다.

<br><br><br>

# 뷰 템플릿

뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.

<br><br>

## `src/main/resources/templates` 디렉터리

`src/main/resources/templates` 는 뷰 템플릿 파일을 저장하는 디렉터리이다. 뷰 템플릿을 호출해서 응답하는 방법을 설명하도록 하겠다.

<br><br>

## 뷰 템플릿 호출

### 뷰 템플릿 생성

`src/main/resources/templates/response/hello.html` 파일

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8"> <title>Title</title>
</head>
<body>
	<p th:text="${data}">empty</p>
</body>
</html>
```

- `th:text="${data}"`
    - 해당 태그 (p 태그)의 text 부분을 data라는 바인딩 된 애트리뷰트의 값으로 치환한다.

    > 뷰템플릿에 대한 자세한 설명은 따로 정리하도록 하겠다.

<br>

### 뷰 템플릿을 호출하는 컨트롤러

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ResponseViewController {

	@RequestMapping("/response-view-v1")
	public ModelAndView responseViewV1() {
		ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
		return mav;
	}

	@RequestMapping("/response-view-v2")
	public String responseViewV2(Model model) {
		model.addAttribute("data", "hello!!");
		return "response/hello";
	}

	@RequestMapping("/response/hello")
	public void responseViewV3(Model model) {
		model.addAttribute("data", "hello!!");
	}

}
```

- `responseViewV1` 메서드
    - `ModelAndView("response/hello")`
        - 뷰의 논리 이름인 `"response/hello"` 를 넘겨준다.
    - `addObject("data", "hello!")`
        - 뷰로 설정될 파일에 `data=hello!` 를 바인딩한다.
    - `return mav;`
        - ModelAndView 객체를 통해 뷰를 호출할 수 있도록 한다.

<br>

- `responseViewV2` 메서드
    - `model.addAttribute("data", "hello!!")`
        - `data=hello!` 를 바인딩한다.
    - `return "response/hello";`
        - 뷰의 논리이름인 `"response/hello"` 를 스프링에게 전달하여 뷰를 호출할 수 있도록 한다.
        - 참고로, 뷰 리졸버를 통해 실제 물리 경로로 변환된다.

<br>

- `responseViewV3`
    - 리턴 값이 없고, `@Controller` 를 사용하고, HTTP 메시지 바디를 처리하는 파라미터가 없다.
        - 따라서, `@RequestMapping("/response/hello")` 의 url인 `/response/hello` 를 논리 경로로 하는 뷰를 호출한다.
    - 비추천 방식이다.

<br>

### 뷰의 논리이름을 실제 물리 경로로 변환하기

메서드의 리턴 타입이 String 라면, **반환된 문자열을 뷰의 논리이름으로 설정** 한다는 것이다. 그렇다면 뷰 리졸버는 이것을 어떻게 물리 경로로 변환하는 것일까?  

**뷰 리졸버는** `application.properties` **파일에 설정되어있는 값을 토대로 물리 경로로 변환** 한다. 그리고 **스프링에 설정되어 있는 기본 값이 바로** `templates/논리이름.html` **이다.** 따라서, 자동적으로 변환할 수 있는 것이다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*