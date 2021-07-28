---
category: Spring-MVC
tags: [스프링, MVC, 요청매핑]
title: "[스프링 - MVC] 요청매핑을 하는 다양한 방법"
date:   2021-07-28 11:30:00 
lastmod : 2021-07-28 11:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> 본 게시글은 로그 라이브러리로 `SLF4J` 와 `LogBack` 을 사용한다. 관련 자료는 따로 검색해보길 바란다.  
SLF4J - [http://www.slf4j.org/](http://www.slf4j.org/)  
LogBack - [http://logback.qos.ch/](http://logback.qos.ch/)

<br><br><br>

# 요청 매핑

이번 글에서는 클라이언트(고객)으로부터 들어온 요청을 어떻게 처리하는지 자세히 다루도록 하겠다.

<br><br>

## 기본 매핑

기본적인 매핑 방식을 예시 코드를 통해 알아보자.

<br>

### 기초적인 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	//로깅을 위한 변수
	// @Slf4j 로 대체 가능
	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * 기본 요청
	 * /hello-basic, /hello-basic/ 모두 매핑됨.
	 * HTTP 메서드 GET, HEAD, POST, PUT, PATCH, DELETE 모두 허용함.
	 */
	@RequestMapping("/hello-basic")
	public String helloBasic() {
		log.info("helloBasic"); //로깅
		return "ok";
	}

}
```

- `@RestController`
    - `@Controller`
        - 메서드의 반환 값이 `String` 이면 뷰 이름으로 인식된다. 그래서 **뷰를 찾고 뷰가 랜더링** 된다.
    - `@RestController`
        - 메서드의 반환 값이 **HTTP 메시지의 바디부분에 바로 입력** 된다.
        - **즉, 뷰를 찾고 랜더링하는 행위를 하지 않는다.**

        > `@ResponseBody` 애너테이션을 다룰때 자세히 설명하겠다.

- `@RequestMapping("/hello-basic")`
    - `/hello-basic` 이나 `/hello-basic/` URL로 요청이 오면, 해당 메서드가 호출된다.
    - 여러 URL을 매핑하도록 설정할 수 있다.
        - 예) `@RequestMapping( {"/hello1" , "/hello2"} )`
    - method 속성을 사용하여 HTTP 메서드를 지정하지 않으면, 모든 HTTP 메서드를 허용한다.

<br><br>

## HTTP 메서드 매핑

### HTTP 메서드를 지정하여 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * method 특정 HTTP 메서드(GET) 요청만 허용
	 */
	@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
	public String mappingGetV1() {
		log.info("mappingGetV1");
			return "ok";
	}
}
```

- `@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)`
    - `@RequestMapping` 의 method 속성을 통해 특정 HTTP 메서드를 지정하여 매핑할 수 있다.
    - **하지만, 직관성이 떨어진다..**

<br>

### HTTP 메서드 설정을 축약하여 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
	public String mappingGetV1() {
		log.info("mappingGetV1");
			return "ok";
	}

	/**
	 * 편리한 축약 애노테이션
	 * @GetMapping
	 * @PostMapping
	 * @PutMapping
	 * @DeleteMapping
	 * @PatchMapping
	*/
	@GetMapping(value = "/mapping-get-v2")
	public String mappingGetV2() {
		log.info("mapping-get-v2");
		return "ok";
	}

}
```

- `@GetMapping(value = "/mapping-get-v2")`
    - `@GetMapping` 은 `@RequestMapping(method = "RequestMethod.GET")` 과 동일하다
    - 즉, `@GetMapping` 애너테이션을 통해 직관적으로 HTTP 메서드를 설정하여 매핑할 수 있다.
    - value 속성은 매핑할 URL을 의미한다.

<br><br>

## PathVariable(경로 변수) 매핑

### PathVariable을 활용하여 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * PathVariable 사용  (변수명이 같으면 생략 가능)
	 * 생략예시) "@PathVariable("userId") String userId" -> "@PathVariable userId"
	*/
	@GetMapping("/mapping/{userId}")
	public String mappingPath(@PathVariable("userId") String data) {
		log.info("mappingPath userId={}", data);
		return "ok";
	}

}
```

- `@GetMapping("/mapping/{userId}")`

    - 사용자가 요청한 URL에서 `{userId}` 에 해당되는 부분을 추출한다.
    - `@PathVariable("userId")` 애너테이션을 통해 추출된 값을 `String data` 에 전달한다.
- `@PathVariable("userId") String data`
    - 경로 변수 중 `{userId}` 에 해당되는 값을 매개변수 `data` 에 전달한다.
    - '매개변수의 이름' 과 '경로 변수의 이름' 이 같다면 `@PathVariable("userId")` 에서 `"userId"` 를 생략할 수 있다.
        - 예) `@PathVariable("userId")` ⇒ `@PathVariable`

참고로, 최근 HTTP API는 아래와 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.

- /mapping/userA
- /users/1

<br>

### PathVariable을 여러 개 사용하기, 경로변수명 생략하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	@GetMapping("/mapping/{userId}")
	public String mappingPath(@PathVariable("userId") String data) {
		log.info("mappingPath userId={}", data);
		return "ok";
	}

	/**
	 * PathVariable 사용 다중
	 */
	@GetMapping("/mapping/users/{userId}/orders/{orderId}")
	public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
		log.info("mappingPath userId={}, orderId={}", userId, orderId);
		return "ok";
	}

}
```

- `@GetMapping("/mapping/users/{userId}/orders/{orderId}")`
    - 이와 같이, 여러 개의 경로변수를 사용할 수 있다.
- `@PathVariable String userId, @PathVariable Long orderId`
    - 경로 변수와 매개변수의 이름이 일치하여 `@PathVariable` 의 속성을 생략할 수 있다.

<br><br>

## 특정 파라미터 조건 매핑

### 특정 파라미터와 값으로 매핑하기

- 클라이언트가 보낸 요청 URL의 **특정한 파라미터의 존재여부와 값을 활용하여 매핑** 한다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * 파라미터로 추가 매핑
	 * params="mode",
	 * params="!mode"
	 * params="mode=debug"
	 * params="mode!=debug" (! = )
	 * params = {"mode=debug","data=good"}
	 */
	@GetMapping(value = "/mapping-param", params = "mode=debug")
	public String mappingParam() {
		log.info("mappingParam");
		return "ok";
	}

}
```

- `@GetMapping(value = "/mapping-param", params = "mode=debug")`
    - `"/mapping-param"` URL로 들어온 GET방식의 요청 중, 요청메시지의 파라미터에서 `"mode=debug"` 를 갖는 요청과 매핑된다.
    - 즉, 이와 같은 URL와 매핑된다.
        - `"/mapping-param?mode=debug"`
- 하지만, 잘 사용하지는 않는다.

<br><br>

## 특정 헤더 조건 매핑

### 특정 헤더와 값으로 매핑하기

- 클라이언트가 보낸 요청 메시지의 **특정한 헤더의 존재여부와 값을 활용하여 매핑** 한다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * 특정 헤더로 추가 매핑
	 * headers="mode",
	 * headers="!mode"
	 * headers="mode=debug"
	 * headers="mode!=debug" (! = )
	 */
	@GetMapping(value = "/mapping-header", headers = "mode=debug")
	public String mappingHeader() {
		log.info("mappingHeader");
		return "ok";
	}

}
```

- `@GetMapping(value = "/mapping-header", headers = "mode=debug")`
    - HTTP 요청 메시지의 헤더에서 'mode'라는 헤더의 값이 'debug'인 것만 매핑한다.

> Postman을 활용하여 테스트해보길 권장한다!

<br><br>

## 미디어 타입 조건 매핑

- 요청 메시지의 **헤더** `content-type` **의 값을 활용하여 매핑** 한다. (consumes)
- 요청 메시지의 **헤더** `Accept` **의 값을 활용하여 매핑** 한다. (produces)

<br>

### `content-type` 헤더 값으로 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	/**
	 * Content-Type 헤더 기반 추가 매핑 Media Type
	 * consumes="application/json"
	 * consumes="!application/json"
	 * consumes="application/*"
	 * consumes="*\/*"
	 * MediaType.APPLICATION_JSON_VALUE
	 */
	@PostMapping(value = "/mapping-consume", consumes = "application/json")
	public String mappingConsumes() {
		log.info("mappingConsumes");
		return "ok";
	}

}
```

- `@PostMapping(value = "/mapping-consume", consumes = "application/json")`
    - 요청 헤더 `content-type` 의 값이 `application/json` 인 요청과 매핑된다.
- `content-type`
    - 요청 메시지의 공식 헤더이다.
    - **요청 메시지의 내용(바디) 형식** 을 알려주는 헤더이다.
    - 서버가 요청 메시지의 내용(바디)을 **사용** 한다. ⇒ **consumes**

<br>

### `Accept` 헤더 값으로 매핑하기

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;
...

@RestController
public class MappingController {

	private Logger log = LoggerFactory.getLogger(getClass());

	@PostMapping(value = "/mapping-consume", consumes = "application/json")
	public String mappingConsumes() {
		log.info("mappingConsumes");
		return "ok";
	}

	/**
	 * Accept 헤더 기반 Media Type
	 * produces = "text/html"
	 * produces = "!text/html"
	 * produces = "text/*"
	 * produces = "*\/*"
	 */
	@PostMapping(value = "/mapping-produce", produces = "text/html")
	public String mappingProduces() {
		log.info("mappingProduces");
		return "ok";
	}

}
```

- `@PostMapping(value = "/mapping-consume", produces = "text/html")`
    - 요청 헤더 `Accept` 의 값이 `/text/html` 인 요청과 매핑된다.
- `Accept`
    - 요청 메시지의 공식 헤더이다.
    - 요청자가 받을 수 있는 **응답 메시지의 내용(바디) 형식** 을 나타낸다.
    - 서버가 응답 메시지의 내용(바디)을 클라이언트에게 **제공** 한다. ⇒ **produces**

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*