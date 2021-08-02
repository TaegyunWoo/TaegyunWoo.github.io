---
category: Thymeleaf
tags: [타임리프, JS인라인]
title: "[타임리프] 자바스크립트 인라인"
date:   2021-08-02 15:30:00 
lastmod : 2021-08-02 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 자바스크립트 인라인

## 개요

타임리프는 자바스크립트에서 타임리프를 편리하게 사용할 수 있는 자바스크립트 인라인 기능을 제공한다. `<script th:inline="javascript">` 를 통해 기능을 지원하는데, 바로 예시 코드로 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/javascript")
	public String javascript(Model model) {
		model.addAttribute("user", new User("userA", 10));
		addUsers(model);
		return "basic/javascript";
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

	<!-- 자바스크립트 인라인 사용 전 -->
	<script>
		var username = [[${user.username}]];
		var age = [[${user.age}]];

		//자바스크립트 내추럴 템플릿
		var username2 = /*[[${user.username}]]*/ "test username";

		//객체
		var user = [[${user}]];
	</script>
	
	<!-- 자바스크립트 인라인 사용 후 -->
	<script th:inline="javascript">
		var username = [[${user.username}]];
		var age = [[${user.age}]];

		//자바스크립트 내추럴 템플릿
		var username2 = /*[[${user.username}]]*/ "test username";

		//객체
		var user = [[${user}]];
	</script>

</body>
</html>
```

<br>

### 결과

- **자바스크립트 인라인 (**`th:inline="javascript"`**) 사용 전**

    ```html
    <script>
    	var username = userA; //문법오류
    	var age = 10;

    	//자바스크립트 내추럴 템플릿
    	var username2 = /*userA*/ "test username"; //렌더링 내용이 주석처리됨

    	//객체
    	var user = BasicController.User(username=userA, age=10); //객체의 toString()을 호출함, 문법오류
    </script>
    ```

<br>

- **자바스크립트 인라인(**`th:inline="javascript"`**) 사용 후**

    ```html
    <script>
    	var username = "userA";
    	var age = 10;

    	//자바스크립트 내추럴 템플릿
    	var username2 = "userA";

    	//객체
    	var user = {"username":"userA","age":10};
    </script>
    ```

<br><br>

## 상세 설명

### 텍스트 렌더링

- `var username = [[${user.username}]];`
    - 인라인 사용 전
        - `var username = userA;`
        - (큰 따옴표가 없다.)
    - 인라인 사용 후
        - `var username = "userA";`
        - (큰 따옴표가 존재한다.)
    - **자바스크립트 인라인 (`th:inline="javascript"`) 을 적용하면, 타임리프가 자바스크립트 문법에 맞게 알아서 따옴표를 붙여준다.**

<br>

### 자바스크립트 내추럴 템플릿

타임리프는 HTML 파일을 직접 열어도 동작하는 내추럴 템플릿 기능을 지원한다는 것을 다시 염두하자. 따라서, JS 코드를 작성할 때도 **주석을 통해 내추럴 템플릿 기능을 유지한다.**

- `var username2 = /*[[${user.username}]]*/ "test username";`
    - 인라인 사용 전
        - `var username2 = /*userA*/ "test username";`
        - 그냥 순수하게 해석을 해버렸다.
        - (주석처리가 되어버린다.)
    - 인라인 사용 후
        - `var username2 = "userA";`
        - 주석 부분이 제거되었다.
        - `[[${user.username}]]` 의 값이 적용되었다.
    - **자바스크립트 인라인 (`th:inline="javascript"`) 을 적용하면, 타임리프가 주석부분을 제거하고 `[[...]]` 문법을 적용하여 값을 적용시킨다.**

<br>

### 객체

타임리프의 자바스크립트 인라인 기능을 사용하면 객체를 JSON으로 자동으로 변환해준다.

- `var user = [[${user}]];`
    - 인라인 사용 전
        - `var user = BasicController.User(username=userA, age=10);`
        - 객체의 toString() 메서드가 호출된 값이다.
        - (그저 문자열 값인데, 따옴표도 없으니 문법 오류이다.)
    - 인라인 사용 후
        - `var user = {"username":"userA","age":10};`
        - 객체가 JSON 형태로 변환되어, JS의 객체로 적용된다.
    - **자바스크립트 인라인 (`th:inline="javascript"`) 을 적용하면, 타임리프가 자바객체를 JSON으로 변환하여 적용해준다.**

<br><br><br>

# 자바스크립트 인라인 each

위에서 자바스크립트 인라인의 적용 예시를 살펴보았다. 그렇다면, each 문법은 어떻게 적용해야할까? 지금부터 설명하도록 하겠다.

<br><br>

## 예시 코드

### 컨트롤러

위에서 작성한 코드와 동일하다.

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/javascript")
	public String javascript(Model model) {
		model.addAttribute("user", new User("userA", 10));
		addUsers(model);
		return "basic/javascript";
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

	<!-- 자바스크립트 인라인 each -->
	<script th:inline="javascript">
		[# th:each="user, stat : ${users}"]
			var user[[${stat.count}]] = [[${user}]];
		[/]
	</script>

</body>
</html>
```

<br>

### 결과

```html
<script>
	var user1 = {"username":"userA","age":10};
	var user2 = {"username":"userB","age":20};
	var user3 = {"username":"userC","age":30};
</script>
```

<br><br>

## 상세 설명

자바스크립트 내부에서 타임리프의 each는 다음과 같이 사용한다.

- `[# th:each="user, stat : ${users}"] 반복내용 [/]`
    - [이전 글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_Each)에서 살펴본 문법과 유사하다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*