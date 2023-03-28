---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 템플릿 조각"
date:   2021-08-02 19:00:00 
lastmod : 2021-08-02 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 템플릿 조각

## 개요

웹 페이지 개발시, 공통 영역 다수 존재한다. (예를 들어, footer나 카테고리 와 같은 영역) 타임리프에서는 보다 편리하게 공통 영역을 처리하는 기능을 제공한다. 예시 코드를 통해 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

> 이전에 작성한 게시글에서 사용한 `BasicController` 클래스가 아닌 새로운 클래스이다.

```java
@Controller
@RequestMapping("/template")
public class TemplateController {

	@GetMapping("/fragment")
	public String template() {
		return "template/fragment/fragmentMain";
	}

}
```

<br>

### 템플릿 조각으로 사용될 HTML

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8">
	<title>Title</title>
</head>
<body>

	<footer th:fragment="copy">
		푸터 자리 입니다.
	</footer>

	<footer th:fragment="copyParam (param1, param2)">
		<p>파라미터 자리 입니다.</p>
		<p th:text="${param1}"></p>
		<p th:text="${param2}"></p>
	</footer>

</body>
</html>
```

<br>

### 메인 HTML

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8">
	<title>Title</title>
</head>
<body>

	<h1>부분 포함</h1>
	<h2>부분 포함 insert</h2>
	<div th:insert="~{template/fragment/footer :: copy}"></div>

	<h2>부분 포함 replace</h2>
	<div th:replace="~{template/fragment/footer :: copy}"></div>

	<h2>부분 포함 단순 표현식</h2>
	<div th:replace="template/fragment/footer :: copy"></div>

<!------------------------------------------------->

	<h1>파라미터 사용</h1>
	<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터 2')}"></div>

</body>
</html>
```

상당히 복잡해보이지만, 한번 이해하고 나면 그리 어렵지 않다. 하나씩 살펴보자.

<br><br>

## 상세 설명

### 템플릿 조각으로 사용될 HTML

- `<footer th:fragment="copy"> ... </footer>`
    - `th:fragment="템플릿_조각_이름"` 을 통해, **해당 태그와 하위 태그들이 모두 템플릿 조각으로 사용될 것** 임을 나타낸다. 설정한 템플릿 조각 이름을 통해, 다른 HTML에서 해당 조각을 사용할 수 있다.

<br>

- `<footer th:fragment="copyParam (param1, param2)"> ... </footer>`
    - `th:fragment="copyParam (param1, param2)"` 을 사용하여, **템플릿 조각으로 설정함과 동시에 '해당 조각을 사용할 메인HTML'으로부터 변수값을 전달받을 수 있다.**
    - 매개변수는 `(param1, param2)` 과 같다. 즉, 2개의 매개변수가 존재한다.

계속해서 상세히 알아보자.

<br>

### 메인 HTML

- `<div th:insert="~{template/fragment/footer :: copy}"></div>`
    - `th:insert="~{template/fragment/footer :: copy}"` 를 통해, **'템플릿 조각이 존재하는 HTML의 경로'와 '전달받은 템플릿 조각의 이름'을 설정한다.**
    - 즉, `th:insert="~{전달받을_템플릿_경로 :: 템플릿_이름}"` 형식으로 이해하면 된다.

        > 참고로, 상대 경로로 설정시 시작 디렉터리는 `src\main\resources\templates` 이다.

    - `th:insert` 는 **본인 태그 내부에 템플릿을 넣을 것임** 을 의미한다.
    - 따라서, 메인HTML의 해당부분에 대한 결과는 다음과 같다.

        ```html
        <div>
        	<footer>
        		푸터 자리 입니다.
        	</footer>
        </div>
        ```

<br>

- `<div th:replace="~{template/fragment/footer :: copy}"></div>`
    - `th:replace="~{template/fragment/footer :: copy}"` 를 통해, **'템플릿 조각이 존재하는 HTML의 경로'와 '전달받은 템플릿 조각의 이름'을 설정한다.**
    - `th:insert` 와 유사해 보이지만, `th:replace` **는 본인 태그 자체와 템플릿을 교체하는 것이다.**
    - 따라서, 메인HTML의 해당부분에 대한 결과는 다음과 같다.

        ```html
        <footer>
        푸터 자리 입니다.
        </footer>
        ```

<br>

- `<div th:replace="template/fragment/footer :: copy"></div>`
    - `~{...}` 표현식을 사용하지 않고 단순하게 경로를 표현할 수도 있다.
    - 하지만, 복잡한 경로일 때는 `~{...}` 을 사용하는 것이 좋다.

<br>

- `<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터 2')}"></div>`
    - `('데이터1', '데이터 2')` 에 주목해보자. 해당 소괄호 안에는 **템플릿 조각에 전달할 데이터** 를 넣는다.
    - 이 데이터를 전달받은 템플릿 조각의 상태는 아래와 같다.

        ```html
        <!-- 기존 템플릿 -->
        <footer th:fragment="copyParam (param1, param2)">
        	<p>파라미터 자리 입니다.</p>
        	<p th:text="${param1}"></p>
        	<p th:text="${param2}"></p>
        </footer>
        <!-------------------------------------------->

        <!-- 매개변수로 데이터를 전달받은 템플릿 ------->
        <footer th:fragment="copyParam (param1, param2)">
        	<p>파라미터 자리 입니다.</p>
        	<p th:text="'데이터1'"></p>
        	<p th:text="데이터 2"></p>
        </footer>
        <!--------------------------------------------->
        ```

    - `th:replace` 을 통해, 위와 같이 데이터를 넘겨받은 템플릿 조각이 메인HTML의 태그와 교체된다.
    - 따라서, 메인HTML의 해당부분에 대한 결과는 다음과 같다.

        ```html
        <footer>
        	<p>파라미터 자리 입니다.</p>
        	<p>데이터1</p>
        	<p>데이터2</p>
        </footer>
        ```

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*