---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 조건부 평가"
date:   2021-08-02 13:00:00 
lastmod : 2021-08-02 13:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 조건부 평가

타임리프의 조건식은 `if` 와 `unless` 그리고 `switch` 를 사용한다.

예시 코드를 통해 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/condition")
	public String condition(Model model) {
		addUsers(model);
		return "basic/condition";
	}

	private void addUsers(Model model) {
		List<User> list = new ArrayList<>();
		list.add(new User("userA", 10));
		list.add(new User("userB", 20));
		list.add(new User("userC", 30));
		model.addAttribute("users", list);
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

	<h1>if, unless</h1>
	<table border="1">
		<tr>
			<th>count</th>
			<th>username</th>
			<th>age</th>
		</tr>
		<tr th:each="user, userStat : ${users}">
			<td th:text="${userStat.count}">1</td>
			<td th:text="${user.username}">username</td>
			<td>
				<span th:text="${user.age}">0</span>
				<span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
				<span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>
			</td>
		</tr>
	</table>

	<h1>switch</h1>
	<table border="1">
		<tr>
			<th>count</th>
			<th>username</th>
			<th>age</th>
		</tr>
		<tr th:each="user, userStat : ${users}">
			<td th:text="${userStat.count}">1</td>
			<td th:text="${user.username}">username</td>
			<td th:switch="${user.age}">
				<span th:case="10">10살</span>
				<span th:case="20">20살</span>
				<span th:case="*">기타</span>
			</td>
		</tr>
	</table>

</body>
</html>
```

아래에서 자세히 설명하도록 하겠다.

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_IfUnlessSwitch/Untitled%2013.png)

<br><br>

## 상세 설명

### if

- `th:if="${조건식}"`
	- 조건이 맞지 않으면, 해당 태그 자체를 렌더링하지 않는다.
	- 즉, 조건식이 false라면, 해당 태그가 제거된다.

<br>

- `<span th:text="'미성년자'" th:if="${user.age lt 20}"></span>`
	- `th:if="${user.age lt 20}"` 에서 조건식에 해당되는 `${user.age lt 20}` 이 false 라면 `<span>` 태그 자체가 제거된다.

<br>

### unless

- `th:unless="${조건식}"`
	- if와 마찬가지로, 조건이 맞지 않으면 해당 태그 자체를 렌더링하지 않는다.
	- 이때, `unless` 는 if의 반대이다. 즉, 조건식에 not이 붙는다.

<br>

- `<span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>`
	- if와 유사하지만, `${user.age ge 20}` 이 false일때 `<span>` 태그가 렌더링된다.

<br>

### switch

- `<td th:switch="${user.age}">`
    - 해당 태그의 하위 태그들이 case 들이다.
    - `th:switch="${user.age}"` 에서 조건식인 `${user.age}` 변수의 값에 따라, case가 적용된다.

<br>

- `th:case="*"`
    - `*` 은 디폴트값으로 해당되는 case들이 없을 때 적용된다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*