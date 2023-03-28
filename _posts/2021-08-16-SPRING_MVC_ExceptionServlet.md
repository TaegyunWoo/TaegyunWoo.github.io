---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 예외처리 - 서블릿 예외 처리"
date:   2021-08-16 16:30:00 
lastmod : 2021-08-16 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 서블릿 예외 처리

## 개요

- 스프링의 예외처리를 설명하기 전, 순수한 서블릿이 예외를 어떻게 처리하는지 알아보자.
- 먼저 새 프로젝트를 생성하자.

    > [프로젝트 다운로드](https://github.com/TaegyunWoo/TaegyunWoo.github.io/blob/master/ExampleProjects/exception.zip)

<br/>

### 서블릿이 처리하는 예외 종류

- 일반 예외 (Exception)
- response.sendError(HTTP상태코드, 오류메시지)

### 예외처리 설명을 위한 설정: `application.properties`

```
server.error.whitelabel.enabled=false
```

> 스프링 부트가 기본적으로 제공하는 예외 페이지(WhiteLabel)를 잠시 꺼둬야한다.

<br/><br/>

## 웹 애플리케이션의 예외 흐름

### 예외(Exception) 흐름

![예외 흐름](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled.png)

- 웹 애플리케이션은 사용자 요청별로 쓰레드가 할당되고, 서블릿 컨테이너 안에서 실행된다.
- 웹 애플리케이션이 예외를 잡지 못하고 서블릿 밖으로 예외가 전달되면, **결국 WAS(톰캣)까지 예외가 전달되고 오류 페이지를 보여준다.**

<br/>

### response.sendError() 흐름

![sendError 흐름](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%201.png)

- 컨트롤러에서 예외가 발생하지 않고 오직 `sendError()` 를 호출했을 때, **인터셉터와 서블릿, 필터, WAS에 예외가 전파되지 않는다.**
    - 왜냐하면, 컨트롤러에서 예외가 발생한 적이 없고 단순히 `sendError()` 를 호출했기 때문이다.
- WAS까지 전달받은 예외가 없지만, **`sendError()` 가 호출된 기록을 WAS가 확인하여 오류 페이지를 보여준다.**

<br/><br/>

## 예외 처리 예시

### 예외 발생 코드: `ExceptionController` 클래스

```java
import ...

@Controller
public class ExceptionController {

  @GetMapping("/exception")
  public void throwException() {
      throw new RuntimeException("사용자 예외 발생!");
  }

}
```

<br/>

### 호출 결과

- 호출: `http://localhost:8080/exception`

![호출 결과](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%202.png)

- **컨트롤러에서 발생한 예외가 처리되지 못하고 WAS까지 전파되어, 클라이언트에게 에러페이지를 보여준다.**

<br/><br/>

## sendError() 처리 예시

### sendError() 코드: `ExceptionController` 클래스

```java
import ...

@Controller
public class ExceptionController {

  @GetMapping("/exception")
  public String throwException() {
      throw new RuntimeException("사용자 예외 발생!");
  }

	@GetMapping("/sendError")
  public void sendErrorToWAS(HttpServletResponse response) throws IOException {
      response.sendError(500, "sendError 호출!");
  }

}
```

<br/>

### 호출 결과

- 호출: `http://localhost:8080/sendError`

<br/>

![호출 결과](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%203.png)

<br/>

- **컨트롤러에서 호출한 `sendError()`가 처리되지 못하고 WAS가 호출기록을 확인하여, 클라이언트에게 에러페이지를 보여준다.**

<br/><br/>

## 오류 화면 제공

- 서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 고객 친화적이지 않다.
- 순수 서블릿: `web.xml` 파일을 통해 오류 페이지 등록
- 스프링부트를 통한 서블릿: `implements WebServerFactoryCustomizer<ConfigurableWebServerFactory>`

<br/>

### 오류 화면 등록: `WebServerCustomizer` 클래스

```java
import ...

@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
        factory.addErrorPages(errorPage404, errorPage500);
    }
    
}
```

- `ErrorPage(설정할_오류_종류, 내부요청할_URL)`

<br/>

### 재요청 처리 컨트롤러: `ErrorPageController` 클래스

```java
import ...

@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        return "error-page/500";
    }

}
```

> 재요청에 대해선 뒤에 자세히 설명한다. 일단 코드에 집중하자.

<br/>

### 오류처리 뷰 템플릿: `404.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>404 오류 페이지!</h1>
</body>
</html>
```

<br/>

### 오류처리 뷰 템플릿: `500.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>500 오류 페이지!</h1>
</body>
</html>
```

<br/>

### 결과 1

- 요청URL

    `http://localhost:8080/존재하지않는페이지`

- 결과

    ![결과](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%204.png)

<br/>

### 결과 2

- 요청URL

    `http://localhost:8080/sendError`

- 결과

    ![결과](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%205.png)

<br/><br/>

## 오류 페이지 작동 원리

- **`Exception` (예외)가 발생하여 서블릿 밖으로 전달되는 경우**
- **`response.sendError()` 가 호출되는 경우**

서블릿은 위 두가지 경우에 설정된 오류 페이지를 찾는다.

<br/>

### 예외 발생 흐름과 내부요청

![예외 발생 흐름과 내부요청](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%206.png)

1. WAS는 **해당 예외를 처리하는 오류 페이지 정보를 확인**한다.
    - `new ErrorPage(오류종류, 내부요청할_URL)`
    - `new ErrorPage(RuntimeException.class, "/error-page/500")` 등
2. 해당 오류 페이지 정보 중 **"내부요청할 URL 정보"를 참고하여 재요청한다.**

<br/>

### `sendError()` 흐름과 내부요청

![sendError()](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%207.png)

1. WAS는 **해당 예외를 처리하는 오류 페이지 정보를 확인**한다.
    - `new ErrorPage(오류종류, 내부요청할_URL)`
    - `new ErrorPage(RuntimeException.class, "/error-page/500")` 등
2. 해당 오류 페이지 정보 중 **"내부요청할 URL 정보"를 참고하여 재요청한다.**

<br/>

### Point!

- **웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어났는지 전혀 모른다.**
- **오직 서버 내부에서 오류 페이지를 찾기 위해 추가적인 호출을 한다.**

<br/><br/>

## 예외 발생시, 서버가 담아주는 예외 정보

- 예외가 발생하면, WAS가 내부적으로 재요청을 수행한다.
- **이때, `HttpServletRequest` 객체에 오류 정보를 담아준다.**

<br/>

### 오류 정보 종류

- `javax.servlet.error.exception`
    - 예외
- `javax.servlet.error.exception_type`
    - 예외 타입
- `javax.servlet.error.message`
    - 오류 메시지
- `javax.servlet.error.request_uri`
    - 클라이언트 요청 URI
- `javax.servlet.error.servlet_name`
    - 오류가 발생한 서블릿 이름
- `javax.servlet.error.status_code`
    - HTTP 상태 코드

<br/>

### 오류 정보 사용 예시: `ErrorPageController` 클래스

```java
import ...

@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
			request.getAttribute("javax.servlet.error.status_code");
    }

}
```

<br/><br/>

## 필터를 활용한 예외 처리

- 서버 내부에서 오류 페이지를 호출할 때마다, 필터와 인터셉터가 다시 호출된다. 이것은 매우 비효율적이다.
- `DispatcherType` 이라는 추가 정보를 통해, 이러한 문제를 해결할 수 있다.
- `DispatcherType` 은 어떤 종류의 요청인지 알려준다.

<br/>

### `DispatcherType` 의 종류

- `REQUEST`
    - 클라이언트 요청
- `ERROR`
    - 오류 요청
- `FORWARD`
    - 서블릿에서 다른 서블릿이나 JSP를 호출할 때
- `INCLUDE`
    - 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
- `ASYNC`
    - 서블릿 비동기 호출

<br/>

### 필터와 `DispatcherType` 활용

- Log를 찍는 필터에 `DispatcherType` 을 적용하여, **내부호출에는 필터가 적용되지 않도록 구현한다. (목표: 오류시에는 로그를 찍지 않도록 구현)**

    > 원래 오류시에 로깅을 통해 기록을 남기는 것이 좋지만, 예시를 위해 이와 같이 진행한다.

<br/>

- **필터 구현: `LogFilter` 클래스**

    ```java
    import ...

    @Slf4j
    public class LogFilter implements Filter {
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            try {
                log.info("로그 필터 doFilter 호출");
                chain.doFilter(request, response);
            } catch (Exception e) {
                throw e;
            }

        }

        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
            log.info("로그 필터 init 호출");
        }

        @Override
        public void destroy() {
            log.info("로그 필터 destroy 호출");
        }
    }
    ```

<br/>

- **필터 등록: `WebConfig` 클래스**
    - `DispatcherType` 을 활용하여, 필터를 적용하지 않을 요청 구분

    ```java
    import ...

    @Configuration
    public class WebConfig {

        @Bean
        public FilterRegistrationBean logFilter() {
            FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
            filterFilterRegistrationBean.setFilter(new LogFilter());
            filterFilterRegistrationBean.setOrder(1);
            filterFilterRegistrationBean.addUrlPatterns("/*");

            //DispatcherType 이 REQUEST인 경우에만 필터 호출
            filterFilterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST);
            
            /*
            filterFilterRegistrationBean.setDispatcherTypes(DispatcherType.ERROR);
            위 코드작성시, ERROR 일때만 (내부호출일때) 필터가 적용된다.
             */

            return filterFilterRegistrationBean;
        }
    }
    ```

    - `setDispatcherTypes(DispatcherType.REQUEST)`
        - `DispatcherType` 이 REQUEST인 요청에 대해서만 필터가 동작한다.

<br/><br/>

## 전체 흐름 정리

### `/hello` 정상 요청

![/hello](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%208.png)

<br/>

### `/exception` 오류 요청

- 필터는 `DispatchType` 으로 중복 호출 제거

> 인터셉터의 경우, `excludePathPatterns()` 메서드를 통해 호출이 안되도록 설정할 수도 있다.

![오류 요청](/assets/img/2021-08-16-SPRING_MVC_ExceptionServlet/Untitled%209.png)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*