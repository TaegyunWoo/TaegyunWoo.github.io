---
category: Spring-ADV
tags: [스프링]
title: "[스프링-ADV] 고급 내용 설명을 위한 프로젝트 설정"
date:   2022-05-29 15:30:00 
lastmod : 2022-05-29 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 기본 프로젝트

앞으로 포스팅할 ‘스프링 고급원리’ 내용을 설명하기 위해 사용될 프로젝트를 생성해야 한다.

이 포스팅을 통해, 기본 프로젝트를 생성하고 앞으로의 내용을 하나씩 설명하겠다.

> 본 포스팅에서 설명하는 프로젝트를 가지고 점점 발전시켜나가는 형태로 포스팅할 예정이다.

<br/>

기본 프로젝트 Repository 는 아래와 같다.

[https://github.com/TaegyunWoo/spring-advanced-study/tree/init](https://github.com/TaegyunWoo/spring-advanced-study/tree/init)

<br/>

> 본 포스팅과 앞으로 작성할 글은 모두 **인프런의 김영한님의 ‘스프링 핵심원리 - 고급편’을 기반으로 작성**된 글입니다.  
자세한 내용은 아래 링크를 참고해주시길 바랍니다!!!  
[인프런 김영한 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)

<br/><br/>

## 새 프로젝트 생성

### 프로젝트 생성 링크

https://start.spring.io

위 링크를 통해, 프로젝트를 생성한다.

### 프로젝트 선택

- Project: Gradle Project
- Language: Java
- Spring Boot: 2.5.x

### 프로젝트 메타데이터

- Group: hello
- Artifact: advanced
- Name: advanced
- Package name: hello.advanced
- Packaging: Jar
- Java: 11

### 의존성 추가

- Dependencies: Spring Web, Lombok

<br/><br/>

## 기본 비즈니스 로직 작성

### `OrderRepositoryV0`

```java
@Repository
@RequiredArgsConstructor
public class OrderRepositoryV0 {

  public void save(String itemId) {
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

### `OrderServiceV0`

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV0 {

  private final OrderRepositoryV0 orderRepository;

  public void orderItem(String itemId) {
    orderRepository.save(itemId);
  }

}
```

### `OrderControllerV0`

```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV0 {
  private final OrderServiceV0 orderService;

  @GetMapping("/v0/request")
  public String request(String itemId) {
    orderService.orderItem(itemId);
    return "ok";
  }
}
```

<br/><br/>

## 로그 추적기 작성

### 요구사항

- 모든 PUBLIC 메서드의 호출과 응답 정보를 로그로 출력하라.
- 애플리케이션의 흐름을 변경하면 안된다.
- 메서드 호출에 걸린 시간도 출력하라.
- 정상 흐름과 예외 흐름을 구분하고, 예외 발생시 예외 정보가 남아야 한다.
- 메서드 호출의 깊이를 표현해서 출력하라.
- HTTP 요청을 구분해서 출력하라.

### `TraceId`

```java
/**
 * 트랜잭션과 호출 깊이 정보를 가지고 있는 클래스
 */
public class TraceId {

  private String id; //트랜잭션 id
  private int level; //호출 depth

  public TraceId() {
    this.id = createId();
    this.level = 0;
  }

  private TraceId(String id, int level) {
    this.id = id;
    this.level = level;
  }

  //식별자 랜덤 생성
  private String createId() {
    return UUID.randomUUID().toString().substring(0, 8);
  }

  //'동일한 ID, 깊이 1 증가'한 TraceId를 반환하는 메서드
  public TraceId createNextId() {
    return new TraceId(id, level + 1);
  }

  //'동일한 ID, 깊이 1 감소'한 TraceId를 반환하는 메서드
  public TraceId createPreviousId() {
    return new TraceId(id, level - 1);
  }

  //첫번째 레벨여부 확인하는 메서드
  public boolean isFirstLevel() {
    return level == 0;
  }

  //GETTER 생략
}
```

> UUID는 ‘Universally Unique Identifier’ 의 약자로, 소프트웨어 구축에 쓰이는 식별자 표준이다.  
(*위키 인용)*
> 

### `TraceStatus`

로그의 상태정보를 나타내는 클래스

```java
public class TraceStatus {

  private TraceId traceId; //위에서 정의한 클래스
  private Long startTimeMs; //호출된 시간
  private String message;

  public TraceStatus(TraceId traceId, Long startTimeMs, String message) {
    this.traceId = traceId;
    this.startTimeMs = startTimeMs;
    this.message = message;
  }

  //GETTER 생략
}
```

### `HelloTraceV1`

```java
@Slf4j
@Component
public class HelloTraceV1 {

  private static final String START_PREFIX = "-->";
  private static final String COMPLETE_PREFIX = "<--";
  private static final String EX_PREFIX = "<X-";

  //PUBLIC 메서드 호출시, 호출할 메서드
  public TraceStatus begin(String message) {
    TraceId traceId = new TraceId();
    Long startTimeMx = System.currentTimeMillis();
    log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);
    return new TraceStatus(traceId, startTimeMx, message);
  }

  //PUBLIC 메서드 정상 종료시, 호출할 메서드
  public void end(TraceStatus status) {
    complete(status, null);
  }

  //예외 발생 시, 호출할 메서드
  public void exception(TraceStatus status, Exception e) {
    complete(status, e);
  }

  //호출완료 처리 메서드
  private void complete(TraceStatus status, Exception e) {
    Long stopTimeMs = System.currentTimeMillis();
    long resultTimeMs = stopTimeMs - status.getStartTimeMs();
    TraceId traceId = status.getTraceId();
    if (e == null) {
      log.info("[{}] {}{} time={}ms", traceId.getId(),
        addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
    } else {
      log.info("[{}] {}{} time={}ms ex={}", traceId.getId(),
        addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
    }
  }

  //호출 깊이만큼 접두어를 추가하는 메서드
  private static String addSpace(String prefix, int level) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < level; i++) {
      sb.append((i == level - 1) ? "|" + prefix : "|    ");
    }
    return sb.toString();
  }
}
```

<br/><br/>

## 로그 추적기 V1 적용하기

### `OrderControllerV1`

```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV1 {

  private final OrderServiceV1 orderService;
  private final HelloTraceV1 trace;

  @GetMapping("/v1/request")
  public String request(String itemId) {
    TraceStatus status = null;
    try {
      status = trace.begin("OrderController.request()");
      orderService.orderItem(itemId);
      trace.end(status);
      return "ok";
    } catch (Exception e) {
      trace.exception(status, e);
      throw e; //반드시 예외를 다시 던져주어야 한다. (로깅 로직이 원래 로직에 영향을 주면 안되므로)
    }
  }
}
```

### `OrderServiceV1`

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV1 {
  private final OrderRepositoryV1 orderRepository;
  private final HelloTraceV1 trace;

  public void orderItem(String itemId) {
    TraceStatus status = null;

    try {
      status = trace.begin("OrderServiceV1.orderItem()");
      orderRepository.save(itemId);
      trace.end(status);
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }
  }
}
```

### `OrderRepositoryV1`

```java
@Repository
@RequiredArgsConstructor
public class OrderRepositoryV1 {

  private final HelloTraceV1 trace;

  public void save(String itemId) {
    TraceStatus status = null;

    try {
      status = trace.begin("OrderRepository.save()");

      if (itemId.equals("ex")) {
        throw new IllegalStateException("예외 발생!");
      }
      sleep(1000);

      trace.end(status);
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }

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

### 적용 결과

- 아직 Level 관련 기능을 개발하지 않았다. 따라서 Level은 항상 0이다.
- 트랜잭션 ID 값도 다르다.
- `/v1/request?itemId=hello` 요청 결과 (정상실행)
    
    ```
    [11111111] OrderController.request()
    [22222222] OrderService.orderItem()
    [33333333] OrderRepository.save()
    [33333333] OrderRepository.save() time=1000ms
    [22222222] OrderService.orderItem() time=1001ms
    [11111111] OrderController.request() time=1001ms
    ```
    
- `/v1/request?itemId=ex` 요청 결과 (예외실행)
    
    ```
    [5e110a14] OrderController.request()
    [6bc1dcd2] OrderService.orderItem()
    [48ddffd6] OrderRepository.save()
    [48ddffd6] OrderRepository.save() time=0ms ex=java.lang.IllegalStateException: 예외 발생!
    [6bc1dcd2] OrderService.orderItem() time=6ms ex=java.lang.IllegalStateException: 예외 발생!
    [5e110a14] OrderController.request() time=7ms ex=java.lang.IllegalStateException: 예외 발생!
    ```
    

### 개선 방법

- 현재 로그의 상태 정보인 `트랜잭션ID` 와 `level` 이 다음으로 전달되어야 한다.
    - 즉, `TraceId` 객체가 전달되어야 한다.
- 가장 단순한 방법인, **‘매개변수를 통한 전달’** 을 구현해보자.

<br/><br/>

## 로그 추적기 V2 적용하기

### 해결해야 하는 문제

- 메서드 호출의 깊이 표현
- 같은 HTTP 요청이면 같은 트랜잭션 ID 남기기

### `HelloTraceV2`

```java
@Slf4j
@Component
public class HelloTraceV2 {

  private static final String START_PREFIX = "-->";
  private static final String COMPLETE_PREFIX = "<--";
  private static final String EX_PREFIX = "<X-";

  public TraceStatus begin(String message) {
    TraceId traceId = new TraceId();
    Long startTimeMx = System.currentTimeMillis();
    log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);
    return new TraceStatus(traceId, startTimeMx, message);
  }

  /**V2에서 추가된 메서드
   * 매개변수로 TraceId를 전달받아, Context를 맞춘다.
   */
  public TraceStatus beginSync(TraceId beforeTraceId, String message) {
    TraceId nextId = beforeTraceId.createNextId();
    Long startTimeMx = System.currentTimeMillis();
    log.info("[{}] {}{}", nextId.getId(), addSpace(START_PREFIX, nextId.getLevel()), message);
    return new TraceStatus(nextId, startTimeMx, message);
  }

  public void end(TraceStatus status) {
    complete(status, null);
  }

  public void exception(TraceStatus status, Exception e) {
    complete(status, e);
  }

  private void complete(TraceStatus status, Exception e) {
    Long stopTimeMs = System.currentTimeMillis();
    long resultTimeMs = stopTimeMs - status.getStartTimeMs();
    TraceId traceId = status.getTraceId();
    if (e == null) {
      log.info("[{}] {}{} time={}ms", traceId.getId(),
        addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
    } else {
      log.info("[{}] {}{} time={}ms ex={}", traceId.getId(),
        addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
    }
  }

  private static String addSpace(String prefix, int level) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < level; i++) {
      sb.append((i == level - 1) ? "|" + prefix : "|    ");
    }
    return sb.toString();
  }
}
```

### V2 적용 방법

- 처음 로그를 남기는 `OrderController.request()` 에서 로그를 남길 때, 어떤 깊이와 어떤 트랜잭션 ID를 사용했는지 다음 차례인 `OrderService.orderItem()` 에 로그를 남기는 시점에 알아야 한다.
- **즉 현재 로그의 상태 정보인 `트랜잭션ID` 와 `level` 이 다음으로 전달되어야 한다.**

![Untitled](/assets/img/2022-05-29-SPRING_Advanced_Project_Initialize/Untitled.png)

이제 본격적으로 적용해보자.

### `OrderControllerV2`

```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV2 {

  private final OrderServiceV2 orderService;
  private final HelloTraceV2 trace;

  @GetMapping("/v2/request")
  public String request(String itemId) {
    TraceStatus status = null;
    try {
			//처음으로 처리할 때는 begin 메서드 사용
      status = trace.begin("OrderController.request()");
      orderService.orderItem(status.getTraceId(), itemId);
      trace.end(status);
      return "ok";
    } catch (Exception e) {
      trace.exception(status, e);
      throw e; //반드시 예외를 다시 던져주어야 한다. (로깅 로직이 원래 로직에 영향을 주면 안되므로)
    }
  }
}
```

### `OrderServiceV2`

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV2 {
  private final OrderRepositoryV2 orderRepository;
  private final HelloTraceV2 trace;

	//파라미터로 TraceId 를 넘겨받는다.
  public void orderItem(TraceId traceId, String itemId) {
    TraceStatus status = null;

    try {
			//넘겨받은 TraceId를 다시 사용한다.
      status = trace.beginSync(traceId, "OrderServiceV1.orderItem()");
      orderRepository.save(status.getTraceId(), itemId);
      trace.end(status);
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }
  }
}
```

### `OrderRepositoryV2`

```java
@Repository
@RequiredArgsConstructor
public class OrderRepositoryV2 {

  private final HelloTraceV2 trace;

	//파라미터로 TraceId 를 넘겨받는다.
  public void save(TraceId traceId, String itemId) {
    TraceStatus status = null;

    try {
			//넘겨받은 TraceId를 다시 사용한다.
      status = trace.beginSync(traceId, "OrderRepository.save()");

      if (itemId.equals("ex")) {
        throw new IllegalStateException("예외 발생!");
      }
      sleep(1000);

      trace.end(status);
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }

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

### 적용 결과

- Level 관련 기능을 개발했다.
- 동일한 HTTP 요청이라면, 트랜잭션 ID 값도 같도록 수정했다.
- `/v1/request?itemId=hello` 요청 결과 (정상실행)
    
    ```
    [c80f5dbb] OrderController.request()
    [c80f5dbb] |-->OrderService.orderItem()
    [c80f5dbb] | |-->OrderRepository.save()
    [c80f5dbb] | |<--OrderRepository.save() time=1005ms
    [c80f5dbb] |<--OrderService.orderItem() time=1014ms
    [c80f5dbb] OrderController.request() time=1017ms
    ```
    
- `/v1/request?itemId=ex` 요청 결과 (예외실행)
    
    ```
    [ca867d59] OrderController.request()
    [ca867d59] |-->OrderService.orderItem()
    [ca867d59] | |-->OrderRepository.save()
    [ca867d59] | |<X-OrderRepository.save() time=0ms ex=java.lang.IllegalStateException: 예외 발생!
    [ca867d59] |<X-OrderService.orderItem() time=7ms ex=java.lang.IllegalStateException: 예외 발생!
    [ca867d59] OrderController.request() time=7ms ex=java.lang.IllegalStateException: 예외 발생!
    ```
    

### 여전히 남은 문제

- HTTP 요청을 구분하고 깊이를 표현하기 위해, `TraceId` 동기화가 필요하다.
    - `TraceId` 동기화를 위해, **관련 메서드의 모든 파라미터를 수정**해야 한다.
- **로그를 처음 시작할 때는 `begin()` 을 호출하고, 처음이 아닐때는 `beginSync()` 를 호출해야 한다.**

이러한 문제를 **쓰레드 로컬**을 통해 해결할 수 있다. 다음 포스팅에서는 ‘쓰레드 로컬’에 대해 설명하겠다.

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*