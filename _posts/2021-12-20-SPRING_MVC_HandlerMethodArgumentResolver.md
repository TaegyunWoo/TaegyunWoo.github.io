---
category: Spring-MVC
tags: [스프링, MVC]
title: "[스프링 - MVC] HandlerMethodArgumentResolver"
date:   2021-12-20 14:00:00 
lastmod : 2021-12-20 14:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# HandlerMethodArgumentResolver

## 개요

### HandlerMethodArgumentResolver란?

- 컨트롤러 메서드에서 특정 조건에 맞는 파라미터가 있을 때, 원하는 값을 바인딩해주는 인터페이스이다.
- `@RequestBody` , `@PathVariable` 등의 애너테이션이 HandlerMethodArgumentResolver를 사용한 것이다.
- 따라서, 스프링에서 제공되지 않는 데이터를 바인딩받고 싶다면, HandlerMethodArgumentResolver 인터페이스를 직접 구현하고, Argument Resolver로 등록하면 된다.

<br/>

### HandlerMethodArgumentResolver 예시

- 컨트롤러 클래스
    
    ```java
    @Controller
    public class MyController {
    	@GetMapping("/api/my/{aaa}")
    	public String controllerMethod(@PathVariable String aaa) {
    		//...
    	}
    }
    ```
    
    - 이때, URL을 분석해서 aaa에 해당하는 값을 `String aaa` 에 바인딩해준다.
    - 이것은 HandlerMethodArgumentResolver의 일종이라고 한다.

<br/><br/>

## 직접 HandlerMethodArgumentResolver 구현하기

예시를 통해, 알아보자.

### Person 객체

```java
@Getter
@Setter
@AllArgsConstructor
public class Person {
	private String name;
	private Integer age;
	private String gender;
}
```

<br/>

### 애너테이션

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomAnno {

}
```

<br/>

### ArgumentResolver

```java
@Component
public class CustomArgumentResolver implements HandlerMethodArgumentResolver {

	//본 Argument Resolver 적용 조건
  @Override
  public boolean supportsParameter(MethodParameter parameter) {
    boolean isCorrectParameterType = parameter.getParameterType().equals(Person.class);
    boolean isCorrectAnnotation = parameter.hasParameterAnnotation(CustomAnno.class);

		//@CustomAnno 애너테이션이 붙어있고 Person 타입일 때, 본 resolver를 적용한다.
    return isCorrectParameterType && isCorrectAnnotation;
  }

	//실제로 Resolver가 동작하는 메서드
  @Override
  public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
    Person person = new Person("MyName", 30, "Man");
		return person;
  }
}
```

- `supportsParameter()` 메서드
    - 해당 Argument Resolver를 적용할 수 있는지 판단하는 메서드이다.
    - return 값이 true이면 적용하고, false이면 적용하지 않는다.
    - `parameter.getParameterType().equals(Person.class)`
        - 컨트롤러 메서드의 매개변수가 Person 타입인지 확인한다.
    - `parameter.hasParameterAnnotation(CustomAnno.class)`
        - 컨트롤러 메서드의 매개변수에 `@CustomAnno` 애너테이션이 붙어있는지 확인한다.

<br/>

- `resolveArgument()` 메서드
    - 바인딩 시, 실행되는 메서드이다.
    - 실제 바인딩할 객체를 리턴한다.
    - 매개변수 설명
        - `MethodParameter parameter`
            - 컨트롤러 메서드의 매개변수 중, 바인딩될 매개변수 관련
        - `ModelAndViewContainer mavContainer`
            - 모델과 뷰 관련 매개변수
            - 호출된 컨트롤러 메서드에 전달될 `Model` 이나 `View` 객체에 접근할 수 있다.
        - `NativeWebRequest webRequest`
            - 들어온 요청 관련 매개변수
            
            ```java
            webRequest.getNativeRequest(HttpServletRequest.class);
            ```
            
            - **위 코드를 통해, 호출된 컨트롤러 메서드에 전달될 `HttpServletRequest` 객체에 접근할 수도 있다.**
        - `WebDataBinderFactory webDataBinderFactory`
            - `WebDataBinder` 객체 생성 팩토리

<br/>

### Argument Resolver 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

	@Autowired
	private CustomArgumentResolver resolver;

	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {

		argumentResolvers.add(resolver);

	}

}
```

<br/>

### 컨트롤러

```java
@RestController
public class MyController {

	@GetMapping("/api/my")
	public Person myMethod(@CustomAnno Person person) {
		//...
	}

}
```

- 위에서 작성한 `resolveArgument()` 메서드의 반환값이 `myMethod` 메서드의 매개변수 `person` 에 바인딩된다.