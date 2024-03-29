---
category: TS
tags: [TS]
title: "로그아웃 후 뒤로가기의 캐시문제"
date:   2021-09-04 12:08:00 
lastmod : 2021-09-04 12:08:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 문제 발생 개발기록
  - [CRUD Web 개발일지: 2021-09-04](https://taegyunwoo.github.io/CRUD_Web/2021-09-04)

<br/>

# 문제 배경

- 로그아웃시, 세션이 만료되도록 코드를 작성했다.
- 그리고 로그아웃 후 로그인 페이지로 Redirect 되도록 하였다.
- **하지만 로그아웃을 하고난 후, 뒤로가기를 누르면 로그인시에만 접근할 수 있는 페이지가 그대로 나타났다.**

<br><br>

# 문제 해결 탐색과정
- 뒤로가기를 누르면 비정상적으로 페이지가 출력되지만, 해당 페이지에서 새로고침을 통해 재요청을 하면 로그인 인증 인터셉터가 정상적으로 동작하여, 홈화면으로 Redirect 되는 것에 주목하였다.
- 즉, 브라우저의 캐시 기능이 이와 같은 문제를 만든다고 의심하여 관련 자료를 찾아보았다.
- HTTP와 캐시 관련 자료를 조사하였다.
- HTTP 헤더를 통해 캐시를 조작할 수 있음을 확인하였다.

<br><br>

# 문제 해결
## 문제 원인

- 뒤로가기를 눌렀을 때, 사용자의 브라우저는 Cache된 페이지를 보여주기 때문에 발생하는 문제이다.
- 요청과 응답의 흐름
  1. 로그인 요청(POST): `/login`
  2. 로그인 성공 응답: `redirect:/board`
  3. 302 리다이렉션 요청(GET): `/board`
  4. 서버의 응답: 응답메시지(`board.html`)
  4. **해당 응답을 캐시에 저장**
  5. 로그아웃 요청(GET): `/logout`
  6. 로그아웃 응답: `redirect:/home`
  7. 뒤로가기: `/board` 요청
  8. **캐싱된 응답을 브라우저가 가져옴**

<br><br>

## 해결방법

- 사용자가 로그인에 성공하여 `/board` 로 Redirect될 때, 해당 응답를 캐싱할 수 없도록 한다.
- **로그인 처리 컨트롤러 호출시 로그인에 성공한다면, `redirect:/board` 로 응답하게 된다. 이때 응답헤더를 통해 해당 응답을 캐싱하지 않도록 설정한다.**
  - Interceptor를 통해 구현할 수 있다. (postHandle 메서드 활용)
- 이를 통해 캐싱된 응답이 존재하지 않을 때, `/board`를 한번 더 요청(뒤로가기)하면 서버에게 다시 요청해야한다.

<br/>

### 해결순서

1. 인터셉터를 아래와 같이 작성한다.  
    ```java
    public class LogoutInterceptor implements HandlerInterceptor {
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
          response.setHeader("Pragma", "no-cache");
          response.setHeader("Cache-Control", "no-cache");
          response.setHeader("Cache-Control", "no-store");
          response.setDateHeader("Expires", 0L);
      }
    }
    ```
    - 스프링이 제공하는 인터셉터 기능을 통해, 응답 헤더를 설정할 수 있다.
    - `postHandle` 메서드는 핸들러가 호출된 후, 호출되는 메서드이므로 여기서 응답헤더를 설정한다.
  
2. 인터셉터를 등록한다.
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LogoutInterceptor())
                    .order(1)
                    .addPathPatterns("/board");
        }
    }
    ```
    - 인터셉터를 적용할 요청URL은 `/board`이다.
      - 로그인 핸들러(컨트롤러)가 로그인 성공시 `redirect:/board` 를 반환하여 `/board`를 클라이언트가 다시 요청(응답상태가 302이므로)하게 한다.
      - 클라이언트가 `/board`를 다시 요청할 때, 적용되는 인터셉터이므로 `/board`와 매칭해야한다.
      - 즉, 클라이언트가 `/board`를 요청하고 이에 대한 응답을 서버가 하게될 때, 해당 응답을 캐싱하지 않도록 헤더로 설정한다.