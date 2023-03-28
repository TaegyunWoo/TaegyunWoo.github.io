---
category: Tech
tags: [Tech]
title: "SpringBoot - Dispatcher Servlet을 활용해 매핑되는 핸들러 구하기"
date:   2022-03-10 16:10:00 
lastmod : 2022-03-10 16:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# Dispatcher Servlet을 활용해 매핑되는 핸들러 구하기
## 개요
- 최근에 스프링부트를 활용한 서버 Application 프로젝트를 진행하던 도중, 아래와 같은 요구사항이 추가되었다.
  - 인터셉터를 통해 404 에러를 처리하라.
  - 그러기 위해선, 서버로 전달된 HTTP 요청 메시지의 URI(Endpoint)에 따라, 어떤 컨트롤러가 매핑되는지 분석한다.
  - 그리고 그 결과를 기반으로 응답 메시지를 다르게 전송한다.
- 단순히 해당 요구사항만 본다면, "컨트롤러 계층이나 서비스 계층에서 처리를 하면 되는 간단한 문제아닌가?" 라고 생각할 수 있다.
- 하지만 이 로직을 **인터셉터 단에서 처리하도록 해야하는 상황**이었다.

> 프로젝트에서 해당 요구사항이 추가된 이유는 "로그인 인터셉터보다 먼저 해당 로직이 동작"해야하기 때문이었다.  
따라서 컨트롤러 단에서 처리할 수 없었다.

> 보다 자세한 내용은 [관련 이슈](https://github.com/Bros4Kkun/Back-end/issues/21)를 참고하자.

- 따라서 관련 사항을 프로젝트에서 구현하기 위해, 공부한 내용을 정리하고자 한다.

- 자세한 설명을 위해, 실제 소스코드를 먼저 보자.
  - 실제로 프로젝트에서 사용된 코드 일부를 그대로 가져왔다.
  - **해당 소스코드에선 인터셉터가 활용되었지만, 핵심 로직을 위주로 참고하자.**

<br/>

## 소스코드
### NoHandlerFoundInterceptor 인터셉터

> 참고로 인터셉터 관련 내용은 [해당 포스팅](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_LoginInterceptor) 을 참고하자.

```java
@RequiredArgsConstructor
@Component
public class NoHandlerFoundInterceptor implements HandlerInterceptor {

  private final DispatcherServlet dispatcherServlet;

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    String uri = request.getRequestURI();
    if (null == getHandler(request)) {
      response.sendError(HttpServletResponse.SC_NOT_FOUND);
      return false;
    }
    return true;
  }

  protected HandlerExecutionChain getHandler(HttpServletRequest request) {
    if (dispatcherServlet.getHandlerMappings() != null) {
      for (HandlerMapping mapping : dispatcherServlet.getHandlerMappings()) {
        try {
          HandlerExecutionChain handlerExecutionChain = mapping.getHandler(request);
          if (!handlerExecutionChain.getHandler().toString().contains("ResourceHttpRequestHandler")) {
            return handlerExecutionChain;
          }
        } catch (Exception e) {
          //ignore
        }
      }
    }

    return null;
  }

}
```

<br/>

## 상세 설명
### `implements HandlerInterceptor`
- `HandlerInterceptor` 는 인터셉터를 구현하기 위한 인터페이스이다.
- 인터셉터 관련 사항은 본 글의 범위를 넘어가니 생략하겠다.

### `private final DispatcherServlet dispatcherServlet`
- 생성자 주입을 통해, `DispatcherServlet` Bean 객체를 주입받았다.
- **해당 Bean 객체가 핵심 역할을 하게 된다. 해당 객체가 제공하는 메서드를 통해 매핑되는 핸들러(컨트롤러)를 찾을 수 있다.**
- 여기서 주입받은 `DispatcherServlet` Bean 객체는 실제로 스프링 구조에서 사용되는 객체이다.
- `DispatcherServlet`(디스패처 서블릿)은 HTTP 요청 메시지를 적절한 핸들러(컨트롤러)에게 전달하는 역할을 수행한다.
- 즉 수문장 역할을 수행한다고 생각하면 된다.
- 이것을 그림으로 표현하면 다음과 같다.
    ![](/assets/img/2022-03-10-Tech_Spring_Find_Mapping_Handler/Untitled.png)

> 디스패처 서블릿을 포함한 스프링 MVC 구조에 대한 내용은 [해당 포스팅](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Structure)을 참고하자.

### `HandlerExecutionChain` 메서드
- 해당 메서드에 핵심 로직이 들어있다. 본격적으로 알아보자.
- `dispatcherServlet.getHandlerMappings()`
  - 해당 메서드를 통해, 핸들러 매핑 정보가 담긴 Collection 객체를 반환받을 수 있다.
- `mapping.getHandler(request)`
  - 매개변수로 `HttpServletRequest` 객체를 전달하므로써, 해당 요청이 어떤 핸들러(컨트롤러)와 매핑되는지 알 수 있다.
  - 만약 사용자(개발자)가 정의(구현)한 핸들러 중, 매핑되는 핸들러가 없다면 `ResourceHttpRequestHandler` 관련 정보를 반환한다.
    - `ResourceHttpRequestHandler` 은 정적 리소스를 응답하는 기본 핸들러이다. 스프링 자체에서 제공하는 핸들러이다.
    - 만약 서버에 들어온 요청이 아무런 핸들러와도 매핑되지 않는다면, 해당 핸들러와 매핑되도록 기본적으로 설정되어 있다.

<br/>

## 정리
- 스프링 구조에서 수문장 역할을 하는 `DispatcherServlet` Bean 객체를 주입받아, 관련 메서드를 사용할 수 있다.
- `DispatcherServlet`가 요청 메시지를 어떤 핸들러에게 보낼지(매핑할지) 결정한다.
- **따라서 해당 객체를 활용하여, 들어온 요청이 어떤 핸들러(컨트롤러)와 매핑되는지 알 수 있다.**
- 핵심 메서드
  - `getHandlerMappings()` : `DispatcherServlet` 의 인스턴스 메서드
  - `getHandler(HttpServletRequest)` : `HandlerMapping` 의 인스턴스 메서드
- 필자는 "인터셉터를 통해 404 에러를 처리"하기 위해, 본 로직을 사용했다.  
본인과 같은 경우가 아니더라도, `DispatcherServlet`에 접근하여 다양한 문제를 해결할 수 있을 것으로 보인다.


<br/><br/>

# Reference

- [https://stackoverflow.com/questions/59387737/spring-boot-spring-security-returns-a-status-401-instead-of-404-for-no-mapping](https://stackoverflow.com/questions/59387737/spring-boot-spring-security-returns-a-status-401-instead-of-404-for-no-mapping)