---
category: Test-Framework
tags: [Test-Framework]
title: "[테스트] 단위테스트: @WebMvcTest와 MockMvc"
date:   2021-09-21 14:30:00 
lastmod : 2021-09-21 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# `@WebMvcTest` 애너테이션과 MockMvc 활용

## `@WebMvcTest`

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

<br/>

### `@WebMvcTest` vs `@SpringBootTest`

- `@WebMvcTest`
    - **특정 레이어만을 테스트한다. (유닛 테스트)**
    - **만약 다른 레이어에 해당하는 빈이 필요하다면 Mock 객체로 만들어야한다.**
    - 레이어에 해당하는 빈만을 가져오므로 속도가 비교적 빠르다.
    - 웹애플리케이션 컨텍스트를 초기화 하지만 모든 설정을 불러오는 것이 아니라 MVC레이어와 관련된 설정만 불러온다.
    - HTTP 요청은 mock을 이용해 가짜로 이뤄지고 실제 연결은 생성되지 않는다.

<br/>

- `@SpringBootTest`
    - **Application Context 전체를 테스트한다. (통합 테스트)**
    - 애플리케이션의 모든 범위를 불러와야하므로 속도가 느리다.
    - 애플리케이션의 크기가 커질수록 불리하다.
    - 웹 애플리케이션 컨텍스트와 설정을 모두 불러와 실제 웹 서버에 연결을 시도한다.

<br/>

- **즉, `@WebMvcTest` 와 `@SpringBootTest` 의 가장 큰 차이점은 얼마나 많은 설정과 컨텍스트를 불러오느냐에 있다.**

<br/><br/>

## MockMvc

### MockMvc란?

- MockMvc는 서블릿 컨테이너의 구동 없이, 시뮬레이션된 MVC 환경에 모의 HTTP 서블릿 요청을 전송하는 기능을 제공하는 유틸리티 클래스다.
- 즉, 주로 Test에 사용되는 (특히 유닛테스트) 가짜 HTTP 통신 기능을 제공하는 클래스이다.

<br/>

### 관련 주요 메서드

- `MockMvc` 관련 메서드
    - `mockMvc.perform(RequestBuilder requestBuilder)`
        - 가짜 HTTP 요청을 수행하고 `ResultActions` 객체 (결과 데이터)를 반환한다.
    - [공식 API 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html)

<br/>

- `RequestBuilder` 관련 메서드
    - `get("요청URL")`, `post("요청URL")`, `put("요청URL")`, `delete("요청URL")` ...
        - 메서드 방식을 선택하여 가짜 요청을 보낸다.
    - `content()`
        - 요청 메시지의 바디를 직접 입력할 수 있다.
    - `contentType()`
        - 요청 메시지의 contentType 헤더를 설정한다.
    - `accept()`
        - 요청 메시지의 accept 헤더를 설정한다.
    - `session()`
        - 요청 메시지와 함께 보낼 세션을 설정한다.
    - [공식 API 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/RequestBuilder.html)

<br/>

- `ResultActions` 관련 메서드
    - `resultActions.andExpect(ResultMatcher matcher)`
        - `ResultActions` 객체 (결과 데이터)를 검증한다.
        - 매개변수(ResultMatcher)로 `status().isOk()` , `content()` 등을 받을 수 있다.
        - 검증할 수 있는 종류는 **[공식문서 참고](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/result/MockMvcResultMatchers.html)**
    - `resultActions.andDo(ResultHandler handler)`
        - 가짜요청에 응답한 결과에 대한 동작을 설정한다.
        - 주요 매개변수: `print()`
    - [공식 API 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultActions.html)

<br/>

- `ResultHandler` 관련 메서드
    - `print()`
        - `andDo()` 의 매개변수로 전달하여 응답 결과를 출력하도록 설정한다.
    - [공식 API 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultHandler.html)

<br/><br/><br/>

# 종합 예시

## spring-boot-starter-test 의존성 추가

- `@WebMvcTest` 를 비롯한 다양한 테스트 라이브러리를 사용하기 위해, spring-boot-starter-test 의존성을 추가해야한다.

<br/>

### `build.gradle` 파일

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-test'
}
```

<br/><br/>

## Lombok 의존성 설정

- 만약 프로젝트에서 Lombok을 사용한다면, 테스트 코드에서도 적용할 수 있도록 아래와 같이 설정해야한다.

<br/>

### `build.gradle` 파일

```java
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-test'

	//기존
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'

	//추가
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
}
```

<br/><br/>

## 프로젝트 소스

> **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/basic-web-CRUD)**

<br/>

### `BoardController` 클래스

```java
package web.crud.basic.basicweb.controller;

import ...

@RequiredArgsConstructor //build.gradle에 설정해두었기 때문에, 테스트코드에서도 작동한다.
@Controller
@RequestMapping("/board")
public class BoardController {

  private final BoardService boardService;

  @GetMapping("/{pageNum}")
  public String showBoard(@PathVariable int pageNum,
                          @SessionAttribute(name = "loginUser", required = false) User loginUser,
                          HttpServletRequest request,
                          Model model) {

    String currentPageNum = request.getRequestURI().split("/")[2];

    List<Article> articleList = boardService.getAllArticles();
    int[] totalPageNumArr = getTotalPageNumArr(articleList);

    Collections.reverse(articleList);
    articleList = getArticlesOfPage(pageNum, articleList);

    model.addAttribute("articleList", articleList);
    model.addAttribute("totalPageNumArr", totalPageNumArr);
    model.addAttribute("currentPageNum", currentPageNum);
    model.addAttribute("totalPageNum", totalPageNumArr.length);

    return "board";
  }

  private int[] getTotalPageNumArr(List<Article> TotalArticles) {
    double totalArticleSize = TotalArticles.size();
    int pageNum = (int) Math.ceil(totalArticleSize/3);
    int[] pageNumArr = new int[pageNum];
    for (int i=0; i<pageNumArr.length; i++) {
        pageNumArr[i] = i+1;
    }
    return pageNumArr;
  }

}
```

<br/>

### `BoardControllerTest` 테스트 클래스

```java
package web.crud.basic.basicweb.controller;

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.mock.web.MockHttpSession;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.ResultMatcher;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import web.crud.basic.basicweb.Service.BoardService;
import web.crud.basic.basicweb.WebConfig;
import web.crud.basic.basicweb.domain.Article;
import web.crud.basic.basicweb.domain.ArticleMapper;
import web.crud.basic.basicweb.domain.User;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@ExtendWith(MockitoExtension.class) //Mock 객체를 위한 확장모델 등록
@ContextConfiguration(classes = WebConfig.class) //Application Context(설정) 등록
@WebMvcTest(BoardController.class) //테스트할 컨트롤러 지정
class BoardControllerTest {

    @Mock
    private BoardService boardService;

    private MockMvc mockMvc; //@Autowired 가 없음에 주의!

    //MockMvc가 BoardController 를 사용할 수 있도록 설정
    @BeforeEach
    void setUp() {
      mockMvc = MockMvcBuilders.standaloneSetup(new BoardController(boardService)).build();
    }

    @DisplayName("게시판 보이기 테스트")
    @Test
    void showBoardTest(@Mock ArticleMapper articleMapper) throws Exception {
      //---------- given ----------
      List<Article> articleList = new ArrayList<>();
      Article article1 = new Article();
      Article article2 = new Article();
      MockHttpSession session = new MockHttpSession();
      User user = new User();

      article1.setTitle("Article1");
      article1.setContent("Article1 Content");
      article1.setWriter(1L);
      article1.setDateTime(LocalDateTime.now());

      article2.setTitle("Article2");
      article2.setContent("Article2 Content");
      article2.setWriter(1L);
      article2.setDateTime(LocalDateTime.now());

      articleList.add(article1);
      articleList.add(article2);

      user.setEmail("mockUser@test.com");
      user.setId(5L);
      user.setPassword("123test!");
      user.setName("mockUser");

      //로그인 사용자 세션 추가
      session.setAttribute("loginUser", user);

      given(boardService.getAllArticles()).willReturn(articleList);

      //---------- when ----------
      ResultActions actions = mockMvc.perform(get("/board/{pageNum}", 1)
          .session(session) //세션 정보 추가
      );

      //---------- then ----------
      actions.andExpect(status().isOk()).andDo(print());

    }

}
```

- `@ContextConfiguration(classes = WebConfig.class)`
    - **애플리케이션의 설정정보(Application Context)를 지정한다.**
    - `WebConfig.class` 는 `WebMvcConfigurer` 를 implements한 클래스이며, 인터럽트 등록 등의 작업을 설정한다.

<br/>

- `@WebMvcTest(BoardController.class)`
    - **`BoardController` 컨트롤러를 테스트할 것임을 설정한다.**

<br/>

- `@Mock private BoardService boardService;`
    - `BoardController` 를 생성하기 위해, 필요한 객체를 Mock으로 만든다.
    - **즉, `BoardController` 의 생성자로 받아야하는 `BoardService` 객체를 Mock으로 만든다.**

<br/>

- `private MockMvc mockMvc`
    - **`@Autowired` 를 적용하지 않는다. 왜냐하면 `@BeforeEach` 를 적용한 메서드를 통해, 테스트할 컨트롤러를 설정하여 `mockMvc` 변수에 저장할 것이기 때문이다.**

<br/>

- `mockMvc = MockMvcBuilders.standaloneSetup(new BoardController(boardService)).build();`
    - **`MockMvc`가 `BoardController` 를 사용할 수 있도록 설정한다.**

<br/>

- `actions.andExpect(status().isOk()).andDo(print());`
    - **요청에 대한 검증을 수행한다.**
    - 200 상태코드인지 확인하고
    - 응답 메시지를 콘솔에 출력한다.

<br/>

> 보다 자세한 사용예시는 다음 링크를 참고: **[사용 종합 예시 링크](https://howtodoinjava.com/spring-boot2/testing/spring-boot-mockmvc-example/)**

<br/>
<hr/>
<br/>

> 본 포스트는 갓대희님의 포스팅을 참고했습니다.  
관련 링크: [https://goddaehee.tistory.com/212?category=367461](https://goddaehee.tistory.com/212?category=367461)