---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 타임리프에서의 연산 방법들"
date:   2021-08-02 11:30:00 
lastmod : 2021-08-02 11:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 연산

## 개요

타임리프에서의 연산은 자바와 크게 다르지 않지만, HTML 안에서 사용하기 때문에 HTML 엔티티를 사용하는 부분에 유의하자.

바로 예시 코드를 통해 설명하도록 하겠다.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/operation")
	public String operation(Model model) {
		model.addAttribute("nullData", null);
		model.addAttribute("data", "Spring!");
		return "basic/operation";
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

	<ul>

		<li>산술 연산
			<ul>
				<li>10 + 2 = <span th:text="10 + 2"></span></li>
				<li>10 % 2 == 0 = <span th:text="10 % 2 == 0"></span></li>
			</ul>
		</li>

		<li>비교 연산
			<ul>
				<li>1 > 10 = <span th:text="1 &gt; 10"></span></li>
				<li>1 gt 10 = <span th:text="1 gt 10"></span></li>
				<li>1 >= 10 = <span th:text="1 >= 10"></span></li>
				<li>1 ge 10 = <span th:text="1 ge 10"></span></li>
				<li>1 == 10 = <span th:text="1 == 10"></span></li>
				<li>1 != 10 = <span th:text="1 != 10"></span></li>
			</ul>
		</li>

		<li>조건식
			<ul>
				<li>(10 % 2 == 0)? '짝수':'홀수' = <span th:text="(10 % 2 == 0)? '짝수':'홀수'"></span></li>
			</ul>
		</li>

		<li>Elvis 연산자
			<ul>
				<li>${data}?: '데이터가 없습니다.' = <span th:text="${data}?: '데이터가없습니다.'"></span></li>
				<li>${nullData}?: '데이터가 없습니다.' = <span th:text="${nullData}?: '데이터가 없습니다.'"></span></li>
			</ul>
		</li>

		<li>No-Operation
			<ul>
				<li>${data}?: _ = <span th:text="${data}?: _">데이터가 없습니다.</span></li>
				<li>${nullData}?: _ = <span th:text="${nullData}?: _">데이터가없습니다.</span></li>
			</ul>
		</li>

	</ul>
</body>
</html>
```

아래에서 자세히 설명하도록 하겠다.

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_Operations/Untitled%2010.png)

<br><br>

## 상세 설명

### 산술 연산

- `th:text="10 + 2"`
    - 문자 그대로 `10+2` 를 계산할 결과를 텍스트로 출력한다.
- `th:text="10 % 2 == 0"`
    - `10%2` 의 결과가 `0` 인지, 불린형으로 출력한다.

<br>

### 비교 연산

- `th:text="1 &gt; 10"`
    - `1 > 10` 과 같은 의미이다.
    - HTML의 엔티티인 `&gt;` 를 사용한다.
    - 연산 결과인 불린형 데이터를 출력한다.
- `th:text="1 gt 10"`
    - 위와 동일하다.
- `th:text="1 >= 10"`
    - 문자 그대로 `1 >= 10` 의 결과를 출력한다.
- `th:text="1 == 10"`
    - 문자 그대로 `1 == 10` 의 결과를 출력한다.

<br>

### 조건식

- `th:text="(10 % 2 == 0)? '짝수':'홀수'"`
    - 소괄호 안에 작성된 식은 **조건식** 이다.
    - `:` 으로 구분된 값은 조건식 결과에 따라 선택될 값들이다.
    - 해당 조건식의 결과값이 true면, 첫 번째 값(`'짝수'`)가 선택되어 출력된다.
    - 해당 조건식의 결과값이 false면, 두 번째 값(`'홀수'`)가 선택되어 출력된다.

<br>

### Elvis 연산자

- `th:text="${data}?: '데이터가없습니다.'"`
    - 조건식 연산자와 유사하지만 차이점이 있다.
    - 조건식이 들어가야 하는 부분에 **변수가 들어간다.**
    - 변수에 값이 존재하면, 해당 값을 출력한다.
    - 변수에 값이 존재하지 않으면(`null`이면), 뒤에 작성된 `'데이터가없습니다.'` 가 출력된다.

<br>

### No-Operation

- `th:text="${data}?: _"`
    - Elvis 연산자와 유사하지만 차이점이 있다.
    - 변수에 값이 존재하면, 해당 값을 출력한다.
    - 변수에 값이 존재하지 않으면(`null`이면), **해당 태그에 Thymeleaf를 적용하지 않는다.**

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*