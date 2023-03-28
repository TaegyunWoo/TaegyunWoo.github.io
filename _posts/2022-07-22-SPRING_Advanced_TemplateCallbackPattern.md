---
category: Spring-ADV
tags: [Spring-Advanced]
title: "[스프링-ADV] 템플릿 콜백 패턴"
date:   2022-07-22 20:10:00 
lastmod : 2022-07-22 20:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 템플릿 콜백 패턴

## 템플릿 콜백 패턴 - 시작

### 개요

- 이전 포스팅에서 전략 패턴에 대해 다뤘다.
    - [[스프링-ADV] 전략 패턴](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_StrategyPattern)
- 그리고 `ContextV1` 과 `ContextV2` 를 통해, 유연하게 전략을 바꾸는 방법을 소개했다.
- `ContextV2` 에서 변하는 부분이 매개변수로 넘어오고, 이렇게 넘어온 `Strategy` 의 코드를 실행해서 처리한다.
    - 이렇게 다른 코드의 인수로서 넘겨주는 실행 가능한 코드를 **콜백(callback)** 이라고 한다.
- `ContextV2` 예제에서 콜백은 `Strategy` 이다. 클라이언트에서는 직접 `Strategy` 를 실행하는 것이 아니라, 클라이언트가 `ContextV2.execute(..)` 를 실행할 때 `Strategy` 를 넘겨주고, `ContextV2` 뒤에서 `Strategy` 가 실행된다.

### 자바 언어에서의 콜백

주로 람다를 사용한다.

### 템플릿 콜백 패턴

- 스프링에서는 `ContextV2` 와 같은 방식의 전략 패턴을 템플릿 콜백 패턴이라고 한다.
- 전략 패턴에서 `Context` 가 템플릿 역할을 하고, `Strategy` 부분이 콜백으로 넘어온다 생각하면 된다.
- **참고로 템플릿 콜백 패턴은 GOF 패턴은 아니고, 스프링 내부에서 이런 방식을 자주 사용하기 때문에 스프링 안에서만 이렇게 부른다.**
    - 전략 패턴에서 템플릿과 콜백 부분이 강조된 패턴이라 생각하면 된다.
    - **스프링에서 이름에 `XxxTemplate` 이 있다면 그것은 템플릿 콜백 패턴으로 만들어져 있다고 생각하면 된다.**

![Untitled](/assets/img/2022-07-22-SPRING_Advanced_TemplateCallbackPattern/Untitled%204.png)

<br/>

## 템플릿 콜백 패턴 - 예시

- 이전 포스팅에서 다뤘던 `ContextV2` 과 크게 다르지 않고, 이름만 다르다.
    - [[스프링-ADV] 전략 패턴](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_StrategyPattern)

### 테스트 코드 `Callback` 인터페이스

```java
public interface Callback {
  void call();
}
```

- 콜백 로직을 전달할 인터페이스이다.
- 전략 패턴에서의 `Strategy` 인터페이스와 동일하다.

### 테스트 코드 `TimeLogTemplate`

```java
@Slf4j
public class TimeLogTemplate {

  public void execute(Callback callback) {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    callback.call();
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime = {}", resultTime);
  }
}
```

### 테스트 코드 `TemplateCallbackTest`

```java
@Slf4j
public class TemplateCallbackTest {

  /**
   * 템플릿 콜백 패턴 - 익명 내부 클래스
   */
  @Test
  void callbackV1() {
    TimeLogTemplate template = new TimeLogTemplate();

    template.execute(new Callback() {
      @Override
      public void call() {
        log.info("비즈니스 로직1 실행");
      }
    });

    template.execute(new Callback() {
      @Override
      public void call() {
        log.info("비즈니스 로직2 실행");
      }
    });
  }

  /**
   * 템플릿 콜백 패턴 - 람다
   */
  @Test
  void callbackV2() {
    TimeLogTemplate template = new TimeLogTemplate();
    template.execute(() -> log.info("비즈니스 로직1 실행"));
    template.execute(() -> log.info("비즈니스 로직2 실행"));
  }
}
```

- 별도의 클래스를 만들어서 전달해도 되지만, 콜백을 사용할 경우 익명 내부 클래스나 람다를 사용하는 것이 편리하다.

> 물론 여러 곳에서 함께 사용되는 경우 재사용을 위해 콜백을 별도의 클래스로 만들어도 된다.

<br/>

## 템플릿 콜백 패턴 - 적용

이제 드디어 실제 애플리케이션에 적용해서, 로그 추적기를 개선해보자!

### `TraceCallback` 인터페이스

```java
public interface TraceCallback<T> {
  T call();
}
```

- 콜백을 전달하는 인터페이스이다.
- `<T>` 제네릭을 사용했다. 콜백의 반환타입을 정의한다.

### `TraceTemplate`

```java
public class TraceTemplate {
  private final LogTrace trace;

  public TraceTemplate(LogTrace trace) {
    this.trace = trace;
  }

  /**
   * 파라미터로 콜백을 전달받는다.
   */
  public <T> T execute(String message, TraceCallback<T> callback) {
    TraceStatus status = null;
    try {
      status = trace.begin(message);

      //로직 호출
      T result = callback.call();

      trace.end(status);
      return result;
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }
  }
}
```

- `TraceTemplate` 은 템플릿 역할을 한다.
- `execute(..)` 을 보면 `message` 데이터와 `TraceCallback callback` 콜백을 전달받는다.
- `<T>` 제네릭을 사용하여, 반환타입을 정의한다.

### `OrderControllerV5`

```java
@RestController
public class OrderControllerV5 {

  private final OrderServiceV5 orderService;
  private final TraceTemplate template;

  //LogTrace를 주입받아, TraceTemplate에 다시 전달
  public OrderControllerV5(OrderServiceV5 orderService, LogTrace trace) {
    this.orderService = orderService;
    this.template = new TraceTemplate(trace);
  }

  @GetMapping("/v5/request")
  public String request(String itemId) {
    return template.execute("OrderController.request()", new TraceCallback<String>() {
      @Override
      public String call() {
        orderService.orderItem(itemId);
        return "ok";
      }
    });
  }
}
```

- `this.template = new TraceTemplate(trace)`
    - `trace` 의존관계 주입을 받으면서 필요한 `TraceTemplate` 템플릿을 생성한다.
    - 참고로 `TraceTemplate` 을 처음부터 스프링 빈으로 등록하고 주입받아도 된다.
- `template.execute(.., new TraceCallback(){..})`
    - 템플릿을 실행하면서 콜백을 전달한다.
    - 여기서는 콜백으로 익명 내부 클래스를 사용했다.

### `OrderServiceV5`

```java
@Service
public class OrderServiceV5 {
  private final OrderRepositoryV5 orderRepository;
  private final TraceTemplate template;

  //LogTrace를 주입받아, TraceTemplate에 다시 전달
  public OrderServiceV5(OrderRepositoryV5 orderRepository, LogTrace trace) {
    this.orderRepository = orderRepository;
    this.template = new TraceTemplate(trace);
  }

  public void orderItem(String itemId) {
    template.execute("OrderService.orderItem()", () -> {
      orderRepository.save(itemId);
      return null;
    });
  }
}
```

- 람다를 사용했다.

### `OrderRepositoryV5`

```java
@Repository
public class OrderRepositoryV5 {

  private final TraceTemplate template;

  //LogTrace를 주입받아, TraceTemplate에 다시 전달
  public OrderRepositoryV5(LogTrace trace) {
    this.template = new TraceTemplate(trace);
  }

  public void save(String itemId) {

    template.execute("OrderRepository.save()", (TraceCallback<Void>) () -> {
      if (itemId.equals("ex")) {
        throw new IllegalStateException("예외 발생!");
      }
      sleep(1000);
      return null;
    });
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

### 실행 결과

- 정상 실행 로그
    
    ```text
    [3cfc6878] OrderController.request()
    [3cfc6878] |-->OrderServiceV1.orderItem()
    [3cfc6878] |    |-->OrderRepository.save()
    [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [3cfc6878] OrderController.request() time=1014ms
    ```
    
<br/>

## 정리

### 지금까지…

- 지금까지 우리는 변하는 코드와 변하지 않는 코드를 분리하고, 더 적은 코드로 로그 추적기를 적용해봤다.
- 템플릿 메서드 패턴, 전략 패턴, 템플릿 콜백 패턴까지 진행하면서 변하는 코드와 변하지 않는 코드를 분리했다.
- 그리고 최종적으로 템플릿 콜백 패턴을 적용하고 람다를 사용해서 코드 사용도 최소화 할 수 있었다.

### 한계

- 그런데 지금까지 설명한 방식의 한계는 **“아무리 최적화를 해도 결국 로그 추적기를 적용하기 위해서 원본 코드를 수정해야 한다는 점”** 이다.
- 로그 추적기를 적용할 클래스가 수백개이면 여전히 수백개를 일일이 수정해야 한다.
- 다음 포스팅을 통해, 이 문제를 해결해보자.

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*