---
category: Spring-MVC
tags: [스프링, MVC, 예외처리]
title: "[스프링 - MVC] 예외처리 - 스프링의 HandlerExceptionResolver"
date:   2021-08-17 14:00:00 
lastmod : 2021-08-17 14:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    1. [예외처리 - 서블릿 예외 처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionServlet)
    2. [예외처리 - 스프링 오류 페이지](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionSpring)
    3. [예외처리 - BasicErrorController](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndBasicController)
    4. [예외처리 - API 예외처리와 HandlerExceptionResolver](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ExceptionAPIAndHandlerExceptionResolver)

<br/><br/>

# 스프링의 HandlerExceptionResolver

## 개요

- `HandlerExceptionResolver` 는 API 예외 처리에 주로 사용된다.
- 스프링 부트가 기본적으로 제공하는 `HandlerExceptionResolver` 가 존재한다.
- 즉, 스프링이 `HandlerExceptionResolver` 의 구현체를 제공한다.

<br/><br/>

## 스프링이 제공하는 HandlerExceptionResolver 종류

- **ExceptionHandlerExceptionResolver**
- **ResponseStatusExceptionResolver**
- **DefaultHandlerExceptionResolver**

하나씩 알아보자.

<br/><br/>

## **ExceptionHandlerExceptionResolver**

- 주요 사용되는 기능으로, 가장 마지막에 설명하도록 하겠다.

<br/><br/>

## ResponseStatusExceptionResolver

- 예외에 따라서 HTTP 상태코드를 지정해주는 역할을 한다.
- 예) `@ResponseStatus(value = HttpStatus.NOT_FOUND)`
- HTTP 상태코드를 지정하고, **`sendError(상태코드)` 를 호출한다.**
    - **즉, WAS에서 다시 오류 페이지 ( `/error` )를 내부호출한다.**

<br/>

### 처리하는 예외

- `@ResponseStatus` 가 달려있는 예외
    ```java
    @ResponseStatus(value = "상태코드" , reason = "메시지소스에 작성된 메시지코드")```
- `ResponseStatusException` 예외
    ```java
    throw new ResponseStatusException("상태코드", "메시지소스에 작성된 메시지코드", 예외종류)
    ```

<br/>

### 사용 예시: `@ResponseStatus`

- ResponseStatusExceptionResolver 는 기본적으로 스프링에 적용되어 있기 때문에 따로 구현할 것이 없다.

<br/>

- **`MyException` 클래스**

    ```java
    import ...

    /**
     * 사용자 예외
     */
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    public class MyException extends RuntimeException { }
    ```

    - 해당 예외가 컨트롤러 밖으로 넘어가면 `ResponseStatusExceptionResolver` 예외가 해당 애너테이션을 확인해서 오류 코드를 `BAD_REQUEST` (400)으로 변경하고, 메시지도 담는다.
    - `sendError(400)` 을 호출하여, 내부호출이 진행된다.

<br/>

- **`ExceptionController` 클래스**

    ```java
    import ...

    @Controller
    public class ExceptionController {

      @GetMapping("/myException")
      public void throwMyException() {
        throw new MyException();
      }

    }
    ```

<br/>

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2018.png)

<br/>

- 결과

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2019.png)

<br/>

- **정리**
    - `ResponseStatusExceptionResolver` 는 `@ResponseStatus` 애너테이션을 확인하고, 해당 예외를 처리한다.
    - `ResponseStatusExceptionResolver` 는 `response.sendError()` 를 통해 오류 응답을 한다.
    - 따라서, 디스패처 서블릿이 `sendError()` 를 확인하고 `/error` 를 내부요청하는데, 이때 `BasicErrorController` 가 호출되며 내부호출을 처리한다.

<br/>

### 사용 예시: `ResponseStatusException`

- `@ResponseStatus` 는 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다.
- 이때 `ResponseStatusException` 예외를 사용한다.

<br/>

- **`ExceptionController` 클래스**

    ```java
    import ...

    @Controller
    public class ExceptionController {

    	@GetMapping("/responseException")
    	public void throwResponseStatusException() {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
      }

    }
    ```

<br/>

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2020.png)

<br/>

- 결과

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2021.png)

<br/><br/>

## DefaultHandlerExceptionResolver

- DefaultHandlerExceptionResolver 는 스프링 내부에서 발생하는 스프링 예외를 해결한다.

<br/>

- **예시: DefaultHandlerExceptionResolver가 적용되지 않는다면**
    1. 파라미터 바인딩 시점에 타입이 맞지 않아, `TypeMismatch` 예외 발생
    2. 예외가 WAS까지 전달되어, `/error` 내부호출
    3. **`BasicErrorController` 가 상태코드 500으로 응답**

<br/>

- **예시: DefaultHandlerExceptionResolver가 적용된다면**
    1. 파라미터 바인딩 시점에 타입이 맞지 않아, `TypeMismatch` 예외 발생
    2. 예외를 DefaultHandlerExceptionResolver 가 처리하고, **상태코드 400으로 오류응답 전달**
    3. WAS가 `sendError()` 호출을 확인하고, `/error` 내부호출
    4. **`BasicErrorController` 가 상태코드 400으로 응답**

<br/>

### 사용예시

- DefaultHandlerExceptionResolver 는 기본적으로 스프링에 적용되어 있기 때문에 따로 구현할 것이 없다.

<br/>

- **`ExceptionController` 클래스**

    ```java
    import ...

    @Controller
    public class ExceptionController {

    	@GetMapping("/defaultResolver")
      public String useDefaultHandlerExceptionResolver(@RequestParam Integer data) {
        return "ok";
      }

    }
    ```

<br/>

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2022.png)

<br/>

- 결과

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2023.png)

<br/><br/><br/>

# **ExceptionHandlerExceptionResolver**

## 개요

- HTML 오류 응답을 할 때는 단순히 `BasicErrorController` 를 사용하면 된다.
- 하지만, API 예외 처리를 해결하기 위해서는 `ExceptionHandlerExceptionResolver` 를 사용해야 한다.
    - 왜냐하면, 세밀한 제어가 가능하기 때문이다.
- `@ExceptionHandler` 애너테이션을 통해, 예외처리를 수행한다.

<br/><br/>

## 사용 예시

- 사용 예시를 먼저 살펴보고, 자세한 설명을 하도록 하겠다.

<br/>

### 예외 발생시 API 응답될 객체: `ErrorItem` 클래스

```java
import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor //모든 private 필드에 대해 생성자를 생성해준다.
public class ErrorItem {
    private String StatusCode;
    private String Errormessage;
}
```

<br/>

### 예외 처리 컨트롤러: `ExceptionHandlerController` 클래스

```java
import ...

@RestController
public class ExceptionHandlerController {

    /**
     * 상태코드를 BAD_REQUEST 로 설정한다.
     * IllegalArgumentException 발생시, 호출된다.
     * @param e 컨트롤러에서 발생한 예외 객체를 담는다.
     * @return 예외 전용 객체를 응답 데이터로서 반환한다.
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorItem illegalExHandle(IllegalArgumentException e) {
        return new ErrorItem("BAD", e.getMessage());
    }

    /**
     * MyException 발생시, 호출된다.
     * @param e 컨트롤러에서 발생한 예외 객체를 담는다.
     * @return 예외 전용 객체를 응답 바디에 담고, 상태코드를 BAD_REQUEST 로 설정하여 응답한다.
     */
    @ExceptionHandler(MyException.class)
    public ResponseEntity<ErrorItem> myExHandler(MyException e) {
        ErrorItem errorItem = new ErrorItem("MY-EX", e.getMessage());
        return new ResponseEntity<>(errorItem, HttpStatus.BAD_REQUEST);
    }

    /**
     * 상태코드를 400으로 설정한다.
     * RuntimeException 발생시, 호출된다.
     * @ExceptionHandler 에서 매핑할 예외가 생략되면 파라미터의 타입으로 매핑한다.
     * @param e 컨트롤러에서 발생한 예외 객체를 담는다.
     * @return 예외 전용 객체를 응답 데이터로서 반환한다.
     */
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorItem runtimeExHandle(RuntimeException e) {
        return new ErrorItem("EX", "내부오류");
    }

    @GetMapping("/ex/{exName}")
    public String throwExceptions(@PathVariable String exName) {
        if (exName.equals("illegal")) {
            throw new IllegalArgumentException("IllegalArgument 예외!");
        }
        if (exName.equals("my")) {
            throw new MyException();
        }
        if (exName.equals("runtime")) {
            throw new RuntimeException();
        }

        return "ok";
    }
}
```

<br/>

### 요청결과: `IllegalArgumentException` 발생시

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2024.png)

<br/>

- 응답

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2025.png)

<br/>

- 실행 흐름
    1. 컨트롤러를 호출한 결과 `IllegalArgumentException` 예외가 컨트롤러 밖으로 던져진다.
    2. 예외가 발생했으므로 `HandlerExceptionResolver` 가 작동한다.
        - 그리고 그 중 가장 우선순위가 높은 `ExceptionHandlerExceptionResolver` 가 실행된다.
    3. `ExceptionHandlerExceptionResolver` 는 해당 컨트롤러(예외가 발생한 컨트롤러)에 `IllegalArgumentException` 을 처리할 수 있는 `@ExceptionHandler` 가 있는지 확인한다.
    4. `illegalExHandler()` 를 실행한다. `@RestController` 이므로 `illegalExHandler()` 에도 `@ResponseBody` 가 적용된다. 따라서 HTTP 컨버터가 사용되고, 응답이 JSON으로 반환된다.
    5. `@ResponseStatus(HttpStatus.BAD_REQUEST)` 를 지정했으므로, HTTP 상태 코드 400으로 응답한다.

<br/>

### 요청결과: `MyException` 발생시

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2026.png)

<br/>

- 응답

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2027.png)

<br/>

- 실행 흐름
    1. 컨트롤러를 호출한 결과 `MyException` 예외가 컨트롤러 밖으로 던져진다.
    2. 예외가 발생했으므로 `HandlerExceptionResolver` 가 작동한다.
        - 그리고 그 중 가장 우선순위가 높은 `ExceptionHandlerExceptionResolver` 가 실행된다.
    3. `ExceptionHandlerExceptionResolver` 는 해당 컨트롤러(예외가 발생한 컨트롤러)에 `MyException` 을 처리할 수 있는 `@ExceptionHandler` 가 있는지 확인한다.
    4. `myExHandler()` 를 실행한다. `@RestController` 이므로 `myExHandler()` 에도 `@ResponseBody` 가 적용된다. HTTP 컨버터가 사용되고, 응답이 HTTP 응답 바디에 직접 입력된다.( `ResponseEntity<>` 를 반환하므로 )
    5. `@ResponseStatus(HttpStatus.BAD_REQUEST)` 를 지정했으므로, HTTP 상태 코드 400으로 응답한다.

<br/>

### 요청결과: `RuntimeException` 발생시

- 요청

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2028.png)

<br/>

- 응답

    ![Untitled](/assets/img/2021-08-17-SPRING_MVC_ExceptionSpringAPI/Untitled%2029.png)

<br/>

### 상세설명

- `@ResponseStatus`
    - 만약 해당 애너테이션 없이, `@ExceptionHandler` 만 사용한다면 **상태코드가 200으로 정상응답될 것이다.**
        - 왜냐하면, `ExceptionHandlerController` 가 예외를 처리하여, 정상적으로 `ErrorItem` 객체를 JSON으로 반환했다. 따라서 WAS 입장에서는 정상적으로 모든 과정이 수행되었다고 인식할 수 밖에 없다.
        - 따라서, `@ResponseStatus` 를 통해 상태코드를 알려주어야 한다. 참고로, 응답 자체는 정상 응답이다. 단지, 상태코드가 200이 아닐 뿐이다.

<br/>

- `@ExceptionHandler`
    - `@ExceptionHandler(매핑될 예외)`
        - 속성으로 작성한 예외가 발생했을 때, 해당 메서드가 매핑되어 호출된다.
    - `@ExceptionHandler`
        - 속성으로 매핑할 예외가 생략되었다면, 해당 메서드의 **매개변수 타입으로 매핑**한다.

<br/>

- `return ErrorItem(..)`
    - 해당 객체를 응답데이터로서 반환한다.
    - 이때, JACKSON 라이브러리의 HTTPMessageConvertor가 응답 메시지 바디에 JSON 형태로 넣어준다.

        > HTTPMessageConvertor 는 [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_HTTPMessageConverter)을 참고하자.

    - WAS는 여기서 반환되어 변환된 JSON 데이터를 정상적으로 응답한다.

<br/><br/>

## `@ExceptionHandler` 상세설명

### 예외 처리 방법

- `@ExceptionHandler` 애너테이션으로 처리하고 싶은 예외를 지정해주면 된다.
- 해당 컨트롤러에서 예외가 발생하면 이 메서드가 호출된다.
- **지정한 예외와 자식 클래스까지 잡을 수 있다.**

    ```java
    @ExceptionHandler(부모클래스.class)
    public String 부모예외처리() { .. }
    ```

    - 자식클래스(예외) 발생시 ⇒ 부모예외처리() 호출

<br/>

### 우선순위

```java
@ExceptionHandler(부모클래스.class)
public String 부모예외처리() { .. }

@ExceptionHandler(자식클래스.class)
public String 자식예외처리() { .. }
```

위와 같이, 예외 처리 메서드를 작성했다면 어떻게 선택될까?

- 자식클래스(예외) 발생시 ⇒ 자식예외처리() 호출
- 부모클래스(예외) 발생시 ⇒ 부모예외처리() 호출

<br/>

### 다양한 예외 처리

```java
@ExceptionHandler(예외A.class, 예외B.class)
public String ex(Exception e) { .. }
```

- 위와 같이, 여러 예외를 한번에 처리할 수 있다.

<br/>

### 예외 생략

```java
@ExceptionHandler
public String ex(MyException e) { .. }
```

- 위와 같이, 예외를 생략하면 매개변수의 타입(`MyException`)으로 매핑한다.

<br/>

### 파라미터와 응답

`@ExceptionHandler` 가 적용된 메서드로 받을 수 있는 매개변수와 반환 가능한 타입은 아래 링크를 참고하자.

[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args)

<br/>

### HTML 오류 화면 응답

```java
@ExceptionHandler
public ModelAndView ex(MyException e) { 
	return new ModelAndView("error");
 }
```

- 위 코드처럼, `ModelAndView` 를 반환하여 HTML 오류 화면으로 응답할 수도 있다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*