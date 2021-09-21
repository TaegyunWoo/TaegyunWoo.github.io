---
category: Test-Framework
tags: [Test]
title: "[테스트] 단위테스트: @WebMvcTest"
date:   2021-09-21 14:30:00 
lastmod : 2021-09-21 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# `@WebMvcTest` 애너테이션

## 개요

### `@WebMvcTest` 애너테이션이란?

- MVC를 위한 테스트나, 컨트롤러를 테스트할 때 사용되는 애너테이션이다.
- **`@WebMvcTest` 사용시, 스캔 대상을 제한하여 `@SpringBootTest` 보다 빠른 효율을 보여준다.**
- `@WebMvcTest` 가 스캔하는 대상 목록
    - `@Controller`
    - `@ControllerAdvice`
    - `@JsonComponent`
    - `Converter`
    - `GenericConverter`
    - `Filter`
    - `HandlerInterceptor`
    - `WebMvcConfigurer`
    - `HandlerMethodArgumentResolver`

<br/>

### `@WebMvcTest` 가 자동으로 추가하는 항목 리스트

- `@WebMvcTest` 가 **Auto Configure**하는 항목은 다음 문서에서 확인할 수 있다.
- `@WebMvcTest` 뿐만 아니라, 다양한 Test에 대한 Auto Configure 항목을 확인할 수 있다.

[공식 문서 링크](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)

<br/>

### 장점

- Web Application과 관련된 Bean들만 등록하기 때문에, 통합 테스트(`@SpringBootTest`)보다 빠르게 수행된다.
- 통합 테스트로 수행하기 힘든 테스트를 수행할 수 있다.
    - Ex) 결제 모듈 테스트시, 가짜 Mock 객체를 통해 테스트를 할 수 있다.

<br/>

### 단점

- 요청부터 응답까지 모든 테스트를 Mock 기반으로 테스트하기 때문에 실제 환경에서는 제대로 동작하지 않을 수 있다.

<br/><br/>

## spring-boot-starter-test 의존성 추가

- `@WebMvcTest` 를 비롯한 다양한 테스트 라이브러리를 사용하기 위해, spring-boot-starter-test 의존성을 추가해야한다.

<br/>

### `build.gradle` 파일

```groovy
dependencies {
	implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0' //mybatis
	implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter-test:2.2.0' //mybatis test
}
```

<br/><br/>

## 예시 코드

```java
//import 생략

@AutoConfigureMybatis // 마이바티스
@WebMvcTest(BoardController.class)
class BoardControllerTest {
  @Autowired
  private ArticleMapper articleMapper;

  @Autowired
  private MockMvc mockMvc; //@WebMvcTest가 자동으로 @AutoConfigureMockMvc를 추가해준다.

	@Test
	void test() {
		// Test code
	}

}
```

> 본 포스트는 갓대희님의 포스팅을 참고했습니다.  
관련 링크: [https://goddaehee.tistory.com/212?category=367461](https://goddaehee.tistory.com/212?category=367461)