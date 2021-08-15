---
category: Spring-MVC
tags: [스프링, MVC, 로그인]
title: "[스프링 - MVC] 로그인처리 - 쿠키와 세션"
date:   2021-08-15 17:00:00 
lastmod : 2021-08-15 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글: [로그인처리 - 기본 프로젝트 설정](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginBasicProject)

<br/><br/>

# 쿠키

## 개요

- 로그인 기능을 구현할 때, 쿠키를 사용할 수 있다.
- 쿠키를 사용하면, 사용자의 로그인 상태를 지속적으로 유지할 수 있다.

<br/>

### 목표

- 사용자가 로그인을 하면, 로그인 상태가 유지된다.
- 로그인 직후, 회원 전용 페이지로 리다이렉션된다.

<br/><br/>

## 쿠키 동작 원리

- **쿠키 생성**

    ![쿠키 생성](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2029.png)

<br/>

- **클라이언트 요청**

    ![클라이언트 요청](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2030.png)

<br/><br/>

## 쿠키의 종류

쿠키의 종류는 아래와 같다.

- **영속 쿠키**
    - 만료 날짜를 입력하면 해당 날짜까지 유지된다.
- **세션 쿠키**
    - 만료 날짜를 생략하면 브라우저 종료시 까지만 유지된다.

<br/><br/>

## 쿠키와 보안 문제

쿠키를 사용해서 로그인 Id를 서버에 전달하면, 로그인을 유지할 수 있다. 하지만, 여기에는 보안 문제가 있다.

<br/>

### 보안 문제

- **쿠키 값이 임의로 변경될 수 있다.**
    - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
- **쿠키에 보관된 정보는 훔쳐갈 수 있다.**
    - 클라이언트의 PC가 털려서, 쿠키 정보를 가져갈 수 있다.
- **해커가 쿠키를 한번 훔치면 평생 사용할 수 있다.**
    - 쿠키에 저장된 사용자 정보를 해커가 계속해서 사용할 수 있다.

<br/>

### 대안

- **쿠키에 중요한 값(사용자 정보 등)을 노출하지 않는다. 사용자 별로 예측 불가능한 임의의 토큰을 노출시킨 후, 서버에서 토큰과 사용자 id를 매핑해서 인식한다.**
- **토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.**
    - (문제 예. "토큰=1"일때, "토큰=2"를 넣으면 다른 사용자의 계정에 접근할 수 있겠다! )
- **토큰 만료시간을 짧게 유지하여, 해커가 털어가도 시간이 지나면 사용할 수 없도록 한다.**

<br/><br/><br/>

# 세션

## 개요

- 쿠키를 사용하여 로그인 기능을 구현하면, 심각한 보안 문제가 발생한다.
- 세션을 사용하여 해결할 수 있다.

<br/><br/>

## 세션 동작 원리

- **로그인**

    ![로그인](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2031.png)

<br/>

- **세션 생성**

    ![세션 생성](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2032.png)

<br/>

- **세션 id를 쿠키로 전달 (클라이언트 측에 전달)**

    ![클라이언트측전달](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2033.png)

<br/>

- **세션 id를 쿠키에서 전달 (서버 측에 전달)**

    ![서버측전달](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2034.png)

<br/>

### 주요 포인트

- **회원 관련 정보는 전혀 클라이언트에 전달하지 않고, 오직 서버에서만 관리한다.**
    - 클라이언트 측 쿠키가 털려도, 거기에는 중요한 정보가 없다.
- **추정 불가능한 세션 id만 쿠키를 통해 클라이언트에 전달한다.**
    - 하나의 세션 id을 통해, 다른 사용자의 세션 id를 유추할 수 없다.
- **세션의 만료시간을 짧게 유지한다.**
    - 세션 id가 털려도, 시간이 지나면 사용할 수 없다.

<br/><br/>

## 로그인 처리: 세션 직접 구현

> [이전에 작성해둔 프로젝트](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginBasicProject#1)를 기반으로 진행한다.

먼저 세션을 직접 구현해보고, 그 다음 자바와 스프링이 제공하는 세션을 사용해보자.

<br/>

### 구현할 기능

- **세션 생성**
    - sessionId 생성 (임의의 추정 불가능한 랜덤 값)
    - 세션 저장소에 sessionId와 보관할 값 저장
    - 응답쿠키를 생성해서 sessionId 값을 클라이언트에 전달

<br/>

- **세션 조회**
    - 클라이언트가 요청(전송)한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회

<br/>

- **세션 만료**
    - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 sessionId와 값 제거

<br/>

### 세션 기능 구현: `SessionManager` 클래스

```java
import ...

@Component
public class SessionManager {

  //클라이언트에 전달할 세션 id 값의 Key 값
  //(mySessionId, 세선id)
  public static final String SESSION_COOKIE_NAME = "mySessionId";

  //세션 저장소
  //CocurrentHashMap => 동시성 문제 해결
  private Map<String, Object> sessionStorage = new ConcurrentHashMap<>();

  /*
   세션 생성
   */
  public void createSession(Object value, HttpServletResponse response) {
      String sessionId = UUID.randomUUID().toString();

      //세션 저장소에 저장
      sessionStorage.put(sessionId, value);

      //세션 id를 쿠키로 응답
      Cookie cookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
			cookie.setPath("/"); //경로 설정
      response.addCookie(cookie);
  }

  /*
   세션 조회
   */
  public Object getSession(HttpServletRequest request) {
      Cookie[] cookies = request.getCookies();
      Cookie sessionIdCookie = findSessionIdCookie(cookies, SESSION_COOKIE_NAME);

      if (sessionIdCookie == null) {
          return null;
      }

      return sessionStorage.get(sessionIdCookie.getValue());
  }

  /*
   세션 만료
   */
  public void expire(HttpServletRequest request) {
      Cookie sessionIdCookie = findSessionIdCookie(request.getCookies(), SESSION_COOKIE_NAME);
      if (sessionIdCookie != null) {
          sessionStorage.remove(sessionIdCookie.getValue());
      }
  }

  /**
   * @return 해당 cookie이름을 찾으면 cookie, 못찾으면 null
   */
  private Cookie findSessionIdCookie(Cookie[] cookies, String cookieName) {
      for (Cookie cookie : cookies) {
          if (cookie.getName().equals(cookieName)) {
              return cookie;
          }
      }
      return null;
  }

}
```

- 쿠키를 클라이언트에게 줄때, `setPath()` 를 통해 경로를 지정해야 정상적으로 작동한다.

<br/>

### 세션 적용: `LoginController` 클래스

```java
/**
 * 로그인
 */
@PostMapping("/login")
public String login(@Validated @ModelAttribute("user") LoginUserForm form,
                    BindingResult bindingResult,
                    HttpServletResponse response) {

    //검증 오류시
    if (bindingResult.hasErrors()) {
        return "login";
    }

    //로그인 실행
    User user = loginService.login(form.getId(), form.getPassword());

    //로그인 실패시, 오브젝트 에러
    if (user == null) {
        bindingResult.reject("wrongUser", "아이디나 비밀번호를 확인하세요.");
        return "login";
    }

    //로그인 성공시
    
    //세션 생성
    sessionManager.createSession(user, response);
    
    return "redirect:/";
}

/**
 * 로그아웃
 */
@PostMapping("/logout")
public String logout(HttpServletRequest request) {
    
    //세션 종료
    sessionManager.expire(request);

    return "redirect:/";
}
```

<br/>

### 세션 활용: `HomeController` 클래스

- 로그인된 사용자 ⇒ 사용자 정보 페이지
- 로그인 안된 사용자 ⇒ 기본 home 페이지

```java
import ...

@Controller
public class HomeController {

    private final SessionManager sessionManager;

    @Autowired
    public HomeController(SessionManager sessionManager) {
        this.sessionManager = sessionManager;
    }

    @GetMapping("/")
    public String home(HttpServletRequest request, Model model) {
        User user = (User) sessionManager.getSession(request);

        //로그인 X 일때, 기본 홈페이지로
        if (user == null) {
            return "home";
        }

        model.addAttribute("user", user);

        //로그인 O 일때, 사용자 정보 페이지로
        return "userPage";
    }
}
```

<br/>

### 결과

- **첫 접속 (로그인 X일때)**

    ![첫 접속](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2035.png)

<br/>

- **로그인**

    ![로그인](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2036.png)

<br/>

- **로그인 성공**

    ![로그인 성공](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2037.png)

<br/><br/>

## 로그인 처리: 서블릿이 제공하는 세션

- 서블릿은 `HttpSession` 이라는 기능을 통해 세션을 지원한다.

<br/>

### `HttpSession` 이란?

- `HttpSession` 도 위에서 구현한 것처럼 동작한다.
- 서블릿을 통해 `HttpSession` 을 생성하면 다음과 같은 쿠키를 생성한다.
    - `JSESSIONID=추정불가능한값`

<br/>

### 세션 적용: `LoginController` 클래스

```java
import ...

@Controller
@RequestMapping("/member")
public class LoginController {

  private final LoginService loginService;

  @Autowired
  public LoginController(LoginService loginService, SessionManager sessionManager) {
      this.loginService = loginService;
      this.sessionManager = sessionManager;
  }

  @GetMapping("/login")
  public String viewUserLoginForm(@ModelAttribute User user) {
      return "login";
  }

  @PostMapping("/login")
  public String login(@Validated @ModelAttribute("user") LoginUserForm form,
                      BindingResult bindingResult,
                      HttpServletRequest request) {

      //검증 오류시
      if (bindingResult.hasErrors()) {
          return "login";
      }

      //로그인 실행
      User user = loginService.login(form.getId(), form.getPassword());

      //로그인 실패시, 오브젝트 에러
      if (user == null) {
          bindingResult.reject("wrongUser", "아이디나 비밀번호를 확인하세요.");
          return "login";
      }

      //로그인 성공시

      //세션 생성 후, 값 추가
      HttpSession session = request.getSession();
      session.setAttribute("loginUser", user);

      return "redirect:/";
  }

  /**
   * 로그아웃
   */
  @GetMapping("/logout")
  public String logout(HttpServletRequest request) {

      //세션 삭제
      if (request.getSession(false) != null) {
          request.getSession().invalidate();
      }

      return "redirect:/";
  }

}
```

- **`request.getSession(true)` , `request.getSession()`**
    - 세션이 있으면 기존 세션을 반환한다.

        > **세션**: 이전에 구현해본 `sessionStorage` 와 유사한 개념

    - **세션은 사용자별로 생성된다.**
    - 세션이 없으면 새로운 세션을 생성해서 반환한다.

<br/>

- **`request.getSession(false)`**
    - 세션이 있으면 기존 세션을 반환한다.
    - 세션이 없으면 세션을 생성하지 않고, null을 반환한다.

<br/>

- **`HttpSession` 원리**

    ![HttpSession](/assets/img/2021-08-15-SPRING_MVC_LoginSession/Untitled%2038.png)

    - 하나의 세션에 여러가지 값을 저장할 수 있다.

<br/>

### 세션 활용: `HomeController` 클래스

```java
package prac.myPrac.controller;

import ...

@Controller
public class HomeController {

  private final SessionManager sessionManager;

  @Autowired
  public HomeController(SessionManager sessionManager) {
      this.sessionManager = sessionManager;
  }

	/**
	 * @SessionAttribute 를 통해 세션에 접근
	 */
  @GetMapping("/")
  public String home(
          @SessionAttribute(name = "loginUser", required = false) User user,
          Model model) {

      //로그인 X 일때, 기본 홈페이지로
      if (user == null) {
          return "home";
      }

      model.addAttribute("user", user);

      //로그인 O 일때, 사용자 정보 페이지로
      return "userPage";
  }
}
```

- `@SessionAttribute(name = "loginUser" , required = false)`
    - `request.getSession(false).getAttribute("loginUser")` 와 동일한 기능을 한다.
    - `@SessionAttribute` 는 세션을 생성하지 않는다.
    - `required = false` 는 `user` 객체에 값을 필수적으로 받지 않아도 된다는 뜻이다.

<br/><br/>

## 세션의 부가기능

### TrackingModes

- 로그인을 처음 시도하면 URL이 `jsessionid` 값을 포함하고 있다.
    - 예시) `http://localhost:8080/;jsessionid=F26137183SASD23`
- 이것은 웹 브라우저가 쿠키를 지원하지 않을 때에 대비하여 작성된 URL이다.
- 해당 기능이 바로 TrackingModes 이다.
- 해당 기능을 끄는 것을 권장한다.
    - `application.properties` 파일에 작성
        - `server.servlet.session.tracking-modes=cookie`

<br/>

### 세션 정보 확인

- `session.getId()`
    - 세션Id, 즉 `jsessionid` 값이다.
- `session.getMaxInactiveInterval()`
    - 세션의 유효 시간 (초 단위)
- `session.getCreationTime()`
    - 세션 생성일시
    - java.util.Date 타입 반환
- `session.getLastAccessedTime()`
    - 세션과 연결된 사용자가 최근에 서버에 접근한 시간
    - java.util.Date 타입 반환
- `session.isNew()`
    - 새로 생성된 세션인지, 아닌지

<br/>

### 세션 기본 종료 시점

- **세션은 사용자가 서버에 최근에 요청한 시간을 기준으로 30분 정도를 유지한다.**
    - 세션유지 시간 = `session.getLastAccessedTime()` + 30분

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*