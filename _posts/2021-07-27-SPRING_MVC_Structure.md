---
category: Spring-MVC
tags: [스프링, MVC, MVC구조]
title: "[스프링 - MVC] MVC 구조에 대한 이해"
date:   2021-07-27 17:30:00 
lastmod : 2021-07-27 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# SpringMVC 구조

![SpringMVC](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled.png)

<br><br>

## Dispatcher Servlet

- 스프링 MVC는 **프론트 컨트롤러 패턴으로 구현** 되어 있다.
- 스프링 MVC의 **프론트 컨트롤러는 디스패쳐 서블릿 (Dispatcher Servlet)** 이다.

> 프론트 컨트롤러 패턴?  
간단히 이야기하자면, 여러가지 컨트롤러를 클라이언트(고객)이 직접 호출하지 않고, 1개의 대표 컨트롤러를 통해 호출하는 구조이다.  
즉, 대표 컨트롤러가 "실제 비즈니스 로직을 호출하거나 수행하는 컨트롤러"를 호출하고, 클라이언트는 오직 대표 컨트롤러만을 통해 서비스를 받게 되는 것이다. 여기서 대표 컨트롤러가 바로 **프론트 컨트롤러** 이다.

<br>

### DispatcherServlet 서블릿 등록

- DispacherServlet은 부모 클래스에서 `HttpServlet` 을 상속 받아서 사용하고, 서블릿으로 동작한다.

- 스프링 부트는 `DispatcherServlet` 을 서블릿으로 자동으로 등록하면서 **모든 경로 (** `urlPatterns="/"` **)** 에 대해서 매핑한다.

<br>

### 요청 흐름

- 서블릿이 호출되면 `HttpServlet` 이 제공하는 `service()` 가 호출된다.

- 스프링 MVC는 `DispatcherServlet` 의 부모인 `FrameworkServlet` 에서 `service()` 를 오버라이드 해두었다.
- `FrameworkServlet.service()` 를 시작으로 여러 메서드가 호출되면서 `DispatcherServlet.doDispatch()` 가 호출된다.
- `DispatcherServlet.doDispatch()` 기능
    1. 핸들러 조회
    2. 핸들러 어댑터 조회 - 해당 핸들러를 처리할 수 있는 어댑터
    3. 핸들러 어댑터 실행
    4. 핸들러 어댑터를 통해 핸들러 실행
    5. `ModelAndView` 반환
    6. 뷰 리졸버를 통해서 찾기
    7. `View` 반환
    8. 뷰 렌더링

> 핸들러는 컨트롤러와 유사한 의미이다.  
(핸들러의 의미가 좀 더 넓긴하다.)

<br><br>

## SpringMVC 구조와 동작 순서

위의 그림을 다시 보자.

![SpringMVC](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled.png)

<br>

### 핸들러

핸들러는 컨트롤러와 유사한 개념으로, 컨트롤러보다는 좀더 포괄적인 개념이다. 본 게시글에서 핸들러는 컨트롤러와 동일하다고 생각해도 문제없다.

<br>

### 핸들러 어댑터

핸들러 어댑터는 핸들러를 일정한 (표준적인) 형태로 사용할 수 있게 해주는 것이다. 만약 핸들러의 return 타입이 String 이라면, `dispatcherServlet` 이 받을 수 있는 타입인 `ModelAndView` 로 변환시켜주는 기능을 할 수 있다.

<br>

### 동작 순서

1. **핸들러 조회**
    - 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. **핸들러 어댑터 조회**
    - 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. **핸들러 어댑터 실행**
    - 핸들러 어댑터를 실행한다.
4. **핸들러 실행**
    - 핸들러 어댑터가 실제 핸들러를 실행한다.
5. `ModelAndView` **반환**
    - 핸들러 어댑터는 핸들러가 반환하는 정보를 `ModelAndView` 로 변환해서 반환한다.
6. **viewResolver 호출**
    - 뷰 리졸버를 찾고 실행한다.
7. **View 반환**
    - 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
8. **뷰 렌더링**
    - 뷰를 통해서 뷰를 렌더링한다.

<br><br>

## 핸들러 매핑과 핸들러 어댑터

핸들러 매핑과 핸들러 어댑터가 어떤 것들이 어떻게 사용되는지 예시 코드를 통해 설명하도록 하겠다.

### 초기 스프링 컨트롤러

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {

	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		System.out.println("OldController.handleRequest");
		return null; //뷰 사용 X
	}

}
```

- `@Component`
    - 이 컨트롤러는 `/springmvc/old-controller` 라는 이름의 스프링 빈으로 등록되었다.
- 빈의 이름으로 URL을 매핑할 것이다.

그렇다면, 이 컨트롤러는 스프링이 어떻게 호출하는 것일까? 지금부터 설명하도록 하겠다.

<br>

### 컨트롤러 호출 방식

위 컨트롤러 ( `OldController` ) 가 호출되려면 다음 2가지가 필요하다.

- **HandlerMapping: 핸들러 매핑**
    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
- **HandlerAdapter: 핸들러 어댑터**
    - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.

스프링은 이미 필요한 **핸들러 매핑과 핸들러 어댑터를 대부분 구현** 해두었다. 따라서, 개발자는 `OldController` 와 같은 핸들러를 만들기만 하면 된다. 즉, 스프링이 자동으로 '개발자가 만든 컨트롤러'를 매핑시키고, 적절한 어댑터를 사용한다.

그렇다면, 스프링이 핸들러 매핑을 하는 기준과 핸들러 어댑터를 찾는 기준은 어떤 것일까. 그것은 아래와 같다.

- **HandlerMapping**

    ![Handler Mapping](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled%201.png)

    - 위 예시 ( `OldController` )는 스프링 빈의 이름으로 핸들러를 찾아야 하기 때문에, `BeanNameUrlHandlerMapping` 이 실행에 성공하게 된다.
- **HandlerAdapter**

    ![Handler Adapter](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled%202.png)

    - 위 예시 ( `OldController` )는 `SimpleControllerHandlerAdapter` 가 적용된다. 왜냐하면, `Controller` 인터페이스를 `implements` 했기 때문이다.

> 가장 앞에 위치한 숫자는 우선순위를 나타낸다.

<br>

핸들러 매핑도, 핸들러 어댑터도 모두 순서대로 찾고 만약 없으면 다음 순서로 넘어간다. 참고로 우선순위가 가장 높은 `RequestMappingHandlerMapping` 과 `RequestMappingHandlerAdapter` 가 실무에서 가장 많이 사용된다. **즉,** `@RequestMapping` **애너테이션을 이용한 핸들러 매핑과 어댑터를 주로 사용한다.**

> `@RequestMapping` 은 다음 글에서 자세히 다룬다.

<br>

- 동작 순서
    1. **핸들러 매핑으로 핸들러 조회**
        1. `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.
        2. `OldController` 예시의 경우, `SimpleControllerHandlerAdapter` 가 실행에 성공한다.
    2. **핸들러 어댑터 조회**
        1. `HandlerAdapter` 의 `supports()` 메서드를 순서대로 호출한다.
        2. `OldController` 예시의 경우, `SimpleControllerHandlerAdapter` 가 대상이 된다.
    3. **핸들러 어댑터 실행**
        1. 디스패처서블릿이 조회한 `HttpRequestHandlerAdapter` 를 실행하면서, 핸들러 정보도 함께 넘겨준다.
        2. `HttpRequestHandlerAdapter` 는 핸들러인 `MyHttpRequestHandler`를 내부에서 실행하고, 그 결과를 반환한다.

<br><br>

## 뷰 리졸버

이번에는 뷰 리졸버에 대해 알아보자.

### 뷰 리졸버?
뷰 리졸버는 컨트롤러(핸들러)에게 전달받은 **뷰의 논리적 이름을 물리적 경로로 변환** 시켜준다.  

예를 들어, 컨트롤러가 호출하고자하는 뷰의 물리적 경로가 `/dir1/dir2/myView.jsp`이고, 컨트롤러가 반환한 뷰의 논리적 이름이 `myView` 이라고 해보자. 이때 뷰 리졸버는 `myView` 라는 논리적 이름을 정확한 경로인 `/dir1/dir2/myView.jsp` 으로 변환해준다. 따라서, 프론트 컨트롤러 ( `DispatcherServlet` )가 해당 뷰를 렌더링할 수 있게 된다.

<br>

### 초기 스프링 컨트롤러: View 조회할 수 있도록 변경

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
	
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		System.out.println("OldController.handleRequest");
		return new ModelAndView("new-form"); //ModelAndView 반환을 통해 뷰 호출하도록 유도
	}

}
```

- 해당 코드를 작성한 뒤, 실행해보면 컨트롤러는 정상적으로 호출되지만, Whitelabel Error Page 오류가 발생한다.
- 왜냐하면, 뷰 리졸버가 `new-form` 이라는 논리적 주소를 어떤 물리적 주소로 어떻게 바꿀지 모르기 때문이다.

<br>

### application.properties 파일: 뷰 리졸버에게 정보 전달

```java
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

- application.properties 파일에 위 내용을 추가한다.
- 스프링 부트가 해당 정보를 사용해서 뷰 리졸버( `InternalResourceViewResolver` )를 자동으로 등록한다.
- 즉, 뷰 리졸버에게 전달할 정보(논리적 주소를 어떻게 물리적 주소로 변환할 것인가)를 application.properties 파일에 작성하면, 스프링이 알아서 뷰 리졸버를 사용한다. (뷰 리졸버가 자체적으로 구현되어 있다.)

<br>

### 뷰 리졸버 동작 방식

![MVC](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled.png)

- 뷰 리졸버

    ![뷰 리졸버](/assets/img/2021-07-27-SPRING_MVC_Structure/Untitled%203.png)

- 동작 순서
    1. **핸들러 어댑터 호출**
        1. 핸들러 어댑터를 통해 `new-form` 이라는 논리 뷰 이름을 획득한다.
    2. **ViewResolver 호출**
        1. `new-form` 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
        2. `BeanNameViewResolver` 는 `new-form` 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없다.
        3. 따라서, `InternalResourceViewResolver` 가 호출된다.
    3. **InternalResourceViewResolver**
        1. 이 뷰 리졸버는 `InternalResourceView` 를 반환한다.
    4. **뷰 - InternalResourceView**
        1. `InternalResourceView` 는 JSP처럼 포워드 `forward()` 를 호출해서 처리할 수 있는 경우에 사용한다.
    5. **view.render()**
        1. `view.render()` 가 호출되고 `InternalResourceView` 는 `forward()` 를 사용해서 JSP를 실행한다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*