---
category: Spring-Security
tags: [Spring, SpringSecurity]
title: "[Spring-Security] Spring Security의 전체 구조"
date:   2023-03-24 21:15:00 
lastmod : 2023-03-24 21:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

Spring Security의 각 기능에 대해 설명하기 전에, 먼저 전체적인 구조를 알아봅시다!

이번 포스팅에서는 Spring Security의 전체 구조에 대해 설명합니다.

<br/><br/>

## 서블릿 필터

Spring Security는 Java EE 스펙 중 일부인 **서블릿 필터를 기반으로 동작**합니다.

그렇다면 서블릿 필터는 무엇일까요?

<br/>

### HTTP 요청 처리

아래 그림은 하나의 HTTP 요청을 처리하는 전형적인 구조입니다.

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled.png)

클라이언트가 특정 URL로 HTTP 요청을 하게 되면, 먼저 `서블릿 컨테이너` 가 `Filter Chain` 을 생성합니다.

이 `Filter Chain` 은 여러 필터와 하나의 서블릿으로 구성됩니다. 그리고 이 `Filter Chain` 을 통해 `HttpServletRequest` 를 처리하게 됩니다.

**Spring MVC에서는 `Dispatcher Servlet` 이 `Filter Chain` 의 서블릿 역할을 하게 됩니다.**

<br/>

### 필터의 역할

필터는 서블릿 호출 전후로 동작하여, `HttpServletRequest` 나 `HttpServletResponse` 를 수정하거나, 흐름을 제어하는 역할을 수행합니다.

여기서 기억해야 하는 것은, **하나의 필터는 다른 필터를 호출하는 형태로 `Filter Chain` 이 구성된다는 것**입니다.

> 이것이 가장 중요합니다!
> 

한번 필터 코드가 어떻게 구성되는지 살펴볼까요?

```java
//Filter 클래스에서 필터링 작업을 수행하는 메서드

public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	//다른 필터를 호출하기 전(or 서블릿 호출 전)에 처리해야 하는 로직이 담길 부분

	chain.doFilter(request, response); //필터 체이닝

	//서블릿까지 호출된 후, 다른 필터 호출 이후에 처리해야 하는 로직이 담길 부분
}
```

위 코드를 보면, 이해하기 수월할 겁니다.

이런 식으로 필터들은 서로 체이닝이 되어 있기 때문에, **필터들간의 순서도 매우 중요**합니다!

여기까지 서블릿 필터에 대해 알아봤습니다.

이제 본격적으로 Spring Security 구조에 대해 설명하겠습니다.

<br/><br/>

## `DelegatingFilterProxy`

필터는 Java EE의 스펙 중 Servlet에 포함된 기술입니다.

그리고 Spring에서도 Bean 객체를 필터로 사용할 수 있도록 지원합니다.

여기서 이상한 점이 느껴지지 않나요?

- **필터는 Java EE의 기술이다. → 즉, 서블릿 컨테이너에서 관리된다.**
- **Spring에서 필터를 만들고, Bean 객체로 등록할 수 있다. → 즉, Spring 컨테이너에서 관리된다.**

Spring은 서블릿 컨테이너 안에서 동작합니다.

Spring 컨테이너는 서블릿 컨테이너 내부에 존재하죠.

Spring 컨테이너와 서블릿 컨테이너는 분명히 서로 다른 환경입니다.

그렇기 때문에, 서블릿 컨테이너는 자체 표준을 사용(`Filter` 인터페이스를 구현하는 방식)해야만 필터로 동작합니다. **따라서 Spring의 Bean 객체는 인식하지 못합니다.**

그러면 어떻게 Spring에서 필터를 다룰 수 있도록 하는 것일까요?

**다시 말해서, 어떻게 필터가 Spring Context를 참조할 수 있을까요?**

**그 해답이 바로, `DelegatingFilterProxy` 입니다.**

<br/>

### `DelegatingFilterProxy` 가 뭐죠?

`DelegatingFilterProxy` 는 **Spring에서 제공하는 필터 구현체**입니다.

> **Spring Security가 아닌 Spring에서 지원하는 구현체입니다!**
> 

이것이 **서블릿 컨테이너와 Spring Context 사이의 중간 다리 역할**을 합니다.

그리고 **이 `DelegatingFilterProxy` 는 단순히 중간 다리 역할을 할 뿐이고, 실제 필터링 로직은 `Filter` 인터페이스를 구현한 Bean 객체에 위임됩니다.**

> 그래서 `Delegating` 이라는 이름이 붙었습니다. (대리자라는 뜻)
> 

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled%201.png)

`DelegatingFilterProxy` 는 Spring Context에 등록된 `필터 Bean 객체` 를 찾고, 호출합니다.

`DelegatingFilterProxy` 의 코드를 살펴볼까요?

```java
//DelegatingFilterProxy 클래스의 일부 코드

public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	//Spring Bean으로 등록된 필터 객체 Lazy 조회
	Filter delegate = getFilterBean(someBeanName);
	//조회한 필터 Bean 객체에 작업을 위임
	delegate.doFilter(request, response);
}
```

어떤 식으로 Bean 객체에 필터링 작업을 위임하는지 이해가 되나요?

<br/>

### `DelegatingFilterProxy` 의 Lazy Loading

`DelegatingFilterProxy` 의 또다른 이점은 바로 **필터 Bean 객체 조회를 나중에 한다는 것**입니다.

그래서 ‘작업을 위임할 필터 Bean 객체’가 참조될 ‘`DelegatingFilterProxy` 클래스의 필드 변수’에는 **null 값이 허용**되어 있습니다.

```java
public class DelegatingFilterProxy extends GenericFilterBean {
	@Nullable
	private volatile Filter delegate; //null 허용
	//...
}
```

만약 해당 필드가 null 값을 갖는다면, `doFilter()` 메서드가 호출됐을 때 실제 필터 Bean 객체를 가져옵니다.

왜 이런 식으로 구현을 해두었을까요?

사실 이런 구조는 반드시 필요합니다.

**서블릿 컨테이너는 실행될 때, 반드시 필터 객체를 가지고 있어야 합니다.**

**하지만 Spring 컨테이너는 이보다 나중에 Bean 객체를 등록합니다.**

따라서 이 시점 차이를 극복하기 위해, Lazy Loading을 할 수 있도록 지원합니다.

<br/><br/>

## `FilterChainProxy` 와 **`SecurityFilterChain`**

위에서 `DelegatingFilterProxy` 에 대해 알아보았습니다.

다시 말하자면, `DelegatingFilterProxy` 은 Spring Security 가 아닌 Spring 자체에서 지원하는 구현체입니다.

그렇다면, Spring Security와 `DelegatingFilterProxy` 은 무슨 관계가 있을까요?

하나씩 알아봅시다.

<br/>

### `FilterChainProxy` 가 뭐죠?

Spring Security는 `FilterChainProxy` 을 사용해서 서블릿 컨테이너와 소통합니다.

`FilterChainProxy` 은 Spring Security에서 제공하는 특별한 필터입니다.

**이 `FilterChainProxy` 은 위에서 살펴본 `DelegatingFilterProxy` 에 의해서 호출됩니다.**

그리고 `FilterChainProxy` 을 통해서 `SecurityFilterChain` 을 호출되고, 다시 다른 필터들에게 필터링 작업을 위임하죠.

> `SecurityFilterChain` 은 조금 있다가 설명할게요!
> 

`FilterChainProxy` 은 Bean 객체이기 때문에, `DelegatingFilterProxy` 에 의해서 호출됩니다.

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled%202.png)

<br/>

### 그럼 `SecurityFilterChain` 은 뭔가요?

`SecurityFilterChain` 은 바로 위에서 살펴본 `FilterChainProxy` 에 의해서 호출됩니다.

`SecurityFilterChain` 은 ‘현재 처리해야 하는 요청’에 대해서, 어떤 `SecurityFilter` 를 호출해야 하는지 결정하는 역할을 합니다.

> **`SecurityFilter` 라는 이름의 인터페이스나 클래스가 존재하는 것은 아니고, 단순히 Spring Security에서 `Filter` 인터페이스를 구현해 제공하는 필터를 의미합니다.  
즉, 일반적인 필터와 동등합니다. Spring Security에서 제공한다는 의미로, 편의상 `SecurityFilter` 라고 명칭하겠습니다.**
> 

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled%203.png)

<br/>

### 그럼 `FilterChainProxy` 은 단순히 `SecurityFilterChain` 를 호출하는 역할인가요?

다시 `FilterChainProxy` 로 돌아와볼까요?

다시 말하자면, **`FilterChainProxy` 는 Spring Security와 서블릿 컨테이너를 연결하는 역할을 수행합니다.**

따라서 `FilterChainProxy` 를 사용했을 때, 얻을 수 있는 이점은 아래와 같습니다.

- `FilterChainProxy` 은 **Spring Security가 적용되는 시작 지점**이기 때문에, 이것을 활용하여 여러 작업을 할 수 있습니다.
    - Spring Security 사용에 문제가 발생했을 때, 이 부분부터 디버깅을 해나간다던지
    - Spring Security에서 발생하는 메모리 누수 문제를 해결하기 위한 작업을 수행한다던지 등…
- 언제 `SecurityFilterChain` 을 호출할지 더 유연한 선택이 가능합니다.
    - 서블릿 컨테이너에서는 필터의 적용 범위를 오직 URL로만 설정할 수 있습니다.
    - 하지만 `FilterChainProxy` 을 통해, 더 다양한 기준을 가지고 필터를 적용할 수 있습니다.

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled%204.png)

위 그림은 여러 `SecurityFilterChain` 이 존재할 때, `FilterChainProxy` 가 적절한 `SecurityFilterChain` 을 선택하고 호출하는 것을 의미합니다.

**중요한 것은 첫번째로 찾은 `SecurityFilterChain` 만 호출된다는 것입니다.**

- 만약 `/api/messages` URL로 요청이 들어오면, `SecurityFilterChain 0` 만 호출됩니다.
    - **왜냐하면, `SecurityFilterChain 0` 의 매칭 패턴인 `/api/**` 과 일치하고, 패턴이 일치하는 `SecurityFilterChain` 중 가장 첫번째에 위치해있기 때문입니다.**
    - 요청된 URL `/api/messages` 은 `SecurityFilterChain n` 의 패턴인 `/**` 과도 일치하지만, 가장 먼저 매칭된 `SecurityFilterChain` 은 `SecurityFilterChain 0` 이기 때문에 호출되지 않습니다.
- 만약 `/foo` URL로 요청이 들어오면, `SecurityFilterChain n` 이 호출됩니다.
    - 일치하는 `SecurityFilterChain` 의 매칭 패턴이 나타날 때까지, 계속 탐색하게 됩니다.

**그리고 `SecurityFilterChain` 마다 독립적으로 `Security Filter` 를 구성할 수 있다는 것도 중요합니다.**

위 그림에서 `SecurityFilterChain 0` 은 총 3개의 `Security Filter` 로 구성되어 있고, `SecurityFilterChain n` 은 총 4개의 `Security Filter` 로 구성되어 있습니다.

그리고 `Security Filter` 가 아예 존재하지 않을 수도 있습니다.

호출 흐름을 정리하자면, `DelegatingFilterProxy` → `FilterChainProxy` → `SecurityFilterChain` → `Security Filter 0` → … → `Security Filter N` 과 같은 흐름이 됩니다.

<br/><br/>

## `Security Filter`

`Security Filter` 에는 실질적인 필터링 로직이 담겨있습니다.

`Security Filter` 는 어떤 특별한 인터페이스나 클래스가 아니라, Java EE 스펙 중 필터 구현을 위한 `Filter` 인터페이스를 구현한 필터를 의미합니다.

평범한 필터이지만 Spring Security에서 제공하기 때문에 `Security Filter` 라고 명명하겠습니다.

`Security Filter` 는 `SecurityFilterChain` 이 List 형식으로 관리합니다.

그리고 글 초반부에 설명했듯이, 필터간의 순서를 잘 설정해야 하지만, 일반적인 경우에는 Spring Security의 필터 순서를 알 필요까지는 없습니다.

하지만 어떤 순서로 적용이 되는지 알고 싶다면, 아래 링크를 참고하세요.

[https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)

<br/><br/>

## Security 관련 예외 처리

이번에는 Spring Security에서 발생하는 보안 관련 예외가 어떻게 처리되는지 알아보겠습니다.

<br/>

### `ExceptionTranslationFilter`

Spring Security에서는 발생한 예외를 HTTP 응답으로 변환할 수 있도록 지원합니다.

이 변환 작업은 `ExceptionTranslationFilter` 통해서 진행됩니다.

`ExceptionTranslationFilter` 를 사용해서, `AccessDeniedException` 과 `AuthenticationException` 예외를 catch하여 처리합니다.

그리고 `ExceptionTranslationFilter` 도 일종의 필터 즉, `SecurityFilter` 이기 때문에, 다른 Spring Security에서 제공하는 필터와 마찬가지로 `SecurityFilterChain` 에서 관리되고 `FilterChainProxy` 에 의해 호출됩니다.

### `ExceptionTranslationFilter` 동작 절차

![Untitled](/assets/img/2023-03-24-SpringSecurity_Structure/Untitled%205.png)

위 그림은 `ExceptionTranslationFilter` 와 다른 컴포넌트 간의 관계를 나타냅니다.

한 단계씩 알아볼까요?

**아래 단계는 `ExceptionTranslationFilter` 가 동작하는 방식에 대한 설명입니다.**

1. 요청이 들어오고 `ExceptionTranslationFilter` 가 실행됩니다.
2. 이제부터 설명하는 모든 단계는 `ExceptionTranslationFilter` 에서 이루어집니다!
3. 일단 아무런 동작을 하지않고 다음 필터를 호출합니다.
4. 다른 필터들이 호출되다가 `AuthenticationExcpetion` 예외가 발생하면, Authentication 작업을 시작합니다.
    1. 이때 먼저 `SecurityContextHolder` 를 비웁니다.
        
        > `SecurityContextHolder` 은 다음 포스팅(Authentication)에서 다루겠습니다!
        > 
    2. 그리고 기존의 `HttpServletRequest` 객체는 `RequestCache` 를 통해서, 서버측 세션에 저장됩니다. (인증 성공 시, 다시 사용하기 위함)
    3. `AuthenticationEntryPoint` 를 통해서, 클라이언트에게 인증을 요청합니다.  
    예를 들어, 클라이언트를 로그인 페이지로 Redirect시키거나, WWW-Authenticate 헤더와 함께 응답 메시지를 보낼 수 있습니다.
        
        > WWW-Authenticate 헤더에 관한 내용은 [여기를 참고](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate)하세요.
        > 
5. 만약 클라이언트가 Authentication 작업에 성공한다면, 2-b 번 단계에서 서버 세션에 저장한 `HttpServletRequest` 를 다시 가져옵니다.  
그리고 해당 요청 URL로 다시 Redirect 시켜서, 원래 요청을 처리할 수 있도록 합니다.
6. 만약 클라이언트가 Authentication 작업에 실패했다면, `AccessDeniedException` 이 발생하고, 원래 요청이 거부됩니다.  
`AccessDeniedHandler` 가 호출돼서, 요청 거부를 제어하게 됩니다.

`ExceptionTranslationFilter` 의 이런 로직은 아래 수도코드로 나타낼 수 있습니다.

```java
try {
	//1.
	filterChain.doFilter(request, response);
} catch (AccessDeniedException | AuthenticationException ex) {
	//2.
	if (!authenticated || ex instanceof AuthenticationException) {
		startAuthentication();
	//3.
	} else {
		accessDenied();
	}
}
```

1. 아직 아무런 작업을 하지 않고, 다음 필터를 호출하여 필터 체이닝을 합니다.
2. 만약 사용자가 인증되어 있지 않거나, `AuthenticationException` 예외가 발생한 경우 인증을 시작합니다.
3. 아니라면 접근을 거부합니다.

<br/><br/>

## 요청 저장

방금 `ExceptionTranslationFilter` 를 설명하면서, 인증되지 않은 상태라면 **기존의 요청을 저장**하고 인증을 클라이언트에게 요청한다고 설명했습니다.

좀 더 자세히 알아볼까요?

‘인증되지 않은 사용자’가 ‘인증된 사용자만 접근할 수 있는 리소스’를 요청했을 때, 인증을 하고 난 뒤 원래 요청을 처리하기 위해선, 이 요청 정보를 서버가 가지고 있어야 합니다.

이 작업을 위해, Spring Security에서는 `RequestCache` 인터페이스의 구현체를 사용해서 `HttpServletRequest` 를 저장합니다.

그럼 `RequestCache` 에 대해 좀 더 알아볼까요?

<br/>

### `RequestCache`

`HttpServletRequest` 는 `RequestCache` 에 의해서 저장됩니다.

이렇게 저장된 `HttpServletRequest` 는 클라이언트가 성공적으로 인증을 마쳤을 때, 다시 원래 요청을 처리하는데 사용됩니다.

그리고 이 `RequestCache` 를 사용하는 것은 `RequestCacheAwareFilter` 입니다.

`RequestCacheAwareFilter` 는 `RequestCache` 를 사용해서 `HttpServletRequest` 를 저장합니다.

참고로 `RequestCache` 의 가장 기본적인 구현체는 `HttpSessionRequestCache` 입니다.

<br/>

### 어떤 요청에 대해서 세션을 조회할까?

기본적으로 Spring Security는 `HTTP 요청 메시지의 파라미터` 에 `continue` 라는 이름이 있으면, 해당 요청에 대해서 세션을 조회합니다.

`/api?continue` 처럼 요청을 보내면, 세션에 관련 요청이 저장됐는지 조회합니다.

아래 코드는 `continue` 라는 이름 대신 다른 이름을 사용할 수 있도록 하는 Configuration 코드입니다.

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
	HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
	requestCache.setMatchingRequestParameterName("myParamName"); //Customized
	http
		// ...
		.requestCache((cache) -> cache
			.requestCache(requestCache)
		);
	return http.build();
}
```

<br/>

### 만약 요청이 저장되는 것을 막고 싶다면?

인증되지 않은 사용자가 로그인 후 무조건 홈페이지로 Redirect 되길 바란다면, 굳이 세션에 기존 요청을 저장하지 않아도 됩니다.

따라서 이 경우에는 아래와 같이, 해당 기능을 끌 수 있습니다.

```java
@Bean
SecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
    RequestCache nullRequestCache = new NullRequestCache();
    http
        // ...
        .requestCache((cache) -> cache
            .requestCache(nullRequestCache)
        );
    return http.build();
}
```

`NullRequestCache` 를 사용해 해당 기능을 끌 수 있습니다.

<br/><br/>

## 정리

지금까지 Spring Security의 전체적인 큰 구조에 대해 알아보았습니다!

너무 길고 복잡한 내용이었지만, 하나씩 읽고 추가적으로 학습해 나간다면 충분히 이해할 수 있을 것입니다.

다음에는 Spring Security의 인증 관련 내용을 정리해보겠습니다.

<br/><br/>

## References

- [https://docs.spring.io/spring-security/reference/servlet/architecture.html](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [https://docs.spring.io/spring-framework/docs/6.0.5/javadoc-api/org/springframework/web/filter/DelegatingFilterProxy.html](https://docs.spring.io/spring-framework/docs/6.0.5/javadoc-api/org/springframework/web/filter/DelegatingFilterProxy.html)
- [https://uchupura.tistory.com/24](https://uchupura.tistory.com/24)