---
category: Thymeleaf
tags: [타임리프, 템플릿, 레이아웃]
title: "[타임리프] 템플릿 레이아웃"
date:   2021-08-02 20:00:00 
lastmod : 2021-08-02 20:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 템플릿 레이아웃

[이전 글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_TemplateFragment)에서 일부 코드 조각을 가지와서 사용하는 템플릿 조각에 대해 알아보았다. 이번 글에서는 개념을 좀 더 확장시켜, 코드 조각을 레이아웃에 넘겨서 사용하는 방법에 대해 설명하도록 하겠다.

예를 들어, 공통 정보들은 한 곳에 모아두고 공통으로 사용하지만, 각 페이지마다 필요한 정보를 더 추가해서 사용해야하는 경우가 있다. 이런 경우, 템플릿 레이아웃을 활용하여 구현하는 것이 적합하다.

바로 예시 코드를 보며 설명하도록 하겠다.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/template")
public class TemplateController {

	@GetMapping("/layout")
	public String layout() {
		return "template/layout/layoutMain";
	}

}
```

<br>

### 템플릿 레이아웃으로 사용될 HTML

> `<head>` 부분을 메인HTML이 가져와 사용할 것이다. 그러므로 `<head>` 부분을 집중해서 보도록 하자!

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="common_header(title,links)">

	<title th:replace="${title}">레이아웃 타이틀</title>

	<!-- 공통 정보 -->
	<link rel="stylesheet" type="text/css" media="all" th:href="@{/css/ awesomeapp.css}">
	<link rel="shortcut icon" th:href="@{/images/favicon.ico}">
	<script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>
	
	<!-- 추가 정보가 들어갈 자리 -->
	<th:block th:replace="${links}" />
	
</head>
<body>

</body>
</html>
```

<br>

### 메인 HTML

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="template/layout/base :: common_header(~{::title},~{::link})">

	<title>메인 타이틀</title>
	<link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
	<link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">

</head>
<body>
	메인 컨텐츠
</body>
</html>
```

<br>

### 결과 - 메인HTML

```html
<!DOCTYPE html>
<html>
<head>

	<title>메인 타이틀</title>

	<!-- 공통 정보 -->
	<link rel="stylesheet" type="text/css" media="all" href="/css/awesomeapp.css">
	<link rel="shortcut icon" href="/images/favicon.ico">
	<script type="text/javascript" src="/sh/scripts/codebase.js"></script>

	<!-- 추가 정보가 들어갈 자리 -->
	<link rel="stylesheet" href="/css/bootstrap.min.css">
	<link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">

</head>
<body>
	메인 컨텐츠
</body>
</html>
```

<br><br>

## 상세 설명

### 템플릿 레이아웃으로 사용될 HTML

- `<head th:fragment="common_header(title,links)"> ... </head>`
    - `<head>` 태그와 내부 태그들을 템플릿으로 삼는다.
    - 템플릿 이름: `common_header`
    - 매개변수: `title` , `links`

<br>

- `<title th:replace="${title}">레이아웃 타이틀</title>`
    - 매개변수 `title` 에 담긴 값으로 `<title>` 태그를 교체한다.
    - **템플릿 조각** 으로 `th:replace` 를 설정했을 때
        - 템플릿 조각의 경우, 오직 메인HTML에만 `th:replace` 이 적용되었고 `~{}` 표현식을 통해 설정하였다.
        - 메인HTML에 적용된 `th:replace` : `th:replace="~{템플릿경로 :: 조각이름}"`
    - **템플릿 레이아웃** 으로 `th:replace` 를 설정했을 때
        - 템플릿 레이아웃의 경우, 메인HTML 뿐만 아니라 템플릿 레이아웃 HTML에도 `th:replace` 이 적용된다.
        - 템플릿 레이아웃 HTML에 적용된 `th:replace` : `th:replace="${매개변수명}"`

이후 설명을 읽고나면, 보다 이해가 수월할 것이다.

<br>

### 메인HTML

- `<head th:replace="template/layout/base :: common_header(~{::title},~{::link})">`
    - `th:replace="template/layout/base :: common_header(~{::title},~{::link})"` 을 통해, `<head>` 태그 전체를 `common_header` 템플릿으로 변경할 것임을 알 수 있다.
    - **`~{::title}` 매개변수**
        - `<head>` 태그 내부에 존재하는 `<title>` 태그를 템플릿 레이아웃 매개변수에 데이터로 넘겨준다.
    - **`~{::link}` 매개변수**
        - `<head>` 태그 내부에 존재하는 `<link>` 태그를 템플릿 레이아웃 매개변수에 데이터로 넘겨준다.
        - 여러 개의 `<link>` 태그가 존재하지만, 모두 넘어간다.
    - 데이터를 넘겨받은 템플릿 레이아웃의 형태는 아래와 같다.

        ```html
        <!---------------------- 기존 코드 ----------------------->
        <head th:fragment="common_header(title,links)">

        	<title th:replace="${title}">레이아웃 타이틀</title>

        	<!-- 공통 정보 -->
        	<link rel="stylesheet" type="text/css" media="all" th:href="@{/css/ awesomeapp.css}">
        	<link rel="shortcut icon" th:href="@{/images/favicon.ico}">
        	<script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>
        	
        	<!-- 추가 정보가 들어갈 자리 -->
        	<th:block th:replace="${links}" />
        	
        </head>
        <!------------------------------------------------------------------------->

        <!------------------------ 매개변수가 적용된 코드 -------------------------->
        <head th:fragment="common_header(title,links)">

        	<title>메인 타이틀</title>

        	<!-- 공통 정보 -->
        	<link rel="stylesheet" type="text/css" media="all" th:href="@{/css/ awesomeapp.css}">
        	<link rel="shortcut icon" th:href="@{/images/favicon.ico}">
        	<script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>
        	
        	<!-- 추가 정보가 들어갈 자리 -->
        	<link rel="stylesheet" href="/css/bootstrap.min.css">
        	<link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">
        	
        </head>
        <!------------------------------------------------------------------------->
        ```

    - 그리고 여기서 끝나는 것이 아니라, 메인HTML에도 변경된 템플릿 레이아웃을 적용(`th:replace`) 해야한다. 그 결과는 다음과 같다.

        ```html
        <!DOCTYPE html>
        <html>
        <head>

        	<title>메인 타이틀</title>

        	<!-- 공통 정보 -->
        	<link rel="stylesheet" type="text/css" media="all" href="/css/awesomeapp.css">
        	<link rel="shortcut icon" href="/images/favicon.ico">
        	<script type="text/javascript" src="/sh/scripts/codebase.js"></script>

        	<!-- 추가 정보가 들어갈 자리 -->
        	<link rel="stylesheet" href="/css/bootstrap.min.css">
        	<link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">

        </head>
        <body>
        	메인 컨텐츠
        </body>
        </html>
        ```

<br><br>

## 적용 범위의 확대

위에서 설명한 것은 `<head>` 태그와 같이 HTML의 일부에만 국한되었다. 하지만 `<html>` 태그에 템플릿 레이아웃을 적용하여, 전체 HTML을 대상으로 설정할 수 있다.

<br>

### 컨트롤러

```java
@Controller
@RequestMapping("/template")
public class TemplateController {

	//기존 <head>에 적용했던 예시의 메서드
	@GetMapping("/layout")
	public String layout() {
		return "template/layout/layoutMain";
	}

	//전체 html에 적용하는 예시 호출
	@GetMapping("/layoutExtend")
	public String layoutExtends() {
		return "template/layoutExtend/layoutExtendMain";
	}

}
```

<br>

### 템플릿 레이아웃으로 사용될 HTML

```html
<!DOCTYPE html>
<html th:fragment="layout (title, content)"
	xmlns:th="http:// www.thymeleaf.org">
<head>
	<title th:replace="${title}">레이아웃 타이틀</title>
</head>
<body>

	<h1>레이아웃 H1</h1>

	<div th:replace="${content}">
		<p>레이아웃 컨텐츠</p>
	</div>

	<footer>
	레이아웃 푸터
	</footer>

</body>
</html>
```

<br>

### 메인 HTML

```html
<!DOCTYPE html>
<html th:replace="~{template/layoutExtend/layoutFile :: layout(~{::title}, ~{::section})}"
	xmlns:th="http://www.thymeleaf.org">
<head>
	<title>메인 페이지 타이틀</title>
</head>
<body>
	<section>
		<p>메인 페이지 컨텐츠</p>
		<div>메인 페이지 포함 내용</div>
	</section>
</body>
</html>
```

<br>

### 결과 - 메인HTML

```html
<!DOCTYPE html>
<html>
<head>
	<title>메인 페이지 타이틀</title>
</head>
<body>
	<h1>레이아웃 H1</h1>
	<section>
		<p>메인 페이지 컨텐츠</p>
		<div>메인 페이지 포함 내용</div>
	</section>
	<footer> 레이아웃 푸터 </footer>
</body>
</html>
```

개념적으로 위에서 설명한 것과 크게 다르지 않다. 단지, 적용 범위가 `<html>` 로 더 확대된 것 뿐이다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*