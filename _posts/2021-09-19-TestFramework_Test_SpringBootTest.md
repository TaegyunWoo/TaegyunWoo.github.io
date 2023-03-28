---
category: Test-Framework
tags: [Test-Framework]
title: "[테스트] 통합테스트: @SpringBootTest"
date:   2021-09-19 19:30:00 
lastmod : 2021-09-19 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# `@SpringBootTest`

## 개요

### `@SpringBootTest` 애너테이션이란?

- `@SpringBootTest`는 통합 테스트를 제공하는 기본적인 스프링 부트 테스트 어노테이션이다.

    > 통합 테스트: DB 접근 등 복합적인 동작을 수행하는 행위에 대한 테스트

- `@SpringBootTest` 어노테이션을 통해 스프링부트 어플리케이션 테스트에 필요한 거의 모든 의존성을 제공받을 수 있다.

<br/>

### JUnit 4에서 `@SpringBootTest` 사용하기

- JUnit 4에서 `@SpringBootTest` 를 사용할 땐, **반드시 `@RunWith(SpringRunner.class)` 를 사용해야한다.**

<br/>

### JUnit 5에서 `@SpringBootTest` 사용하기

- JUnit 4의 경우와는 다르게, `@SpringBootTest` 만 사용해도 된다.
- 즉, `@ExtendWith(SpringExtension.class)` 를 작성할 필요가 없다.

<br/><br/>

## `@SpringBootTest` 의 기능

### 프로퍼티 직접 추가

- **`@SpringBootTest` 가 제공하는 속성 `properties` 를 통해, 테스트 클래스에 특정 값을 전달해줄 수 있다.**
- "key=value" 형식으로 설정한다.

```java
@SpringBootTest(
	properties = {
		"my.value=MY_VALUE",
		"test=TEST"
	}
)
public class Test {
  @Value("${my.value}")
  private String a; // => MY_VALUE

  @Value("${test}")
  private String b; // => TEST

  @DisplayName("값 확인")
  @Test
  void test() {
	  System.out.println("a = " + a);
	  System.out.println("b = " + b);
  }
}
```

<br/>

### 스프링 설정 파일 변경

- 테스트 클래스는 기본적으로 `/src/main/resources/application.properties` 을 설정으로 사용한다.
- **`properties` 속성을 통해, 스프링의 설정파일을 `application.properties` 가 아닌 다른 것으로 설정할 수 있다.**

```java
@SpringBootTest(
  properties = {
    "spring.config.location=classpath:application-test.properties"
	}
)
public class Test {
  @Test
  void test() {}
}
```

<br/>

### 웹 테스트 환경 선택

- 테스트 클래스는 기본적으로 Mock 서블릿을 테스트 환경으로 선택한다.
    - Mock 서블릿이란
        - 내장된 실제 서블릿 컨테이너가 아닌 가짜 서블릿
    - Mock 서블릿을 사용하는 이유
        - **브라우저에서 요청과 응답을 의미하는 객체로서 Controller 테스트을 용이하게 해준다.**
    - `@AutoConfigureMockMvc`
        - `@AutoConfigureMockMvc` 과 MockMvc을 함께 사용하면, 별도의 설정없이 MockMvc를 사용할 수 있다.

<br/>

- **`webEnviroment` 속성을 통해, 테스트 코드를 MockMvc로 구동할 것인지, 실제 MVC로 구동할 것인지 선택할 수 있다.**
    - `webEnviroment=WebEnvironment.Mock` ⇒ MockMvc
    - `SpringBootTest.WebEnvironment.RANDOM_PORT` ⇒ 실제 내장 서블릿

<br/>

- `MockMvc` 객체의 메서드
    - `perform()`
        - 해당 메서드를 통해, 브라우저에서 서버에 URL 요청을 하듯 컨트롤러를 실행시킬 수 있다.
        - `RequestBuilder` 객체를 인자로 받는다.
        - `RequestBuilder` 객체는 `MockMvcRequestBuilder`의 정적 메소드를 이용해서 생성한다.
        - `MockMvcRequestBuilder` 은 GET, POST, PUT, DELETE 요청 방식과 매핑되는 `get()`, `post()`, `put()`, `delete()` 메소드를 제공한다.
    - `andExpect()`
        - `perform()` 을 통해 전송한 요청에 대한 응답을 검증한다.
        - 응답 상태 코드 검증: `status()`
        - 뷰/리다이렉트 검증: `view()`
        - 모델 정보 검증: `model()`

<br/>

- **테스트 환경을 MockMvc로 선택**

    ```java
    @AutoConfigureMockMvc
    @SpringBootTest(
      webEnvironment = SpringBootTest.WebEnvironment.MOCK
    )
    class ArticleMapperTest {

      @Autowired
      private MockMvc mockMvc;

      @Test
      void test() throws Exception {
        mockMvc.perform(get("/board/1"))
            .andExpect(status().isOk());
      }

    }
    ```

<br/>

- **테스트 환경을 내부 서블릿(실제 MVC)로 선택**

    ```java
    @SpringBootTest(
      webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT
    )
    class ArticleMapperTest {

      @Test
      void test() throws Exception {
    		//테스트코드
      }

    }
    ```

<br/><br/>

## 참고: `@Transactional` 애너테이션

### `@Transactional` 애너테이션이란?

- 트랜잭션 테스트시, 테스트 데이터를 DB서버에 남기지 않도록 Rollback 시키는 애너테이션이다.
- **RAMDOM_PORT 등을 통해 테스트 실행시, 별도의 쓰레드에서 작동하기 때문에 Rollback되지 않는다.**

<br/>

### 적용 코드

```java
@AutoConfigureMybatis
@SpringBootTest
class ArticleMapperTest {

  @Autowired
  private ArticleMapper articleMapper;

  @Transactional
  @DisplayName("Insert & Select 테스트")
  @Test
  void test() {
    Article article = new Article();
    article.setTitle("Test");
    article.setDateTime(LocalDateTime.now());
    article.setWriter(1L);
    article.setContent("Test Content");
    articleMapper.insertArticle(article);

    Article selectArticle = articleMapper.getById(article.getId());

    assertEquals(article, selectArticle);
  }
}
```

<br>

---

<br>

- *본 게시글은 갓대희님의 포스팅을 참고하여 작성한 글입니다.*  
[갓대희의 작은공간](https://goddaehee.tistory.com/211)