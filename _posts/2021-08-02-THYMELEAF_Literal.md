---
category: Thymeleaf
tags: [타임리프, 문자리터럴]
title: "[타임리프] 타임리프의 문자 리터럴"
date:   2021-08-02 10:30:00 
lastmod : 2021-08-02 10:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 문자 리터럴

## 개요

- 리터럴은 소스 코드상에 고정된 값을 말하는 용어이다.

> 쉬워보이지만 실수를 많이 할 수 있으므로 잘 기억해두자!

<br><br>

## 리터럴 종류

- 문자 리터럴
    - 'hello'
- 숫자 리터럴
    - 10
- 불린 리터럴
    - true, false
- null 리터럴
    - null

<br><br>

## 문자 리터럴

- 타임리프에서 문자 리터럴은 항상 `'` (작은 따옴표)로 감싸야한다.
- 하지만, 생략 가능한 경우가 있다.

<br>

### 작은 따옴표 생략

- 문자 리터럴이 **공백 없이 쭉 이어진다면** 하나의 의미있는 토큰으로 타임리프가 인지한다.
- 문자 토큰으로 인지하는 문자들
    - 알파벳
    - 숫자
    - `[` , `]` (대괄호)
    - `.` (마침표)
    - `-`
    - `_` (언더바)
- 생략 예시
    - 원칙상 표현방법
        - `th:text="'hello'"`
    - 작은 따옴표 생략 표현방법
        - `th:text="hello"`
    - 생략이 불가능한 경우
        - `th:text="hello world"` ⇒ 불가능!

계속해서 예시 코드를 통해 설명하도록 하겠다.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/literal")
	public String literal(Model model) {
		model.addAttribute("data", "Spring!");
		return "basic/literal";
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
	<h1>리터럴</h1>
	<ul>
	<!--주의! 다음 주석을 풀면 예외가 발생함-->
	<!--    <li>"hello world!" = <span th:text="hello world!"></span></li>-->
	<li>'hello' + ' world!' = <span th:text="'hello' + ' world!'"></span></li>
	<li>'hello world!' = <span th:text="'hello world!'"></span></li>
	<li>'hello ' + ${data} = <span th:text="'hello ' + ${data}"></span></li>
	<li>리터럴 대체 |hello ${data}| = <span th:text="|hello ${data}|"></span></li>
	</ul>
</body>
</html>
```

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_Literal/Untitled%209.png)

<br><br>

## 리터럴 대체

- `<span th:text="|hello ${data}|"></span>`
- 위 예시처럼 `|` (수직선 기호)를 사용하면 `+` 연산자를 통해 문자열을 연결하지 않아도 된다.

> Javascript의 ES6에서 추가된 문법 ` (역따옴표)와 같은 기능을 한다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*