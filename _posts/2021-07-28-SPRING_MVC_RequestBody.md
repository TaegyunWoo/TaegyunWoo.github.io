---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] HTTP 요청 바디 데이터 조회"
date:   2021-07-28 17:30:00 
lastmod : 2021-07-28 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

이전 게시글들 ([요청파라미터](https://taegyunwoo.github.io/spring/SPRING_MVC_HTTP_RequestParameter) , [@ModelAttribute](https://taegyunwoo.github.io/spring/SPRING_MVC_ModelAttribute)) 에서 "GET, 쿼리 파라미터"와 "POST, HTML Form" 으로 전달된 파라미터를 처리하는 방법에 대해 설명하였다.

요청 메시지가 어떻게 데이터를 전달하는지 다시 확인해보자면 아래와 같다.

- **GET - 쿼리 파라미터**
- **POST - HTML Form**
- **HTTP message body에 데이터를 직접 담아서 전달하는 방식**

이 중 첫 번째와 두 번째 방식을 처리하는 방법에 대해서 이전 글에서 다뤘으니, 이제 세 번째 방식 '요청 메시지의 바디부에 데이터가 담긴 방식' 을 어떻게 처리할 것인지 알아보자.

<br><br><br>

# 개요

## HTTP message body에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용한다. (JSON, XML, TEXT 등)
- **데이터 형식은 주로 JSON을 사용한다.**

이 방식의 경우, 요청 파라미터와는 다르게 `@RequestParam` , `@ModelAttribute` 를 사용할 수 없다.

> 물론, HTML Form 형식을 통해 바디부에 입력되는 경우 제외

<br><br><br>

# 요청 메시지 바디 데이터 처리

## 기초적인 방식

### 직접 InputStream을 가져와서 바디 데이터 조회하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {

	@PostMapping("/request-body-string-v1")
	public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
		
		//InputStream을 통해 요청msg의 바디부 데이터를 받는다. (바이트코드로 받음)
		ServletInputStream inputStream = request.getInputStream();

		//바이트코드로 구성된 바디부 데이터를 UTF-8 형식으로 인코딩하여 문자열로 변환한다.
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

		log.info("messageBody={}", messageBody);
		response.getWriter().write("ok");
	}

}
```

- `request.getInputStream()`
    - 요청 메시지의 바디부분에 저장되어있는 데이터를 바이트코드로 받는다.
- `StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8)`
    - InputStream으로 받은 데이터를 인코딩하여 문자열로 변환한다.

<br>

### 스프링에게 `InputStream` , `Writer` 객체를 전달받아 조회하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {

	/**
	 * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
	 * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
	 */
	@PostMapping("/request-body-string-v2")
	public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
		log.info("messageBody={}", messageBody);
		responseWriter.write("ok");
	}

}
```

- **매개변수** `InputStream inputStream`
    - 스프링으로부터 '요청 메시지의 바디 데이터가 들어있는 `InputStream` 객체' 를 전달받는다.
- **매개변수** `Writer responseWriter`
    - 스프링으로부터 '응답 메시지의 바디에 데이터를 작성할 수 있는 `Writer` 객체' 를 전달받는다.
- **InputStream** (Reader)
    - HTTP 요청 메시지 바디의 내용을 직접 조회하는 기능을 지원한다.
- **OutputStream** (Writer)
    - HTTP 응답 메시지의 바디에 직접 결과 출력하는 기능을 지원한다.

<br>

### 스프링에게 `HttpEntity<>` 객체를 전달받아 조회하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {

	/**
	 * HttpEntity: HTTP header, body 정보를 편라하게 조회
	 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
	 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 *
	 * 응답에서도 HttpEntity 사용 가능
	 * - 메시지 바디 정보 직접 반환(view 조회X)
	 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 */
	@PostMapping("/request-body-string-v3")
	public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
		
		//요청 메시지의 바디부분 데이터 받기
		String messageBody = httpEntity.getBody();

		log.info("messageBody={}", messageBody);

		//HttpEntity<> 객체로 응답도 할 수 있다.
		return new HttpEntity<>("ok");
	}

}
```

- `HttpEntity`
    - HTTP header, body 정보를 편리하게 조회할 수 있도록 지원하는 객체이다.
    - 매개변수 `HttpEntity<String> httpEntity` 는 스프링으로부터 받는다.
    - 요청 파라미터를 조회하는 기능과 관계는 없다.
        - `@RequestParam` , `@ModelAttribute` 와 관계 없음.
    - 응답에도 사용할 수 있다.
        - 응답으로 사용시, 메시지 바디 정보를 직접 반환하게 된다.
        - 헤더 정보 포함 가능
        - view 조회는 안된다.

- `HttpEntity` 를 상속받은 객체들
    - `RequestEntity`
        - HttpMethod, url 정보가 추가됐다.
        - 요청에서 사용한다.
    - `ResponseEntity`
        - HTTP 상태 코드를 설정할 수 있다.

> 참고로, 스프링 MVC 내부에서 'HTTP 메시지 바디'를 읽어서 문자나 객체로 변환하여 전달해준다. 이때 사용되는 기능이 바로 **"HTTP 메시지 컨버터"** 이다. HTTP 메시지 컨버터는 나중에 자세히 다룬다.

<br><br>

## 애너테이션 방식

### `@RequestBody` 를 통해 요청 메시지의 바디 데이터 조회하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {

	/**
	 * @RequestBody
	 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
	 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 *
	 * @ResponseBody
	 * - 메시지 바디 정보 직접 반환(view 조회X)
	 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 */
	@ResponseBody
	@PostMapping("/request-body-string-v4")
	public String requestBodyStringV4(@RequestBody String messageBody) {
		log.info("messageBody={}", messageBody);
		return "ok";
	}

}
```

- `@RequestBody`
    - HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.
    - 헤더 정보가 필요하면 위에서 설명한 `HttpEntity` 를 사용하거나, `@RequestHeader` 를 사용하면 된다.
    - 요청 파라미터 조회 기능인 `@RequestParam` 과 `@ModelAttribute` 와 관계가 없다.
- `@ResponseBody` 역시 존재한다.
    - HTTP 응답 메시지에 직접 정보를 담아서 전달할 수 있다.

<br><br>

## 요청 파라미터 vs HTTP 메시지 바디

### 요청 파라미터 조회 기능

- `@RequestParam`
- `@ModelAttribute`

<br>

### HTTP 메시지 바디 정보 조회 기능

- `@RequestBody`

<br><br><br>

# 요청 메시지 바디의 JSON 데이터 처리

이번에는 요청 메시지의 바디가 JSON 형식일 경우에 대해 알아보자.

<br><br>

## 기초적인 방식

### 직접 InputStream을 가져와서 바디 데이터를 자바 객체로 전환하기

```java
/**
 * 요청메시지의 정보
 * 바디: {"username":"hello", "age":20} 
 * 헤더: content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {

	//JSON을 처리하기 위한 객체
	private ObjectMapper objectMapper = new ObjectMapper();

	@PostMapping("/request-body-json-v1")
	public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
		
		//InputStream을 통해 요청 메시지 바디 정보를 가져옴.
		ServletInputStream inputStream = request.getInputStream();

		//인코딩하여 문자열로 변환
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

		log.info("messageBody={}", messageBody);

		//ObjectMapper 객체를 통해 JSON 데이터를 HelloData 객체에 바인딩
		HelloData data = objectMapper.readValue(messageBody, HelloData.class);

		log.info("username={}, age={}", data.getUsername(), data.getAge());
		response.getWriter().write("ok");
	}

}
```

- `request.getInputStream()`
    - 위에서 설명했던 것과 동일하다.
- `ObjectMapper` **객체**
    - 문자로 된 JSON 데이터를 Jackson 라이브러리인 `ObjectMapping` 를 사용해서 자바 객체로 변환한다.

<br><br>

## 애너테이션 방식

### `@RequestBody` 를 통해 JSON 데이터를 문자열로 받아 객체로 전환하기

```java
/**
 * 요청메시지의 정보
 * 바디: {"username":"hello", "age":20} 
 * 헤더: content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {

	private ObjectMapper objectMapper = new ObjectMapper();

	/**
	 * @RequestBody
	 * HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 *
	 * @ResponseBody
	 * - 모든 메서드에 @ResponseBody 적용
	 * - 메시지 바디 정보 직접 반환(view 조회X)
	 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
	 */
	@ResponseBody
	@PostMapping("/request-body-json-v2")
	public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

		//문자열로 받은 JSON 데이터를 HelloData 객체로 변환한다.
		HelloData data = objectMapper.readValue(messageBody, HelloData.class);

		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
	}

}
```

- 위에서 설명한 `@RequestBody` 를 적용하여 문자열을 자바 객체로 변환했다.
- 하지만, 여전히 불편하다...
- 아래에 설명을 계속하겠다.

<br>

### `@RequestBody` 를 통해 JSON 데이터를 자바 객체로 전환하기

```java
/**
 * 요청메시지의 정보
 * 바디: {"username":"hello", "age":20} 
 * 헤더: content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {

	/**
	 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
	 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content- type: application/json)
	 *  */
	@ResponseBody
	@PostMapping("/request-body-json-v3")
	public String requestBodyJsonV3(@RequestBody HelloData data) {
		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
	}

}
```

- **매개변수** `@RequestBody HelloData data`
    - HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
    - 위 코드 (`requestBodyJsonV3` 메서드) 의 경우, JSON 데이터를 자바 객체로 변환하여 매개변수로 전달해준 것이다.

    > 다시 이야기하지만, 이 마법같은 HTTP 메시지 컨버터는 나중에 다루겠다.

- **주의사항**
    - HTTP 요청 메시지의 헤더가 `content-type: application/json` 를 포함하는지 확인해야한다.
    - 그래야만, HTTP 메시지 컨버터가 정상적으로 JSON 데이터를 자바 객체로 변환해준다.

위 코드는 아래 코드와 동일하다.

<br>

### `HttpEntity` 객체를 통해 JSON 데이터를 자바 객체로 전환하기

```java
/**
 * 요청메시지의 정보
 * 바디: {"username":"hello", "age":20} 
 * 헤더: content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {

	@ResponseBody
	@PostMapping("/request-body-json-v3")
	public String requestBodyJsonV3(@RequestBody HelloData data) {
		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
	}

	//위 코드 (requestBodyJsonV3 메서드)와 동일하게 동작한다.
	@ResponseBody
	@PostMapping("/request-body-json-v4")
	public String requestBodyJsonV4(HttpEntity<HelloData> data) {
		log.info("username={}, age={}", data.getUsername(), data.getAge());
		return "ok";
	}

}
```

<br>

### JSON 형식으로 응답하기

```java
/**
 * 요청메시지의 정보
 * 바디: {"username":"hello", "age":20} 
 * 헤더: content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {

	/**
	 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
	 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content- type: application/json)
	 *
	 * @ResponseBody 적용
	 * - 메시지 바디 정보 직접 반환(view 조회X)
	 * - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용 (Accept: application/json)
	 */
	@ResponseBody
	@PostMapping("/request-body-json-v5")
	public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
		log.info("username={}, age={}", data.getUsername(), data.getAge());

		//자바 객체를 그대로 반환하여, JSON 형식으로 응답한다.
		//이러한 동작은 @ResponseBody를 통해 가능하다.
		return data;
	}

}
```

- `@ResponseBody`
    - 응답 메시지 바디에 직접 데이터를 입력한다. (반환값을 응답 데이터로)
    - 자바 객체를 반환할 땐, JSON 형식으로 응답하게 된다.

<br><br>

## 정리

### `@RequestBody`

- JSON 요청 → HTTP 메시지 컨버터 → 객체
- 매개변수의 데이터타입에 따라, 동작이 달라진다.
    - String 타입의 매개변수에 적용시
        - 요청 메시지의 바디 정보를 문자열로 넘겨준다.
    - 단순 타입 이외의 매개변수에 적용시
        - 요청 메시지의 바디 정보를 해당 객체로 변환하여 넘겨준다.
        - (당연히 JSON 타입으로 요청시, `content-type: application/json` 이어야 한다.)

### `@ResponseBody`

- 객체 → HTTP 메시지 컨버터 → JSON 응답
- 반환값의 데이터타입에 따라, 동작이 달라진다.
    - 반환값이 String 타입일 때
        - 응답 메시지 바디에 문자열(text)로 담아 응답한다.
    - 반환값이 단순 타입 이외의 매개변수일 때
        - 응답 메시지 바디에 JSON 형식으로 담아 응답한다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*