---
category: Spring-MVC
tags: [스프링, MVC, 예외처리]
title: "[스프링 - MVC] 예외처리 - API 예외처리와 HandlerExceptionResolver"
date:   2021-08-16 19:30:00 
lastmod : 2021-08-16 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    1. [예외처리 - 서블릿 예외 처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)
    2. [예외처리 - 스프링 오류 페이지](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring)
    3. [예외처리 - API 예외처리와 BasicErrorController](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndBasicController)

<br/><br/>

# HandlerExceptionResolver 기초

## 개요

- [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndBasicController)에서 오류 API를 `BasicErrorController` 를 통해 처리하는 방법에 대해 알아보았다.
- 하지만 이 방법은 한계가 분명히 있다.
- `HandlerExceptionResolver` 를 통해, 문제를 해결할 수 있다.

<br/>

### 목표

- 예외가 발생해서 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다. **발생하는 예외에 따라서 다른 상태코드로 처리될 수 있게 한다.**
- 오류 메시지, 형식 등을 API마다 다르게 처리할 수 있게 한다.

<br/>

### HandlerExceptionResolver 란?

- 스프링은 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다.
- 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 `HandlerExceptionResolver` 를 사용하면 된다.

<br/><br/>

## `HandlerExceptionResolver` 원리

### `HandlerExceptionResolver` 적용 전

![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver/Untitled%2014.png)

<br/>

### `HandlerExceptionResolver` 적용 후

![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver/Untitled%2015.png)

1. 컨트롤러에서 예외 발생시, 디스패쳐 서블릿에 예외가 전달된다.
2. 인터셉터의 `postHandle` 은 호출되지 않는다.
3. **디스패쳐 서블릿이 `HandlerExceptionResolver` 를 통해 예외를 해결하고자 한다.**
4. **`HandlerExceptionResolver` 의 예외 처리 시도**
    1. **예외를 처리하고, 디스패쳐 서블릿이 예외 대신 `ModelAndView` 객체를 전달받는다.**
    2. **예외를 처리하고, 디스패쳐 서블릿이 예외나 에러응답을 전달받는다.**

    > **즉, 예외를 무조건 처리하지는 않는다.**

5. 이후 동작
    1. `ModelAndView` 객체를 전달받았다면, 정상적으로 동작한다.
    2. 예외나 에러응답을 전달받았다면, 내부요청을 진행한다.

<br/><br/>

## `HandlerExceptionResolver` 인터페이스

```java
public interface HandlerExceptionResolver {
	ModelAndView resolveException( HttpServletRequest request,
					HttpServletResponse response, 
					Object handler, Exception ex);
}
```

<br/>

### 매개변수 설명

- `Object handler`
    - 핸들러(컨트롤러) 정보
- `Exception ex`
    - 핸들러(컨트롤러)에서 발생한 예외

<br/><br/>

## `HandlerExceptionResolver` 적용

### `HandlerExceptionResolver` 구현: `MyHandlerExceptionResolver` 클래스

```java
package example.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
  @Override
  public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
    
    try {
      if (ex instanceof IllegalArgumentException) {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
				return new ModelAndView();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }

    return null;
  }
}
```

- 위 예시 코드의 목적
    - `IllegalArgumentException` 과 같은 예외는 대부분 사용자가 값을 잘못 넘긴 경우에 발생한다.
    - 해당 예외가 WAS까지 전달되면, 스프링은 500 코드로 응답한다. 하지만, 사용자 잘못이므로 400대 코드로 응답해야한다.
- `IllegalArgumentException` 이 발생하면, `response.sendError(400)` 이 호출되며 HTTP 상태 코드를 400으로 지정하고, 비어있는 `ModelAndView` 객체를 반환한다.

<br/>

### 반환 값에 따른 동작 방식

- **빈 ModelAndView**
    - 뷰를 렌더링하지 않는다.
    - 정상 흐름으로 서블릿이 리턴된다.
    - 즉, 디스패처 서블릿이 정상작동임을 알 수 있게 ModelAndView를 반환해준다.

    > **하지만, 위 예시에선 `sendError()` 를 호출하여, 다시 내부호출이 되기는 한다.**

- **ModelAndView 지정**
    - 뷰를 렌더링한다.
- **null**
    - 다음 `HandlerExceptionResolver` 를 찾아서 실행한다.
    - **만약, 처리할 수 있는 `HandlerExceptionResolver` 가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던진다.**

<br/>

### `MyHandlerExceptionResolver` 등록: `WebConfig` 클래스

```java
import ...

@Configuration
public class WebConfig implements WebMvcConfigurer {

    /**
     * HandlerExceptionResolver 의 구현체 등록
     * (스프링이 기본적으로 등록하는 ExceptionResolver 가 유지되도록 extend 처리)
     */
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }

}
```

<br/><br/><br/>

# HandlerExceptionResolver 활용

## 예외 마무리하기

### 개요

- 예외 발생시 WAS까지 예외가 던져지고, 다시 `/error` 를 내부호출하는 과정은 너무 복잡하다.
- `HandlerExceptionResolver` 에서 직접 정상적인 응답을 한다면, 해결되는 문제이다.

<br/>

### `HandlerExceptionResolver` 에서 정상 응답하기: `MyHandlerExceptionResolver` 클래스 수정

```java
package example.exception;

import ...

public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    //JACKSON 라이브러리
    private ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try {
            if (ex instanceof IllegalArgumentException) {
                //일단 상태 코드는 400으로 설정
                response.setStatus(400);
                
                //요청 헤더 accept 값 확인
                String acceptHeader = request.getHeader("accept");

                //만약 json 형식의 응답을 원한다면
                if (acceptHeader.equals("application/json")) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());
                    
                    //JSON으로 응답하기 위해 문자열로 변환 (Jackson 라이브러리)
                    String result = objectMapper.writeValueAsString(errorResult);
                    
                    //정상 응답하기
                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);

                    return new ModelAndView();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;

    }
}
```

<br/>

### IllegalArgumentException 던지기: `ExceptionController` 클래스

```java
import ...

@Controller
public class ExceptionController {
    @GetMapping("/illegal")
    public void sendIllegalError(HttpServletResponse response) throws IOException {
        throw new IllegalArgumentException("Illegal예외 발생!");
    }
}
```

<br/>

### 결과

- 요청

    ![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver/Untitled%2016.png)

<br/>

- 결과

    ![Untitled](/assets/img/2021-08-16-SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver/Untitled%2017.png)

<br/>

### 정리

- `HandlerExceptionResolver` 를 구현하여, 컨트롤러에서 예외가 발생해도 `HandlerExceptionResovler` 에서 예외를 처리해버린다.
    - `return ModelAndView()` 를 통해 정상흐름 유지
    - `response.getWriter().write(result)` 를 통해 정상응답

    > 위 예시의 경우, `sendError()` 를 사용하지 않았다. 그러므로, 내부요청은 수행되지 않는다.

<br/>

- 따라서 결과적으로 WAS 입장에서는 정상 처리가 되었다.
    - 내부 요청 로직이 수행되지 않아, 깔끔해졌다.

<br/>

- **하지만, 상당히 복잡하다. 다음 게시글에서 보다 간편하게 예외를 처리하는 방법에 대해 설명하겠다.**

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*