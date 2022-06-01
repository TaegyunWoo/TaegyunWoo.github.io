---
category: Test-Framework
tags: [Test]
title: "[테스트] MockMvc와 실제 ServletContainer 통합 테스트"
date:   2022-06-01 23:10:00 
lastmod : 2022-06-01 23:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# MockMvc VS 실제 Servlet Container

## MockMvc를 활용한 테스트

### MockMvc란?

- MockMvc는 **서블릿 컨테이너의 구동 없이, 시뮬레이션된 MVC 환경에 모의 HTTP 서블릿 요청을 전송**하는 기능을 제공하는 유틸리티 클래스다.
- 즉, 주로 Test에 사용되는 (특히 유닛테스트) 가짜 HTTP 통신 기능을 제공하는 클래스이다.
- **실제 환경에서의 컨테이너 환경(DispatcherServlet 등)과 다르기 때문에, 예상치 못한 문제가 발생할 수 있다.**
  - MockMvc를 사용하면, Dispatcher Servlet이 MockUp 된다.
- **`@WebMvcTest`를 통해 오직 컨트롤러만 테스트할 때나, `@SpringBootTest`를 통해 통합테스트를 할 때, 모두 사용할 수 있다.**

> 보다 상세한 내용은 ["이 포스팅 글"](https://taegyunwoo.github.io/test-framework/TestFramework_Test_WebMvcTestAndMockMvc)을 참고하자.

<br/><br/>

## MockMvc를 활용한 통합 테스트 - 예시 코드
### `UserRestController` 클래스
- 아래 코드는 사용자 회원가입을 처리하는 컨트롤러 클래스이다.

```java
package com.four.brothers.runtou.controller;

import ...

@Slf4j
@RequiredArgsConstructor
@RestController
@RequestMapping("/api/user")
public class UserRestController {
  private final UserService userService;

  @Operation(summary = "유저 회원가입")
  @PostMapping("/signup/orderer")
  public SignUpAsOrdererResponse signUpAsOrderer(
    @Parameter(name = "회원 정보")
    @Validated @RequestBody SignUpAsOrdererRequest request,
    BindingResult bindingResult, HttpServletRequest requestMsg
  ) {
    boolean result = false;

    if (bindingResult.hasFieldErrors()) {
      throw new BadRequestException(RequestExceptionCode.WRONG_FORMAT, bindingResult.getFieldError().getDefaultMessage());
    }

    try {
      result = userService.signUpAsOrderer(request);
    } catch (DataIntegrityViolationException e) {
      throw new BadRequestException(SignupExceptionCode.ALREADY_EXIST_INFO, "회원가입 정보가 중복됩니다.");
    }

    return new SignUpAsOrdererResponse(result);
  }

}

```

<br/>

### `UserIntegrationTest` 테스트 클래스

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.four.brothers.runtou.dto.OrdererDto;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.transaction.annotation.Transactional;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@AutoConfigureMockMvc //MockMvc 사용 설정
@SpringBootTest(
  properties = {
    "spring.config.location=classpath:application-local.properties",
    "spring.config.location=classpath:application.properties"
  }
  /* 아래 속성은 Default이므로 생략할 수 있다.
   * webEnvironment = SpringBootTest.WebEnvironment.MOCK
   */
)
public class UserIntegrationTest {
  @Autowired
  private MockMvc mockMvc;
  @Autowired
  private ObjectMapper objectMapper;

  @Transactional
  @DisplayName("사용자 회원가입")
  @Test
  void userSignUpTest() throws Exception {
    //GIVEN
    String accountId1 = "testAccountId1";
    String realName1 = "testRealName1";
    String nickname1 = "testNickname1";
    String password1 = "testPassword1";
    String phoneNumber1 = "01011112222";
    String accountNumber1 = "1234567890";

    String content;

    OrdererDto.SignUpAsOrdererRequest request = new OrdererDto.SignUpAsOrdererRequest();
    request.setAccountId(accountId1);
    request.setRealName(realName1);
    request.setNickname(nickname1);
    request.setPassword(password1);
    request.setPhoneNumber(phoneNumber1);
    request.setAccountNumber(accountNumber1);

    content = objectMapper.writeValueAsString(request);

    //WHEN-THEN (MockUp된 Dispatcher Servlet에 요청을 보낸다.)
    mockMvc.perform(
      post("/api/user/signup/orderer")
        .content(content)
        .accept("*/*")
        .contentType(MediaType.APPLICATION_JSON)
    ).andExpect(status().isOk());
  }
}
```

<br/><br/>

## 실제 Servlet Container를 활용한 테스트

- 이번에는 실제 Servlet Container를 활용하여 테스트해보자.

<br/>

### `UserRestController` 클래스
- 위에서 작성한 코드와 완전히 동일하다.

<br/>

### `UserIntegrationTest` 테스트 클래스

```java
import com.four.brothers.runtou.dto.OrdererDto;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.transaction.annotation.Transactional;

@SpringBootTest(
  properties = {
    "spring.config.location=classpath:application-local.properties",
    "spring.config.location=classpath:application.properties"
  },
  //실제 서블릿 컨테이너를 사용하기 위해, 사용할 포트를 랜덤으로 설정
  webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT
)
public class UserIntegrationTest {
  //TestRestTemplete 객체를 사용하여, 테스트를 진행한다.
  @Autowired
  private TestRestTemplate template;

  @Transactional
  @DisplayName("사용자 회원가입")
  @Test
  void userSignUpTest() {
    //GIVEN
    String accountId1 = "testAccountId1";
    String realName1 = "testRealName1";
    String nickname1 = "testNickname1";
    String password1 = "testPassword1";
    String phoneNumber1 = "01011112222";
    String accountNumber1 = "1234567890";

    OrdererDto.SignUpAsOrdererRequest request = new OrdererDto.SignUpAsOrdererRequest();
    request.setAccountId(accountId1);
    request.setRealName(realName1);
    request.setNickname(nickname1);
    request.setPassword(password1);
    request.setPhoneNumber(phoneNumber1);
    request.setAccountNumber(accountNumber1);

    //WHEN (실제 서블릿 컨테이너에 요청을 보낸다.)
    HttpStatus resultStatusCode = template.postForEntity("/api/user/signup/orderer", request, OrdererDto.SignUpAsOrdererResponse.class)
      .getStatusCode();

    //THEN
    assertEquals(true, resultStatusCode.is2xxSuccessful());
  }
}
```