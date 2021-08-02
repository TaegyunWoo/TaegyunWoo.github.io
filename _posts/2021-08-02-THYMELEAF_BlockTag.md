---
category: Thymeleaf
tags: [타임리프, 블록태그]
title: "[타임리프] 타임리프에서 제공하는 block 태그"
date:   2021-08-02 14:00:00 
lastmod : 2021-08-02 14:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 블록 태그

타임리프는 자체적으로 `<th:block>` 태그를 제공한다. 이 역시, 예시 코드로 간단히 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/block")
	public String block(Model model) {
		addUsers(model);
		return "basic/block";
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

	<th:block th:each="user : ${users}">
		<div>
			사용자 이름1 <span th:text="${user.username}"></span>
			사용자 나이1 <span th:text="${user.age}"></span>
		</div>
		
		<div>
			요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
		</div>
	</th:block>

</body>
</html>
```

<br>

### 결과

![결과](/assets/img/2021-08-02-THYMELEAF_BlockTag/Untitled%2015.png)

<br><br>

## 상세 설명

### 블록 태그

- 타임리프 특성상 HTML 태그안에 속성으로 기능을 정의하여 사용하는데, 위 예시처럼 사용하기 애매한 경우에 사용한다.
- `<th:block th:each="user : ${users}">`
    - `th:each` 를 사용하기 위해, `th:block` 을 활용하였다.
    - `<th:block>` 내부에 작성된 태그들을 반복시키고자 하지만, 내부 태그들을 묶을 적당한 방법을 HTML에서 제공받을 수 없으므로, 자체적인 `<th:block>` 태그를 사용한다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*