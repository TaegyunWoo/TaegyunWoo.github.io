---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 기본 표현식: text와 utext"
date:   2021-08-01 16:30:00 
lastmod : 2021-08-01 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 개요

간단히 기본 표현식의 종류를 살펴보고, 하나씩 본격적으로 설명하도록 하겠다. 이번 게시글에서는 `text` 와 `utext` 를 다룬다.

<br><br>

## 기본 표현식 종류

### 간단한 표현

- **${...}**
    - 변수 표현식
- ***{...}**
    - 선택 변수 표현식
- **#{...}**
    - 메시지 표현식
- **@{...}**
    - 링크 URL 표현식
- **~{...}**
    - 조각 표현식

<br>

### 리터럴

- **'one text'**, **'Another one!'**
    - 텍스트 리터럴
- **0**, **34**, **3.0**, **12.3**
    - 숫자
- **true**, **false**
    - 불린
- **null**
    - 널
- **one**, **sometext**, **main**
    - 리터럴 토큰

<br>

### 문자 연산

- **+**
    - 문자 합치기
- **|The name is ${name}|**
    - 리터럴 대체 (파이프)

<br>

### 산술 연산

- **+**, **-**, **\***, **/**, **%**
    - Binary operators
- **-**
    - Minus sign

<br>

### 불린 연산

- **and**, **or**
    - Binary operators
- **!**, **not**
    - Boolean negation

<br>

### 비교와 동등

- **>**, **<**, **>=**, **<=** (**gt**, **lt**, **ge**, **le**)
    - 비교
- **==**, **!=** (**eq**, **ne**)
    - 동등 연산

<br>

### 조건 연산

- **(조건식) ? (true일때 수행할 내용)**
- **(조건식) ? (true일때 수행할 내용) : (false일때 수행할 내용)**
- **(조건식) ?: (기본)**

<br>

### 특별한 토큰

- **_**
    - No-Operation

<br><br><br>

# 텍스트

## text

### 사용방법

HTML의 콘텐츠에 데이터를 출력하는 방법

- `th:text`
    - 예시) `<span th:text="'MyText!'">`
- `[[...]]`
    - 예시) `<p>[[${myVar}]]</p>`

<br>

### 사용 예시

- 컨트롤러

    ```java
    @Controller
    @RequestMapping("/basic")
    public class BasicController {

    	@GetMapping("/text-basic")
    	public String textBasic(Model model) {
    		model.addAttribute("data", "Hello Spring!");
    		return "basic/text-basic";
    	}

    }
    ```

- 타임리프

    ```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    	<meta charset="UTF-8">
    	<title>Title</title>
    </head>
    <body>
    	<h1>컨텐츠에 데이터 출력하기</h1>
    	<ul>
    		<li>th:text 사용 <span th:text="${data}"></span></li>
    		<li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>
    	</ul>
    </body>
    </html>
    ```

- 결과

    ![text결과](/assets/img/2021-08-01-THYMELEAF_TextUtext/Untitled.png)

<br>

### 이스케이프 문제

- `th:text` 속성을 통해 설정한 문자열을 통해 HTML 태그를 적용하고 싶다면 어떻게 해야할까?
- 예를 들어, `th:text="'Hello <b>Spring!</b>'"` 처럼 설정하여 `<b>` 태그를 적용하려했을 때, 결과는 다음과 같다.
    - `Hello <b>Spring!</b>`
- 즉, `<` 와 `>` 가 모두 이스케이프 문자로 처리되어 그대로 문자로 출력된다. 이는 `[[...]]` 를 사용해도 마찬가지이다.
- **따라서, 이스케이프 처리를 하지 않으려 할 때는** **`th:utext`** **속성을 사용해야한다.**

<br><br>

## utext

### 사용방법

데이터를 이스케이프 문자처리를 적용하지 않고 출력하는 방법

- `th:utext`
    - 예시) `<span th:utext="'MyText!'">`
- `[(...)]`
    - 예시) `<p>[(${myVar})]</p>`

<br>

### 사용 예시

- 컨트롤러

    ```java
    @Controller
    @RequestMapping("/basic")
    public class BasicController {

    	//이스케이프 처리 O
    	@GetMapping("/text-basic")
    	public String textBasic(Model model) {
    		model.addAttribute("data", "Hello Spring!");
    		return "basic/text-basic";
    	}

    	//이스케이프 처리 X
    	@GetMapping("/text-unescaped")
    	public String textUnescaped(Model model) {
    		model.addAttribute("data", "Hello <b>Spring!</b>");
    		return "basic/text-unescaped";
    	}
    }
    ```

- 타임리프

    ```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    	<meta charset="UTF-8">
    	<title>Title</title>
    </head>
    <body>
    	<h1>text vs utext</h1>
    	<ul>
    		<li>th:text = <span th:text="${data}"></span></li>
    		<li>th:utext = <span th:utext="${data}"></span></li>
    	</ul>

    	<h1><span th:inline="none">[[...]] vs [(...)]</span></h1>
    	<ul>
    		<li><span th:inline="none">[[...]] = </span>[[${data}]]</li>
    		<li><span th:inline="none">[(...)] = </span>[(${data})]</li>
    	</ul>
    </body>
    </html>
    ```

    - 참고: `th:inline="none"` 는 해당 태그를 타임리프가 해석하지 말라는 옵션이다.
- 결과

    ![utext결과](/assets/img/2021-08-01-THYMELEAF_TextUtext/Untitled%201.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*