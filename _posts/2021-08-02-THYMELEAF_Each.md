---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프] 반복문 each"
date:   2021-08-02 12:30:00 
lastmod : 2021-08-02 12:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 반복문

타임리프에서 반복은 `th:each` 를 통해 수행할 수 있다. 또한, 반복에서 사용할 수 있는 여러 상태 값을 지원한다.

바로 예시 코드로 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/each")
	public String each(Model model) {
		addUsers(model);
		return "basic/each";
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

	<h1>기본 테이블</h1>
	<table border="1">
		<tr>
			<th>username</th>
			<th>age</th>
		</tr>
		<tr th:each="user : ${users}">
			<td th:text="${user.username}">username</td> <td th:text="${user.age}">0</td>
		</tr>
	</table>

	<h1>반복 상태 유지</h1>
	<table border="1">
		<tr>
			<th>count</th>
			<th>username</th>
			<th>age</th>
			<th>etc</th>
		</tr>
		<tr th:each="user, userStat : ${users}">
			<td th:text="${userStat.count}">username</td>
			<td th:text="${user.username}">username</td>
			<td th:text="${user.age}">0</td>
			<td>
				index = <span th:text="${userStat.index}"></span>
				count = <span th:text="${userStat.count}"></span>
				size = <span th:text="${userStat.size}"></span>
				even? = <span th:text="${userStat.even}"></span>
				odd? = <span th:text="${userStat.odd}"></span>
				first? = <span th:text="${userStat.first}"></span>
				last? = <span th:text="${userStat.last}"></span>
				current = <span th:text="${userStat.current}"></span>
			</td>
		</tr>
	</table>

</body>
</html>
```

아래에서 자세히 설명하도록 하겠다.

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_Each/Untitled%2012.png)

<br><br>

## 상세 설명

### 기본 반복

- `<tr th:each="user : ${users}">`
    - `th:each="변수 : 반복가능한객체"` 를 통해, 반복문을 사용할 수 있다.
    - `th:each` 가 포함된 태그와 하위 태그를 반복하여 생성한다.
    - 반복 가능한 객체에서 값이나 객체를 하나씩 꺼내 변수에 저장하며 반복한다.
    - 자바의 `for(a : A)` 문법과 유사하다.

<br>

- 반복 가능한 객체
    - 반복 가능한 객체로 아래와 같은 타입이 존재한다.
    - `List`
    - `java.util.Iterable`
    - `java.util.Enumeration`
    - `Map`
        - 이 경우 변수에 담기는 값은 `Map.Entry` 이다.

<br>

### 반복 상태

- `<tr th:each="user, userStat : ${users}">`
    - 반복의 두 번째 파라미터 (`userStat`)을 설정하여 반복의 상태 정보를 얻을 수 있다.
    - 두 번째 파라미터 생략시, `변수명Stat` 로 접근할 수 있다.

<br>

- 반복 상태 유지 기능
    - `index`
        - 0부터 시작하는 값
    - `count`
        - 1부터 시작하는 값
    - `size`
        - 전체 사이즈
    - `even` , `odd`
        - (`count`의) 짝수, 홀수 여부
    - `first`
        - 처음인가?
    - `last`
        - 마지막인가?
    - `current`
        - 현재 객체

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*