---
category: Thymeleaf
tags: [타임리프, 변수표현식, SpringEL]
title: "[타임리프] 기본 표현식: 변수 표현식과 Spring EL"
date:   2021-08-01 19:30:00 
lastmod : 2021-08-01 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 변수 표현식과 Spring EL

## 개요

### 타임리프 변수 표현식

- 타임리프에서 변수를 사용할 때는 아래와 같은 변수 표현식을 사용한다.
    - `${...}`

<br>

### Spring EL

타임리프 변수 표현식 ( `${...}` )에서 스프링이 제공하는 표현식을 사용할 수 있다.

> 아래 예시코드를 통해 바로 설명하겠다.

<br><br>

## 예시 코드

- 컨트롤러

    ```java
    @Controller
    @RequestMapping("/basic")
    public class BasicController {

    	@GetMapping("/variable")
    	public String variable(Model model) {
    		User userA = new User("userA", 10);
    		User userB = new User("userB", 20);

    		List<User> list = new ArrayList<>();
    		list.add(userA);
    		list.add(userB);

    		Map<String, User> map = new HashMap<>();
    		map.put("userA", userA);
    		map.put("userB", userB);

    		model.addAttribute("user", userA);
    		model.addAttribute("users", list);
    		model.addAttribute("userMap", map);
    		return "basic/variable";
    	}

    	@Data
    	static class User {
    		private String username;
    		private int age;
    		
    		public User(String username, int age) {
    			this.username = username;
    			this.age = age;
    		}
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
    	<h1>SpringEL 표현식</h1>
    	<ul>Object
    		<li>${user.username} = <span th:text="${user.username}"></span></li>
    		<li>${user['username']} = <span th:text="${user['username']}"></span></li>
    		<li>${user.getUsername()} = <span th:text="${user.getUsername()}"></span></li>
    	</ul>

    	<ul>List
    		<li>${users[0].username} = <span th:text="${users[0].username}"></span></li>
    		<li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    		<li>${users[0].getUsername()} = <span th:text="$ {users[0].getUsername()}"></span></li>
    	</ul>

    	<ul>Map
    		<li>${userMap['userA'].username} = <span th:text="$ {userMap['userA'].username}"></span></li>
    		<li>${userMap['userA']['username']} = <span th:text="${userMap['userA'] ['username']}"></span></li>
    		<li>${userMap['userA'].getUsername()} = <span th:text="$ {userMap['userA'].getUsername()}"></span></li>
    	</ul>
    </body>
    </html>
    ```

- 결과

    ![결과](/assets/img/2021-08-01-THYMELEAF_VariableAndSpringEL/Untitled%202.png)

<br><br>

## 예시 코드 설명 - SpringEL

### Object

- `user.username`
    - user의 username을 프로퍼티 접근한다.
    - `user.username` == `user.getUsername()`
- `user['username']`
    - 위와 같다.
- `user.getUsername()`
    - user의 `getUsername()` 을 직접 호출한다.

<br>

### List

- `users[0].username`
    - List에서 첫 번째 user를 찾고 username 프로퍼티에 접근한다.
    - `users[0].username` == `users.get(0).getUsername()`
- `users[0]['username']`
    - 위와 같다.
- `users[0].getUsername()`
    - List 에서 첫 번째 회원을 찾고 메서드를 직접 호출한다.

<br>

### Map

- `userMap['userA'].username`
    - Map에서 userA를 찾고, username 프로퍼티에 접근한다.
    - `userMap['userA'].username` == `userMap.get("userA").getUsername()`
- `userMap['userA']['username']`
    - 위와 같다
- `userMap['userA'].getUsername()`
    - Map에서 userA를 찾고 메서드를 직접 호출한다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*