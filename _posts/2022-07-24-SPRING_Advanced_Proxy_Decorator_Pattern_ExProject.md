---
category: Spring-ADV
tags: [Spring-Advanced]
title: "[스프링-ADV] 프록시 패턴, 데코레이터 패턴 예시 프로젝트 설정"
date:   2022-07-24 17:00:00 
lastmod : 2022-07-24 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 프록시 패턴, 데코레이터 패턴 학습을 위한 예시 프로젝트 설정

## 개요

- 이번 포스팅에선 다음에 설명할 ‘프록시 패턴'과 ‘데코레이터 패턴'을 위한 예시 프로젝트를 설정하겠다.
- 실제 프로젝트 파일은 아래에서 받을 수 있다.
    - [https://github.com/TaegyunWoo/spring-advanced-study/tree/main/proxy](https://github.com/TaegyunWoo/spring-advanced-study/tree/main/proxy)
    
    > 앞으로 설명할 모든 코드가 들어있다.

<br/>

## 예제 프로젝트 만들기 - v1

- 다양한 상황에서 프록시 사용법을 이해하기 위해 다음과 같은 기준으로 기본 예제 프로젝트를 만들어보자.
- 예제 종류
    - v1
        - 인터페이스와 구현 클래스 (MVC)
        - 스프링빈으로 수동 등록
    - v2
        - 인터페이스 없는 구체 클래스 (MVC)
        - 스프링 빈으로 수동 등록
    - v3
        - 컴포넌트 스캔으로 스프링 빈 자동 등록 (MVC)

### `OrderRepositoryV1` 인터페이스

```java
package hello.proxy.app.v1;

public interface OrderRepositoryV1 {
    void save(String itemId);
}
```

### `OrderRepositoryV1Impl` 클래스

```java
package hello.proxy.app.v1;

public class OrderRepositoryV1Impl implements OrderRepositoryV1 {
  @Override
  public void save(String itemId) {
    //저장 로직
    if (itemId.equals("ex")) {
      throw new IllegalStateException("예외 발생!");
    }
    sleep(1000);
  }

  private void sleep(int millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

### `OrderServiceV1` 인터페이스

```java
package hello.proxy.app.v1;

public interface OrderServiceV1 {
  void orderItem(String itemId);
}
```

### `OrderServiceV1Impl` 클래스

```java
package hello.proxy.app.v1;

public class OrderServiceV1Impl implements OrderServiceV1 {

  private final OrderRepositoryV1 orderRepository;

  public OrderServiceV1Impl(OrderRepositoryV1 orderRepository) {
    this.orderRepository = orderRepository;
  }

  @Override
  public void orderItem(String itemId) {
    orderRepository.save(itemId);
  }
}
```

### `OrderControllerV1` 인터페이스

```java
package hello.proxy.app.v1;

@RequestMapping //스프링은 @Controller 또는 @RequestMapping 이 있어야 스프링 컨트롤러로 인식
@ResponseBody
public interface OrderControllerV1 {

  @GetMapping("/v1/request")
  String request(@RequestParam("itemId") String itemId);

  @GetMapping("/v1/no-log")
  String noLog();
}
```

- `@RequestMapping`
    - 스프링MVC는 타입에 `@Controller` 또는 `@RequestMapping` 애너테이션이 있어야 스프링 컨트롤러로 인식한다.
    - 스프링 컨트롤러로 인식해야 HTTP URL이 매핑되고 동작한다.
    - 이 애너테이션은 인터페이스에 사용해도 된다.
- `@ResponseBody`
    - HTTP 메시지 컨버터를 사용해서 응답한다.
    - 이 애너페이션도 인터페이스에 사용해도 된다.
- `@RequestParam("itemId") String itemId`
    - 애너테이션을 생략해도 정상적으로 동작하지만, 인터페이스에 넣을 경우 자바 버전에 따라 인식하지 못할 수 있다.
    - 따라서 꼭 넣어주자.
- `request(..)` , `noLog()`
    - `request(..)` 는 `LogTrace` 를 적용할 대상이다.
    - `noLog()` 는 `LogTrace` 를 적용하지 않을 대상이다.

### `OrderControllerV1Impl` 클래스

```java
package hello.proxy.app.v1;

@Slf4j
public class OrderControllerV1Impl implements OrderControllerV1 {

  private final OrderServiceV1 orderService;

  public OrderControllerV1Impl(OrderServiceV1 orderService) {
    this.orderService = orderService;
  }

  @Override
  public String request(String itemId) {
    orderService.orderItem(itemId);
    return "ok";
  }

  @Override
  public String noLog() {
    return "ok";
  }
}
```

### `AppV1Config` 클래스

이제 스프링 빈으로 `Repository`, `Service`, `Controller` 를 모두 수동 등록해보자.

> 각 구현체에 `@Component` 가 없기 때문에 수동으로 등록해야 한다.

```java
package hello.proxy.config;

@Configuration
public class AppV1Config {
  @Bean
  public OrderControllerV1 orderControllerV1() {
    return new OrderControllerV1Impl(orderServiceV1());
  }

  @Bean
  public OrderServiceV1 orderServiceV1() {
    return new OrderServiceV1Impl(orderRepositoryV1());
  }

  @Bean
  public OrderRepositoryV1 orderRepositoryV1() {
    return new OrderRepositoryV1Impl();
  }
}
```

### `ProxyApplication` 에 코드 추가

스프링 애플리케이션을 실행시키는 메인 메서드가 포함된 클래스에 `AppV1Config` 를 등록시켜서 작동시키자.

```java
package hello.proxy;

@Import(AppV1Config.class)
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

}
```

- `@Import(AppV1Config.class)`
    - 애너테이션 속성으로 넘긴 클래스를 스프링 빈으로 등록한다.
    - 여기서는 `AppV1Config.class` 를 스프링 빈으로 등록한다.
    - **일반적으로 `@Configuration` 같은 설정 파일을 등록할 때 사용하지만, 스프링 빈을 등록할 때도 사용할 수 있다.**
- `@SpringBootApplication(scanBasePackages = "hello.proxy.app")`
    - `@ComponentScan` 의 기능과 같다. 즉, 컴포넌트 스캔을 시작할 위치를 지정한다.
    - `AppV1Config` 의 패키지 경로는 `hello.proxy.config` 이므로, 컴포넌트 스캔으로 자동 빈 등록이 되지 않는다. (`@Configuration` 내부에 `@Component` 가 존재한다.)
    
    > 추후 설명을 하기에 용이한 구조로 설정하기 위해 위와 같이 설정했다.
    > 

### 실행 결과

`/v1/request?itemId=hello` 실행시, `ok` 출력

<br/>

## 예제 프로젝트 만들기 - v2

이번에는 인터페이스가 없는 `Controller` , `Service` , `Repository` 를 스프링 빈으로 수동 등록해보자.

### `OrderRepositoryV2` 클래스

```java
package hello.proxy.app.v2;

public class OrderRepositoryV2 {
  public void save(String itemId) {
    //저장 로직
    if (itemId.equals("ex")) {
      throw new IllegalStateException("예외 발생!");
    }
    sleep(1000);
  }

  private void sleep(int millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

### `OrderServiceV2` 클래스

```java
package hello.proxy.app.v2;

public class OrderServiceV2 {
  private final OrderRepositoryV2 orderRepository;

  public OrderServiceV2(OrderRepositoryV2 orderRepository) {
    this.orderRepository = orderRepository;
  }

  public void orderItem(String itemId) {
    orderRepository.save(itemId);
  }
}
```

### `OrderControllerV2` 클래스

```java
package hello.proxy.app.v2;

@Slf4j
@RequestMapping
@ResponseBody
public class OrderControllerV2 {

  private final OrderServiceV2 orderService;

  public OrderControllerV2(OrderServiceV2 orderService) {
    this.orderService = orderService;
  }

  @GetMapping("/v2/request")
  public String request(String itemId) {
    orderService.orderItem(itemId);
    return "ok";
  }

  @GetMapping("/v2/no-log")
  public String noLog() {
    return "ok";
  }
}
```

- `@Controller` 애너테이션이 없는 것을 확인할 수 있다.
    - 컨트롤러 클래스를 빈으로 수동 등록하기 위해, 해당 애너테이션을 작성하지 않았다.
    - `@Controller` 애너테이션 내부에 `@Component` 가 있기 때문에, `@Controller` 애너테이션을 붙이면 컴포넌트 스캔의 대상이 된다.
    - **스프링MVC는 `@Controller` 나 `@RequestMapping` 애너테이션이 있다면 컨트롤러로 인식하기 때문에, `@Controller` 가 없어도 컨트롤러로 인식하여 컨트롤러 역할을 할 수 있다.**

### `AppV2Config` 클래스

```java
package hello.proxy.config;

@Configuration
public class AppV2Config {

  @Bean
  public OrderControllerV2 orderControllerV2() {
    return new OrderControllerV2(orderServiceV2());
  }

  @Bean
  public OrderServiceV2 orderServiceV2() {
    return new OrderServiceV2(orderRepositoryV2());
  }

  @Bean
  public OrderRepositoryV2 orderRepositoryV2() {
    return new OrderRepositoryV2();
  }

}
```

### `ProxyApplication` 에 코드 추가

스프링 애플리케이션을 실행시키는 메인 메서드가 포함된 클래스에 `AppV2Config` 까지 등록시켜서 작동시키자.

```java
@Import({AppV1Config.class, AppV2Config.class}) //코드 추가됨
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

}
```

- 변경 사항
    - 기존: `@Import({AppV1Config.class})`
    - 변경: `@Import({AppV1Config.class, AppV2Config.class})`
    - `@Import` 안에 배열로 등록하고 싶은 설정파일을 다양하게 추가할 수 있다.

### 실행 결과

`/v2/request?itemId=hello` 실행시, `ok` 출력

<br/>

## 예제 프로젝트 만들기 - v3

이번에는 컴포넌트 스캔으로 스프링 빈을 자동 등록해보자.

### `OrderRepositoryV3` 클래스

```java
package hello.proxy.app.v3;

@Repository
public class OrderRepositoryV3 {
  public void save(String itemId) {
    //저장 로직
    if (itemId.equals("ex")) {
      throw new IllegalStateException("예외 발생!");
    }
    sleep(1000);
  }

  private void sleep(int millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

### `OrderServiceV3` 클래스

```java
package hello.proxy.app.v3;

@Service
public class OrderServiceV3 {
  private final OrderRepositoryV3 orderRepository;

  public OrderServiceV3(OrderRepositoryV3 orderRepository) {
    this.orderRepository = orderRepository;
  }

  public void orderItem(String itemId) {
    orderRepository.save(itemId);
  }
}
```

### `OrderControllerV3` 클래스

```java
package hello.proxy.app.v3;

@Slf4j
@RestController
public class OrderControllerV3 {

  private final OrderServiceV3 orderService;

  public OrderControllerV3(OrderServiceV3 orderService) {
    this.orderService = orderService;
  }

  @GetMapping("/v3/request")
  public String request(String itemId) {
    orderService.orderItem(itemId);
    return "ok";
  }

  @GetMapping("/v3/no-log")
  public String noLog() {
    return "ok";
  }
}
```

`ProxyApplication` 에서 `@SpringBootApplication(scanBasePackages = "hello.proxy.app")` 를 사용했고, 각각 `@RestController` , `@Service` , `@Repository` 를 사용했기 때문에 자동 컴포넌트 스캔의 대상이 된다.

### 실행 결과

`/v3/request?itemId=hello` 실행시, `ok` 출력

이제 ‘프록시 패턴', ‘데코레이터 패턴'을 학습하기 위한 기본 프로젝트 설정을 마쳤다.

<br/>

## 요구사항 추가

### 기존 요구사항

이전 게시글에서 아래 요구사항을 만족시키는 애플리케이션을 구현해보았다.

- 기존 요구사항
    - 모든 PUBLIC 메서드의 호출과 응답 정보를 로그로 출력
    - 애플리케이션의 흐름을 변경하면 안됨
        - 로그를 남긴다고 해서 비즈니스 로직의 동작에 영향을 주면 안됨
    - 메서드 호출에 걸린 시간
    - 정상 흐름과 예외 흐름 구분
        - 예외 발생시 예외 정보가 남아야 함
    - 메서드 호출의 깊이 표현
    - HTTP 요청을 구분
        - HTTP 요청 단위로 특정 ID를 남겨서 어떤 HTTP 요청에서 시작된 것인지 명확하게 구분이 가능해야 함
        - 트랜잭션 ID (DB 트랜잭션 아님)
- 기존 요구사항 결과 예시
    
    ```text
    //정상 동작 시
    [3cfc6878] OrderController.request()
    [3cfc6878] |-->OrderServiceV1.orderItem()
    [3cfc6878] |    |-->OrderRepository.save()
    [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [3cfc6878] OrderController.request() time=1014ms
    
    //예외 발생 시
    [1c444352] OrderController.request()
    [1c444352] |-->OrderServiceV1.orderItem()
    [1c444352] |    |-->OrderRepository.save()
    [1c444352] |    |<X-OrderRepository.save() time=0ms ex=java.lang.IllegalStateException: 예외 발생!
    [1c444352] |<X-OrderServiceV1.orderItem() time=10ms ex=java.lang.IllegalStateException: 예외 발생!
    [1c444352] OrderController.request() time=13ms ex=java.lang.IllegalStateException: 예외 발생!
    ```
    
- 기존 구현 방식의 한계
    - 하지만 이 요구사항을 만족하기 위해선, 기존 코드를 많이 수정해야 한다.
    - 코드 수정을 최소화 하기 위해 `템플릿 메서드 패턴`과 `콜백 패턴`도 사용했지만, 로그를 남기고 싶은 클래스가 수백개라면 수백개의 클래스를 모두 고쳐야 한다.
    - 로그를 남길 때 기존 원본 코드를 변경해야 한다는 사실 그 자체가 개발자에게는 가장 큰 문제로 남는다.

### 새로운 요구사항

기존 요구사항에 아래 요구사항이 추가되었다고 하자.

- 요구사항 추가
    - 원본 코드를 전혀 수정하지 않고, 로그 추적기를 적용해라.
    - 특정 메서드는 로그를 출력하지 않는 기능
        - 보안상 일부는 로그를 출력하면 안된다.
    - 다음과 같은 다양한 케이스에 적용할 수 있어야 한다.
        - v1 - 인터페이스가 있는 구현 클래스에 적용
        - v2 - 인터페이스가 없는 구현 클래스에 적용
        - v3 - 컴포넌트 스캔 대상에 기능 적용

위 요구사항들을 만족시키기 위해, ‘프록시 패턴'과 ‘데코레이터 패턴'에 대해 다음 포스팅에서 설명하겠다.

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*