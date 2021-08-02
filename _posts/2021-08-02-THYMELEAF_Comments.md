---
category: Thymeleaf
tags: [타임리프, 주석처리]
title: "[타임리프] 주석처리의 다양한 방법"
date:   2021-08-02 13:30:00 
lastmod : 2021-08-02 13:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 주석 처리

타임리프는 여러가지 방식의 주석처리를 지원한다. 예시 코드로 알아보자.

<br><br>

## 예시코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/comments")
	public String comments(Model model) {
		model.addAttribute("data", "Spring!");
		return "basic/comments";
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

	<h1>예시</h1>
	<span th:text="${data}">html data</span>
	<h1>1. 표준 HTML 주석</h1>
	<!--
	<span th:text="${data}">html data</span>
	-->
	

	<h1>2. 타임리프 파서 주석</h1>
	<!--/* [[${data}]] */-->
	
	<!--/*-->
	<span th:text="${data}">html data</span>
	<!--*/-->

	<h1>3. 타임리프 프로토타입 주석</h1>
	<!--/*/
	<span th:text="${data}">html data</span>
	/*/-->
	
</body>
</html>
```

아래에서 자세히 설명하도록 하겠다.

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_Comments/Untitled%2014.png)

<br><br>

## 상세 설명

### HTML 주석

- `<!-- 주석내용 -->`
    - HTML이 제공하는 기본적인 주석처리 방법이다.

<br>

### 타임리프 파서 주석

- `<!--/* 주석내용 */ -->`
    - 타임리프가 제공하는 주석이다.
    - 렌더링을 할 때, 해당 주석 부분을 제거한다.

<br>

### 타임리프 프로토타입 주석

- `<!--/*/ 주석내용 /*/-->`
    - 타임리프가 제공하는 주석이다.
    - **HTML 파일을 그대로 열어보면 (직접 파일을 열었을 때) 주석처리가 되지만, 타임리프를 렌더링 한 경우에만 주석처리가 되지 않는 기능이다.**
    - 즉, 타임리프로 렌더링된 경우에는 주석처리가 안된다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*