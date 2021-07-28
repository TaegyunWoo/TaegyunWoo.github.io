---
category: Spring
tags: [스프링, MVC, 요청파라미터]
title: "[스프링 - MVC] HTTP 요청 파라미터 조회"
date:   2021-07-28 14:30:00 
lastmod : 2021-07-28 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 개요

## 클라이언트에서 서버로 요청 데이터 전달

### 전달방식

- **GET - 쿼리 파라미터**
    - /url **?myQuery=myValue**
    - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달하는 방식

<br>

- **POST - HTML Form**
    - content-type: application/x-www-form-urlencoded
    - **메시지 바디에 쿼리 파라미터 형식으로 전달** 하는 방식
        - 예) 바디부분: myParam=myValue

<br>

- **HTTP message body에 데이터를 직접 담아서 전달하는 방식**
  

클라이언트가 서버에 데이터를 전달하는 방법은 위 세가지가 전부이다.

<br><br><br>

# 쿼리 파라미터, HTML Form 요청 데이터

## 요청 데이터

### GET, 쿼리 파라미터 요청(전송) 예시

`http://localhost:8080/request-param?username=hello&age=20`

<br>

### POST, HTML Form 요청 메시지 예시

```java
POST /request-param
content-type: application/x-www-form-urlencoded

username=hello&age=20
```

<br>

### 쿼리 파라미터와 HTML Form

- GET 쿼리 파라미터 전송 방식이든, POST HTML Form 전송 방식이든 **둘다 형식이 같으므로 구분없이 조회** 할 수 있다.
    - 즉, 쿼리 파라미터 방식과 Form 방식 모두 조회하는 방법이 같다.
- 단지 "URL에 담느냐", "Form을 통해 바디부분에 담느냐" 의 차이이다.
- 이것을 **요청 파라미터 조회** 라고 한다.

지금부터 예제코드를 통해 하나씩 알아보자.

<br><br>

## 요청 파라미터 기본 조회

### 기초적인 방법으로 요청 파라미터 조회하기

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;

@Slf4j
@Controller
public class RequestParamController {

	/**
	 * 반환 타입이 없으면서 이렇게 응답에 값을 직접 집어넣으면, view 조회X
	 */
	@RequestMapping("/request-param-v1")
	public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
		
		//request.getParameter() 메서드를 통해 파라미터 조회
		String username = request.getParameter("username");
		int age = Integer.parseInt(request.getParameter("age"));
		
		log.info("username={}, age={}", username, age);
		response.getWriter().write("ok");
	}

}
```

- `request.getParameter("파라미터이름")`
    - 아주 단순하게 HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회한다.

<br><br>

## 애너테이션을 활용한 요청 파라미터 조회

### `@RequestParam` 을 사용하여 기본적인 조회하기

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {

	/**
	 * @RequestParam 사용
	 * - 파라미터 이름으로 바인딩
	 * @ResponseBody 추가
	 * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
	 */
	@ResponseBody
	@RequestMapping("/request-param-v2")
	public String requestParamV2( @RequestParam("username") String memberName,
				@RequestParam("age") int memberAge) {

		log.info("username={}, age={}", memberName, memberAge);
		return "ok";
	}

}
```

- `@RequestParam("username") String memberName`
    - 요청 메시지의 파라미터 중, 'username' 이라는 이름의 파라미터의 값을 `memberName`에 전달한다.
    - 파라미터 이름으로 바인딩한다.
- `@ResponseBody`
    - View 조회를 무시하고, HTTP message body에 직접 해당 내용을 입력한다.

위 예시 코드처럼, '요청 메시지의 파라미터 이름' 과 '매개변수의 이름'이 같으면 '요청 메시지의 파라미터 이름'을 생략할 수 있다. 아래 설명을 보자.

<br>

### 파라미터 이름을 생략하여 파라미터 조회하기

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {

	@ResponseBody
	@RequestMapping("/request-param-v2")
	public String requestParamV2( @RequestParam("username") String memberName,
				@RequestParam("age") int memberAge) {

		log.info("username={}, age={}", memberName, memberAge);
		return "ok";
	}

	/**
	 * @RequestParam 사용
	 * HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능 
	 */
	@ResponseBody
	@RequestMapping("/request-param-v3")
	public String requestParamV3(@RequestParam String username, @RequestParam int age) {
		log.info("username={}, age={}", username, age);
		return "ok";
	}

}
```

- `@RequestParam`
    - 매개변수 이름과 요청 파라미터 이름이 같기 때문에, `name 속성` 을 생략할 수 있다.

여기서 `@RequestParam` 애너테이션 자체를 생략할 수도 있다. 만약, 매개변수의 타입이 단순타입(`int` , `String` , `Integer` 등)이라면 애너테이션 생략이 가능하다!

<br>

### `@RequestParam` 을 생략하여 파라미터 조회하기

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {

	@ResponseBody
	@RequestMapping("/request-param-v2")
	public String requestParamV2( @RequestParam("username") String memberName,
				@RequestParam("age") int memberAge) {

		log.info("username={}, age={}", memberName, memberAge);
		return "ok";
	}

	@ResponseBody
	@RequestMapping("/request-param-v3")
	public String requestParamV3(@RequestParam String username, @RequestParam int age) {
		log.info("username={}, age={}", username, age);
		return "ok";
	}

	/**
	 * @RequestParam 자체를 생략했다.
	 */
	@ResponseBody
	@RequestMapping("/request-param-v4")
	public String requestParamV4(String username, int age) {
		log.info("username={}, age={}", username, age);
		return "ok";
	}

}
```

이처럼 생략할 수 있지만, 명시성이 감소하는 문제 때문에 보통 많이 쓰이지는 않는다.

<br><br>

## 파라미터 조회 속성

스프링은 `@RequestParam` 애너테이션에 사용할 수 있는 다양한 속성을 지원한다. 하나씩 알아보자.

<br>

### 파라미터 필수 여부 속성

`required` 속성을 통해 설정한다.

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {
	
	/**
	 * @RequestParam.required
	 * /request-param -> username이 없으므로 예외
	 *
	 * 주의!
	 * /request-param?username= -> 빈문자로 통과
	 *
	 * 주의!
	 * /request-param
	 * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는 defaultValue 사용)
	 */
	@ResponseBody
	@RequestMapping("/request-param-required")
	public String requestParamRequired( @RequestParam(required = true) String username,
				@RequestParam(required = false) Integer age) {

		log.info("username={}, age={}", username, age);
		return "ok";
	}
	
}
```

- `@RequestParam.required`
    - 파라미터 필수 여부
    - **required = true**
        - 요청 파라미터 값이 반드시 존재해야 한다.
        - 기본값이 true이다.
        - 값이 없으면 400 에러가 발생한다.
    - **required = false**
        - 요청 파라미터 값이 없어도 된다.
        - 값이 없으면 null 이 들어가므로, null에 대한 처리가 필요하다.
            - (특히 매개변수가 기본형 데이터타입일 때 주의)
- **파라미터 이름만 전달했을때**
    - `/request-param?username=` 과 같이 이름만 전달했을 때
    - 빈문자로 통과한다.

<br>

### 기본값 적용 속성

`defaultValue` 속성을 통해 설정한다.

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {
	
	/**
	 * @RequestParam
	 * - defaultValue 사용
	 *
	 * 참고: defaultValue는 빈 문자의 경우에도 적용
	 * /request-param?username=
	 */
	@ResponseBody
	@RequestMapping("/request-param-default")
	public String requestParamDefault(
				@RequestParam(required = true, defaultValue = "guest") String username,
				@RequestParam(required = false, defaultValue = "-1") int age) {

		log.info("username={}, age={}", username, age);
		return "ok";
	}
	
}
```

- `@RequestParam.defaultValue`
    - 해당 매개변수의 기본 값을 적용한다.
    - 해당 속성이 사용되면, `required` 속성은 의미가 사라진다.
        - 왜냐하면, `required=true` 이라도 `defaultValue` 가 있으면 매개변수 생략시, 기본값이라도 들어가기 때문이다.

<br><br>

## 전체 파라미터를 Map으로 조회

### Map에 전체 파라미터를 전달받아서 조회하기

```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;
...

@Slf4j
@Controller
public class RequestParamController {
	
	/**
	 * @RequestParam Map, MultiValueMap
	 * Map(key=value)
	 * MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])
	 */
	@ResponseBody
	@RequestMapping("/request-param-map")
	public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
		log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
		return "ok";
	}
	
}
```

- `@RequestParam Map<String, Object> paramMap`
    - 모든 요청 파라미터를 `paramMap` 에 담는다.
    - key 중복(파라미터 이름 중복)이 되지 않는다면, `Map` 을 사용해도 된다.
    - key 중복(파라미터 이름 중복)이 될 수 있다면, `MultiValueMap` 을 사용해야한다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*