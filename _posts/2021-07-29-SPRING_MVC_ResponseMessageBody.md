---
category: Spring-MVC
tags: [스프링, MVC, 메시지바디, 응답]
title: "[스프링 - MVC] 메시지 바디에 직접 입력하여 응답하기"
date:   2021-07-29 15:00:00 
lastmod : 2021-07-29 15:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

[이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ResponseStaticTemplate)에서 정적 리소스와 뷰 템플릿을 통해 응답하는 방법을 살펴보았다. 이번 글에서는 HTTP 메시지 바디에 직접 데이터를 입력하여 응답하는 방법을 알아보도록 하겠다.

<br><br><br>

# 메시지 바디 응답

## 메시지 바디를 이용한 응답

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Controller
public class ResponseBodyController {

	@GetMapping("/response-body-string-v1")
	public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
	}
	
	/**
	 * HttpEntity, ResponseEntity(Http Status 추가)
	 */
	@GetMapping("/response-body-string-v2")
	public ResponseEntity<String> responseBodyV2() {
		return new ResponseEntity<>("ok", HttpStatus.OK);
	}

	@ResponseBody
	@GetMapping("/response-body-string-v3")
	public String responseBodyV3() {
		return "ok";
	}

	@GetMapping("/response-body-json-v1")
	public ResponseEntity<HelloData> responseBodyJsonV1() {
		HelloData helloData = new HelloData();
		helloData.setUsername("userA");
		helloData.setAge(20);
		return new ResponseEntity<>(helloData, HttpStatus.OK);
	}

	@ResponseStatus(HttpStatus.OK)
	@ResponseBody
	@GetMapping("/response-body-json-v2")
	public HelloData responseBodyJsonV2() {
		HelloData helloData = new HelloData();
		helloData.setUsername("userA");
		helloData.setAge(20);
		return helloData;
	}

}
```

메서드 하나하나씩 설명하도록 하겠다.

<br>

### `responseBodyV1` 메서드

- `response.getWriter().write("ok");`
    - 서블릿을 직접 다룰 때처럼, HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 `ok` 응답 메시지를 전달한다.

<br>

### `responseBodyV2` 메서드

- `return new ResponseEntity<>("ok", HttpStatus.OK);`
    - ResponseEntity 객체를 통해 `"ok"` 라는 데이터를 응답 메시지 바디에 입력하고, `OK 상태코드` 를 입력한다.
    - 이렇게 입력된 객체를 생성한 뒤, 반환하여 응답한다.

<br>

### `responseBodyV3` 메서드

- `@ResponseBody`
    - 메서드에서 반환된 문자열이 응답 메시지 바디에 입력된다.

<br>

### `responseBodyJsonV1` 메서드

- `return new ResponseEntity<>(helloData, HttpStatus.OK);`
    - `ResponseEntity` 객체를 통해, 새로 생성한 객체 `helloData` 를 입력한다. (이렇게 자바 객체를 입력하면 JSON 형식으로 입력된다.)
    - `helloData` 가 JSON 형식으로 담긴 `ResponseEntity` 객체를 응답하는데 사용한다.

<br>

### `responseBodyJsonV2()` 메서드

- `@ResponseBody`
    - 메서드에서 반환된 자바 객체가 응답 메시지 바디에 JSON 형식으로 입력된다.

<br><br>

## `@ResponseBody`

`@ResponseBody` 애너테이션은 메서드의 반환값을 직접 응답 메시지 바디에 입력하는 기능을 지원한다. 하지만, 각 메서드마다 사용하기가 번거롭다면 `@RestController` 를 사용하면 된다.

<br>

### `@RestController`

- 클래스 레벨에 적용하는 애너테이션이다.
- `@Controller` 와 `@ResponseBody` 를 포함하는 애너테이션이다.
- 즉, 해당 클래스에 `@Controller` 를 적용하고, 모든 메서드에 `@ResponseBody` 를 적용시킨다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*