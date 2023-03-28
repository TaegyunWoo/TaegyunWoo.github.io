---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 타임리프가 제공하는 객체"
date:   2021-08-01 19:30:00 
lastmod : 2021-08-01 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 기본 객체

## 기본 객체

타임리프는 아래와 같은 기본 객체들을 제공한다.

- `${#request}`
- `${#response}`
- `${#session}`
- `${#servletContext}`
- `${#locale}`

<br><br>

## 편의 객체

타임리프는 기본 객체 뿐만 아니라, 보다 편리한 편의 객체도 제공한다.

<br>

### HTTP 요청 파라미터 접근

- `${param.파라미터이름}`

해당 객체를 통해, 보다 편리하게 요청 파라미터에 접근할 수 있다.

<br>

### HTTP 세션 접근

- `${session.세션데이터이름}`

해당 객체를 통해 세션데이터에 접근할 수 있다.

<br>

### 스프링 빈 접근

- `${@빈객체이름.메소드이름()}`

이와 같이, 빈 객체에도 접근할 수 있다.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/basic-objects")
	public String basicObjects(HttpSession session) {
		session.setAttribute("sessionData", "Hello Session");
		return "basic/basic-objects";
	}

	//빈 객체 등록
	@Component("helloBean")
	static class HelloBean {
		public String hello(String data) {
			return "Hello " + data;
		}
	}
	
}
```

<br>

### 타임리프

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8">
	<title>Title</title>
</head>
<body>
	<h1>식 기본 객체 (Expression Basic Objects)</h1>
	<ul>
		<li>request = <span th:text="${#request}"></span></li>
		<li>response = <span th:text="${#response}"></span></li>
		<li>session = <span th:text="${#session}"></span></li>
		<li>servletContext = <span th:text="${#servletContext}"></span></li>
		<li>locale = <span th:text="${#locale}"></span></li>
	</ul>

	<h1>편의 객체</h1>
	<ul>
		<li>Request Parameter = <span th:text="${param.paramData}"></span></li>
		<li>session = <span th:text="${session.sessionData}"></span></li>
		<li>spring bean = <span th:text="${@helloBean.hello('Spring!')}"></span></li>
	</ul>

</body>
</html>
```

<br>

### 결과

![결과](/assets/img/2021-08-01-THYMELEAF_BasicObjects/Untitled%203.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*