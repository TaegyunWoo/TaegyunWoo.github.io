---
category: Spring
tags: [스프링, MVC, 요청헤더]
title: "[스프링 - MVC] HTTP 헤더 정보 조회"
date:   2021-07-28 13:30:00 
lastmod : 2021-07-28 13:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

[이전 글](https://taegyunwoo.github.io/spring/SPRING_MVC_RequestMapping_AnnotationX)에서 요청을 매핑하는 것에 대해 다뤘다. 이번 글에서는 HTTP 헤더 정보를 위주로 간단하게 정리하겠다.

<br><br><br>

# HTTP 헤더 정보 조회

HTTP 헤더 정보를 조회하는 방법을 예시코드와 함께 다루겠다.

<br>

## 예시코드

### 헤더정보 조회하기

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

@Slf4j
@RestController
public class RequestHeaderController {

	@RequestMapping("/headers")
	public String headers(HttpServletRequest request,
				HttpServletResponse response,
				HttpMethod httpMethod,
				Locale locale,
				@RequestHeader MultiValueMap<String, String> headerMap,
				@RequestHeader("host") String host,
				@CookieValue(value = "myCookie", required = false) String cookie) {
		log.info("request={}", request);
		log.info("response={}", response);
		log.info("httpMethod={}", httpMethod);
		log.info("locale={}", locale);
		log.info("headerMap={}", headerMap);
		log.info("header host={}", host);
		log.info("myCookie={}", cookie);
		return "ok";
	}

}
```

- `HttpSerlvetRequest`
    - 요청 메시지에 대한 정보를 담고 있는 객체이다.
- `HttpSerlvetResponse`
    - 응답 메시지에 대한 정보를 담고 있는 객체이다.
- `HttpMethod`
    - HTTP 메서드를 조회한다.
- `Locale`
    - Locale 정보를 조회한다.
- `@RequestHeader MultiValueMap<String, String> headerMap`
    - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.

    > MultiValueMap는 MAP과 유사하지만, 같은 키를 가지는 값을 담을 수 있다.

- `@RequestHeader("host") String host`
    - 특정 HTTP 헤더를 조회한다.
- `@CookieValue(value = "myCookie", required = false) String cookie`
    - 특정 쿠키를 조회한다.
- `@Slf4j`
    - 로그 관련 애너테이션

위에 설명한 모든 매개변수는 스프링이 넘겨준다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*