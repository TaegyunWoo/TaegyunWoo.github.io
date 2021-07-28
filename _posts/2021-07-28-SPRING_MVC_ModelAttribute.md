---
category: Spring
tags: [스프링, MVC, ModelAttribute, 객체바인딩]
title: "[스프링 - MVC] @ModelAttribute"
date:   2021-07-28 15:30:00 
lastmod : 2021-07-28 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 개요

## `@ModelAttribute` 란?

- 보통 요청 파라미터를 받아서, 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다.
- 이러한 과정을 쉽게 할 수 있도록 도와주는 기능이 있는데, 이것이 바로 `@ModelAttribute` 이다.

<br><br>

## 파라미터를 바인딩할 객체 클래스

`@ModelAttribute` 를 설명하기 전에, 먼저 파라미터를 담을(바인딩) 객체의 클래스를 작성하자.

```java
import lombok.Data;

@Data
public class HelloData {
	private String username;
	private int age;
}
```

- `@Data`
    - 롬복 라이브러리가 제공하는 애너테이션이다.
    - `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를 자동으로 적용해준다.
    - 즉, 해당 클래스를 VO객체로 만들어주는 애너테이션이다. (관련 메서드를 자동으로 생성해줌.)

이제 본격적으로 `@ModelAttribute` 애너테이션을 설명하도록 하겠다.

<br><br><br>

# `@ModelAttribute`

## 요청파라미터를 객체에 바인딩

### 기본적인 방식으로 객체에 바인딩하기

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
	 * @ModelAttribute 사용
	 * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때 자세히 설명
	 */
	@ResponseBody
	@RequestMapping("/model-attribute-v1")
	public String modelAttributeV1(@ModelAttribute HelloData helloData) {
		log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
		return "ok";
	}
	
}
```

- `@ModelAttribute HelloData helloData` 수행순서
    1. `HelloData` 객체 생성
    2. 요청 파라미터의 이름으로 `HelloData` 객체의 프로퍼티를 찾는다.  
    3. 해당 프로퍼티의 setter를 호출하여, 파라미터의 값을 입력(바인딩)한다.

        예) 파라미터 이름이 `username` 이면 `setUsername()` 메서드를 찾아서 호출하면서 값을 입력한다.

<br>

### `@ModelAttribute` 를 생략하여 객체에 바인딩하기

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
	 * @ModelAttribute 생략 가능
	 * String, int 같은 단순 타입 = @RequestParam
	 * argument resolver 로 지정해둔 타입 외 = @ModelAttribute
	 */
	@ResponseBody
	@RequestMapping("/model-attribute-v2")
	public String modelAttributeV2(HelloData helloData) {
		log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
		return "ok";
	}
	
}
```

- `@ModelAttribute` 가 생략되었다.
- 생략시 규칙
    - `String` , `int` , `Integer` 와 같은 단순 타입의 매개변수
        - `@RequestParam`
    - 나머지 타입의 매개변수
        - `@ModelAttribute` (argument resolver로 지정해둔 타입 외)

            > argument resolver 관련 내용은 나중에 자세히 다룬다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*