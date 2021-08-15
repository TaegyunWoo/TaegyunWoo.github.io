---
category: Spring-MVC
tags: [스프링, MVC, 로그인, 인터셉터]
title: "[스프링 - MVC] 로그인처리 - 인터셉터"
date:   2021-08-15 20:30:00 
lastmod : 2021-08-15 20:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    1. [로그인처리 - 기본 프로젝트 설정](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginBasicProject)
    2. [로그인처리 - 쿠키와 세션](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginSession)
    3. [로그인처리 - 필터](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginFilter)

<br/>

# 인터셉터

## 개요

- 스프링 인터셉터는 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다.

<br/>

## 스프링 인터셉터의 흐름들

### 스프링 인터셉터 흐름

```
HTTP요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
```

- 스프링 인터셉터는 **디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출된다.**
- 스프링 인터셉터에는 URL 패턴을 보다 정밀하게 설정할 수 있다.

<br/>

### 스프링 인터셉터 제한

```
로그인O: HTTP요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
로그인X: HTTP요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터 -> (비정상적인 요청이라고 판단, 서블릿 호출X)
```

<br/>

### 스프링 인터셉터 체인

```
HTTP요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> .. -> 컨트롤러
```

- 스프링 인터셉터는 체인으로 구성된다.
- 중간에 인터셉터를 자유롭게 추가할 수 있다.

<br/><br/>

## 스프링 인터셉터 호출 흐름

![스프링 인터셉터 호출 흐름](/assets/img/2021-08-15-SPRING_MVC_LoginInterceptor/Untitled%2044.png)

> 스프링의 구조에 대해서는 [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Structure#0)을 참고하자.

<br/>

### 정상 흐름

- **`preHandle`**
    - 핸들러 어댑터 호출 전에 호출된다.
    - `preHandle` 의 응답값이 `true` 인 경우
        - 다음으로 진행한다.
    - `preHandle` 의 응답값이 `false` 인 경우
        - 더이상 진행하지 않는다.
- **`postHandle`**
    - 핸들러 어댑터 호출 이후에 호출된다.
- **`afterCompletion`**
    - 뷰가 렌더링 된 이후에 호출된다.

<br/>

### 인터셉터 예외 흐름

![인터셉터 예외 흐름](/assets/img/2021-08-15-SPRING_MVC_LoginInterceptor/Untitled%2045.png)

- **예외 발생시**
    - `preHandle`
        - 핸들러 어댑터 호출 전에 호출된다.
    - `postHandle`
        - **컨트롤러(핸들러)에서 예외가 발생하면 호출되지 않는다.**
    - `afterCompletion`
        - **항상 호출된다. (따라서, 예외와 무관한 로직이 필요할 때 사용할 수 있다)**
        - 이 경우 예외를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.

<br/><br/>

## 인증 체크 인터셉터 구현

### 인증 체크 인터셉터: `LoginCheckInterceptor` 클래스

```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {

  /**
   * 핸들러 어댑터 호출 전, 실행되는 메서드
   */
  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
      HttpSession session = request.getSession(false);
      String requestURL = request.getRequestURI();

      if (session == null || session.getAttribute("loginUser") == null) {
          response.sendRedirect("/member/login?redirectURL=" + requestURL);
          return false;
      }
      return true;
  }

	/**
	 * 핸들러 호출 후, 실행되는 메서드
	 */
  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
      log.info("postHandle 호출");
  }

	/**
	 * 항상 실행되는 메서드
	 */
  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
      log.info("afterCompletion 호출");
  }
}
```

- `implements HandlerInterceptor`
    - 인터셉터를 구현하기 위해 `HandlerInterceptor` 를 `implements` 해야한다.

<br/>

- `preHandle()` 의 반환값
    - `true` : 계속 진행 (핸들러 호출)
    - `false` : 중단 (핸들러 호출 X)

<br/>

### 인터셉터 등록: `WebConfig` 클래스

```java
import ...

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
				//아래 코드는 규격화된(문법과 같이) 패턴이다.
        registry.addInterceptor(new LoginCheckInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/member/add", "/member/login", "/member/logout");
    }
}
```

- `implements WebMvcConfigurer`
    - 인터셉터를 등록하기 위해, Configuration 클래스에 `WebMvcConfigurer` 를 `implements` 해야한다.

<br/>

- `addInterceptors(InterceptorRegistry registry)`
    - 인터셉터를 등록하는 메서드이다.

<br/>

- `addInterceptor(new LoginCheckInterceptor())`
    - `LoginCheckInterceptor` 를 인터셉터로 등록한다.

<br/>

- `order(1)`
    - 인터셉터의 호출 순서이다.
    - 첫 번째로 호출된다.

<br/>

- `addPathPatterns("/**")`
    - 인터셉터를 적용할 URL 패턴을 지정한다.

<br/>

- `excludePathPatterns("/", "/member/add", "/member/login", "/member/logout")`
    - 인터셉터 적용을 제외할 패턴을 지정한다.

<br/>

- 스프링의 URL 경로
    - 서블릿과는 다르게 세밀하게 설정할 수 있다.
    - 대표적인 문법

        ```
        ? 한 개의 문자 일치
        * 경로 안에서 0개 이상의 경로 일치
        ** 경로 끝까지 0개 이상의 경로 일치
        ```

    - [PathPattern 공식 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)

<br/>

### 결과

- **로그인X, 바로 URL(`http://localhost:8080/service`) 접근**

    ![로그인X, 바로 URL](/assets/img/2021-08-15-SPRING_MVC_LoginInterceptor/Untitled%2042.png)

- **로그인 완료**

    ![로그인 완료](/assets/img/2021-08-15-SPRING_MVC_LoginInterceptor/Untitled%2043.png)

<br/><br/>

## 스프링 인터셉터 부가 설명

### `preHandle` , `postHandle` , `afterCompletion` 공동 값

- `preHandle` , `postHandle` , `afterCompletion` 에서 공통으로 사용할 값이 있다면, **`HttpServletRequest객체.addAttribute()` 를 사용해서 담아두어야 한다.**
    - 일반 지역변수(메서드 내부에 선언된 변수)로 해결할 수 없다.
    - 멤버 변수(클래스 내부에 선언된 변수)를 사용하기에도 위험하다. (싱글톤 문제)
    - **왜냐하면, `preHandle` , `postHandle` , `afterCompletion` 의 호출시점이 모두 다르고, 인터셉터 객체가 싱글톤으로 사용되기 때문이다.**
    - **호출시점이 다르더라도, `HttpServletRequest객체` 는 공통적으로 사용한다.**

<br/>

- 예시 코드

    ```java
    @Slf4j
    public class MyInterceptor implements HandlerInterceptor {

      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    		String myValue = "공용으로 사용하고 싶은 값";
    		request.addAttribute("myVar", myValue);
      }

    	/**
    	 * 핸들러 호출 후, 실행되는 메서드
    	 */
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
          request.getAttribute("myVar", myValue);
      }

    	/**
    	 * 항상 실행되는 메서드
    	 */
      @Override
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
          request.getAttribute("myVar", myValue);
      }
    }
    ```

<br/>

### 핸들러 정보

- **`HandlerMethod` 객체**
    - `@Controller` 와 `@RequestMapping` 을 사용하는 핸들러(컨트롤러 메서드)에 대한 모든 정보가 담겨있다.
    - 예시 코드

        ```java
        @Slf4j
        public class MyInterceptor implements HandlerInterceptor {

          @Override
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        		if (handler instanceof HandlerMethod) {
        			HandlerMethod hm = (HandlerMethod) handler; //호출할 컨트롤러 메서드의 모든 정보가 담겨있음
        		}
          }

        }
        ```

- **`ResourceHttpRequestHandler` 객체**
    - `/resources/static` 와 같은 정적 리소스가 호출되는 경우에 사용되는 핸들러(컨트롤러 메서드)에 대한 모든 정보가 담겨있다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*