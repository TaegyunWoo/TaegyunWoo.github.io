---
category: Spring-MVC
tags: [스프링, MVC, 로그인, 필터]
title: "[스프링 - MVC] 로그인처리 - 필터"
date:   2021-08-15 19:00:00 
lastmod : 2021-08-15 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    1. [로그인처리 - 기본 프로젝트 설정](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginBasicProject)
    2. [로그인처리 - 쿠키와 세션](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginSession)

<br/>

# 서블릿 필터

## 개요

- [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginSession)에서 다룬 프로젝트에 다음 코드를 추가하자.
    - 뷰 템플릿 (`userPage.html`) - 기존 코드 수정

        ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
          <style>
            body {
              font-size: larger;
            }
          </style>
        </head>
        <body>
          <h1>사용자 페이지</h1>
          아이디: <span th:text="${user.id}">사용자 아이디</span> <br/>
          이름: <span th:text="${user.name}">사용자 이름</span> <br/>

        	<!-- 추가된 버튼 -->
          <button th:onclick="|location.href='@{/service}'|">서비스</button>
          <button th:onclick="|location.href='@{/member/logout}'|">로그아웃</button>
        </body>
        </html>
        ```

    - 뷰 템플릿 (`serviceOnlyForUser.html`)

        ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
        </head>
        <body>
          <h1>로그인한 사용자만 접근할 수 있는 페이지</h1>
        </body>
        </html>
        ```

    - 컨트롤러 (`OnlyUserController.java`)

        ```java
        package prac.myPrac.controller;

        import ...

        @Controller
        @RequestMapping("/service")
        public class OnlyUserController {

            @GetMapping
            public String viewForUser(@ModelAttribute("user") User user) {
                return "serviceOnlyForUser";
            }
        }
        ```

- 로그인된 사용자만 접근할 수 있는 서비스를 위해, 위와 같은 코드를 추가했다.
- **하지만, 기대와는 달리 URL만 알고있다면 누구나 접근할 수 있다. (`http://localhost/service`)**
- 해당 문제를 수정해보자!

<br/>

### 의도한 애플리케이션 흐름

- **첫 접속**

    ![첫 접속](/assets/img/2021-08-15-SPRING_MVC_LoginFilter/Untitled%2039.png)

<br/>

- **로그인**

    ![로그인](/assets/img/2021-08-15-SPRING_MVC_LoginFilter/Untitled%2040.png)

<br/>

- **서비스 접근**

    ![서비스 접근](/assets/img/2021-08-15-SPRING_MVC_LoginFilter/Untitled%2041.png)

<br/>

### 목표

- 로그인 한 사용자만 서비스 페이지로 넘어갈 수 있다.
- 해결해야 하는 문제
    - 로그인을 하지 않아도, URL을 직접 입력하여 서비스 페이지로 접근할 수 있다.

<br/><br/>

## 서블릿 필터

### 서블릿 필터란?

- 서블릿이 지원하는 수문장이다.

<br/>

### 필터 흐름

```
HTTP요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```

- 필터 적용시, **필터가 호출된 뒤 서블릿이 호출된다.**
- 참고: 스프링을 사용하는 경우, **서블릿==디스패처_서블릿**

<br/>

### 필터 제한

```
로그인O: HTTP요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
로그인X: HTTP요청 -> WAS -> 필터 -> (비정상적인 요청이라고 판단, 서블릿 호출X)
```

<br/>

### 필터 체인

```
HTTP요청 -> WAS -> 필터1 -> 필터2 -> ... -> 서블릿 -> 컨트롤러
```

- 필터는 체인으로 구성된다.
- 중간에 필터를 자유롭게 추가할 수 있다.

<br/><br/>

## 인증 체크 필터 구현

### 인증 체크 필터: `LoginCheckFilter` 클래스

```java
import ...

public class LoginCheckFilter implements Filter {

    private static final String[] whitelist = {"/", "/member/add", "/member/login", "/member/logout"};

    /**
     * 실제 필터 로직 작성
     */
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            if (isLoginCheckPath(httpRequest.getRequestURI())) {
                HttpSession session = httpRequest.getSession(false);
                if (session == null || session.getAttribute("loginUser") == null) {

                    /*
                    로그인 X일때 login 화면으로 redirect
                    다시 돌아올 url을 파라미터로 같이 넘겨줌
                     */
                    httpResponse.sendRedirect("/member/login?redirectURL=" +
                            httpRequest.getRequestURI());

                    return; //더이상 진행X
                }
            }
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        }

    }

    /**
     * 서블릿 컨테이너가 생성될 때 호출된다.
     */
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    /**
     * 서블릿 컨테이너가 종료될 때 호출된다.
     */
    @Override
    public void destroy() {
        Filter.super.destroy();
    }

    /**
     * whitelist 요소에 포함된 url => false
     * whitelist 요소에 포함되지 않은 url => true
     */
    private boolean isLoginCheckPath(String requestUrl) {
        return !PatternMatchUtils.simpleMatch(whitelist, requestUrl);
    }
}
```

- `implements Filter`
    - 필터를 구현할 때는, 해당 인터페이스를 구현해야한다.

<br/>

- `init()`
    - 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.

<br/>

- `doFilter()`
    - 고객의 요청이 올 때마다, 해당 메서드가 호출된다.
    - 필터의 로직을 구현하면 된다.

<br/>

- `destroy()`
    - 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

<br/>

- `whitelist = {"/", "/member/add", "/member/login", "/member/logout"}`
    - 인증 필터를 적용해도, 해당 url에는 접근할 수 있어야한다.

<br/>

- `isLoginCheckPath(requestURI)`
    - 화이트 리스트를 제외한 모든 경우에 인증 체크 로직을 적용한다.

<br/>

- `httpResponse.sendRedirect("/member/login?redirectURL=" + httpRequest.getRequestURI())`
    - 미인증 사용자는 로그인 화면으로 리다이렉트한다.
    - 로그인 화면에서 로그인을 완료하면, 다시 원래 페이지로 돌아올 수 있다.
        - 쿼리 스트링으로 돌아올 URL을 로그인 컨트롤러에 전달했기 때문에

<br/>

- `return;`
    - 미인증 사용자가 컨트롤러를 호출할 수 없도록 막는다.
    - 앞서서 `sendRedirect` 를 사용했기 때문에, `redirect`가 응답으로 적용되고 요청이 끝난다.

<br/>

### 필터 등록: `WebConfig` 클래스

```java
import ...

@Configuration
public class WebConfig {
    
    @Bean
    public FilterRegistrationBean loginCheckFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<Filter>();
        filterFilterRegistrationBean.setFilter(new LoginCheckFilter()); // 로그인 필터 등록
        filterFilterRegistrationBean.setOrder(1); // 필터 순서=1
        filterFilterRegistrationBean.addUrlPatterns("/*"); // 적용할 url
        
        return filterFilterRegistrationBean;
    }
}
```

- `setFilter(new LoginCheckFilter())`
    - LoginCheckFilter 를 필터로 등록한다.
- `setOrder(1)`
    - 필터 적용 순서이다.
    - 첫번째 순서로 호출된다.
- `addUrlPatterns("/*")`
    - 해당 필터를 적용할 url을 설정한다.
    - /* 은 모든 url을 의미한다.

<br/>

### 로그인 성공시 기존 url로 돌아오기: `LoginController` 클래스

```java
@PostMapping("/login")
public String login(@Validated @ModelAttribute("user") LoginUserForm form,
                    BindingResult bindingResult,
                    @RequestParam(defaultValue = "/") String redirectURL ,
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
  session.setAttribute(SessionConst.LOGIN_USER, user);

	//다시 돌아가기
  return "redirect:" + redirectURL;
}
```

- `@RequestParam(defaultValue = "/") String redirectURL`
    - 해당 매개변수 이름을 갖는 쿼리 스트링을 받아온다.
    - `defaultValue` : 해당 이름이 없을 때의 기본 값

<br/>

### 결과

- **로그인X, 바로 URL(`http://localhost:8080/service`) 접근**

    ![미인증시](/assets/img/2021-08-15-SPRING_MVC_LoginFilter/Untitled%2042.png)

<br/>

- **로그인 완료**

    ![로그인 완료시](/assets/img/2021-08-15-SPRING_MVC_LoginFilter/Untitled%2043.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*