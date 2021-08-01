---
category: Thymeleaf
tags: [타임리프, 링크표현식]
title: "[타임리프] 기본 표현식: URL 링크"
date:   2021-08-01 20:30:00 
lastmod : 2021-08-01 20:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# URL 링크

타임리프에서 URL을 생성할 때는 `@{...}` 을 사용하면 된다. 바로 예시 코드로 알아보자.

<br><br>

## 예시 코드

### 컨트롤러

```java
@Controller
@RequestMapping("/basic")
public class BasicController {

	@GetMapping("/link")
	public String link(Model model) {
		model.addAttribute("param1", "data1");
		model.addAttribute("param2", "data2");
		return "basic/link";
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
	<h1>URL 링크</h1>
	<ul>
		<li><a th:href="@{/hello}">basic url</a></li>
		<li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
		<li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
		<li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
	</ul>
</body>
</html>
```

<br>

### 결과

![결과](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%204.png)

- **basic url**

    ![basic url](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%205.png)

- **hello query param**

    ![hello query param](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%206.png)

- **path variable**

    ![path variable](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%207.png)

- **path variable + query param**

    ![path variable + query param](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%208.png)

<br><br>

## 예시 코드 설명

### basic url

- `th:href="@{/hello}"`
- 아주 기초적인 형태이다.
- 변환 결과

  ![basic url](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%205.png)

<br>

### **hello query param**

- `th:href="@{/hello(param1=${param1}, param2=${param2})}"`
- 소괄호 안에 작성된 `param1=${param1}` 과 `param2=${param2}` 는 `/hello` url 뒤에 쿼리스트링으로 변환된다.
- 변환 결과

    ![hello query param](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%206.png)

<br>

### **path variable**

- `th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}"`
- 스프링에서의 `@PathVariable` 애너테이션과 같은 기능을 지원한다.
- `{param1}` , `{param2}` 자리에 소괄호 안에 작성된 `param1=${param1}` 과 `param2=${param2}` 의 값이 들어간다.
- 이때, `param1=${param1}` 과 `param2=${param2}` 의 이름 부분 (`param1` , `param2`) 과 경로변수의 이름 ( `{param1}` , `{param2}` )이 매핑되어 들어간다.
- 변환 결과

    ![path variable](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%207.png)

<br>

### **path variable + query param**

- `th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}"`
- 경로변수와 쿼리스트링을 동시 사용하는 방법이다.
- 경로변수( `{param1}` )와 소괄호 안에 작성된 것들 중 `param2=${param2}` 가 매핑되어 경로가 완성된다.
- 나머지 `param2=${param2}` 는 완성된 URL 뒤에 붙어, 쿼리스트링이 된다.
- 변환 결과

    ![path variable + query param](/assets/img/2021-08-01-THYMELEAF_Link/Untitled%208.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*