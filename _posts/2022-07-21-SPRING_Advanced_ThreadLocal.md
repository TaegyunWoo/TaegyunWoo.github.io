---
category: Spring-ADV
tags: [스프링]
title: "[스프링-ADV] 쓰레드 로컬"
date:   2022-07-21 15:30:00 
lastmod : 2022-07-21 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 쓰레드 로컬 - ThreadLocal

## 필드 동기화 - 개발

### 개요

- 이전 포스팅 글 ([[스프링-ADV] 고급 내용 설명을 위한 프로젝트 설정](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_Project_Initialize)) 에서 로그 추적기를 만들어보았다.
    - *해당 글을 먼저 꼭 읽자.*
- **여기에는 `트랜잭션ID` 와 `level` 을 동기화하는 문제가 있었다.**
- 위 문제를 해결하는 ‘기존의 해결방법’은 아래와 같다.
    - **`TraceId` 를 파라미터로 넘기기**
- 해당 해결방법의 문제는 아래와 같다.
    - **로그를 출력하는 모든 메서드에 `TraceId` 파라미터를 추가해야 한다.**
- `TraceId` 를 파라미터로 넘기지 않고 해당 문제를 해결해보자. 바로 코드를 통해 알아보자.

<br/>

### `LogTrace` 인터페이스

```java
import hello.advanced.trace.TraceStatus;

public interface LogTrace {
  TraceStatus begin(String message);

  void end(TraceStatus status);

  void exception(TraceStatus status, Exception e);
}
```

> 추후 다양한 구현체로 변경할 수 있도록 해당 인터페이스를 먼저 만들자.

<br/>

### `FieldLogTrace`

먼저 소스코드를 보고, 설명을 읽자.

```java
import hello.advanced.trace.TraceId;
import hello.advanced.trace.TraceStatus;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class FieldLogTrace implements LogTrace {
  private static final String START_PREFIX = "-->";
  private static final String COMPLETE_PREFIX = "<--";
  private static final String EX_PREFIX = "<X-";

	/*
	 * 새로 추가된 필드
	 * traceId 동기화, 동시성 이슈 발생, traceId를 계속 들고 있는다.
	 */
  private TraceId traceIdHolder;

  @Override
  public TraceStatus begin(String message) {
    syncTraceId(); //traceId를 동기화한다.

    TraceId traceId = traceIdHolder;
    Long startTimeMs = System.currentTimeMillis();
    log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);

    return new TraceStatus(traceId, startTimeMs, message);
  }

  @Override
  public void end(TraceStatus status) {
    complete(status, null);
  }

  @Override
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

    releaseTraceId(); //동기화를 해제할 수 있다면, 해제한다.
  }

	/**
	 * 새로 추가된 메서드
	 * TraceId를 동기화한다.
	 */
  private void syncTraceId() {
    if (traceIdHolder == null) {
      traceIdHolder = new TraceId();
    } else {
      traceIdHolder = traceIdHolder.createNextId();
    }
  }

	/**
	 * 새로 추가된 메서드
	 * TraceId를 해지한다.
	 */
  private void releaseTraceId() {
    if (traceIdHolder.isFirstLevel()) {
      traceIdHolder = null;
    } else {
      traceIdHolder = traceIdHolder.createPreviousId();
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

- 기존 코드인 `HelloTraceV2` 와의 차이점
    - `HelloTraceV2` 에서 `TraceId` 을 동기화하여 사용하기 위해, `beginSync(...)` 메서드를 만들고 파라미터로 `TraceId beforeTraceId` 를 선언했다.
    - 따라서 `beginSync(...)` 메서드에 `beforeTraceId` 를 전달하기 위해, 실제 비즈니스 로직에서도 `TraceId` 객체를 계속 전달받아야 했다. ← 문제점
    - **`FieldLogTrace` 에서는 이 문제를 해결하기 위해, `beginSync(...)` 메서드를 삭제하고 아래  필드와 메서드를 추가하여 사용했다.**
        - `TraceId traceIdHolder`
        - `syncTraceId()`
        - `releaseTraceId()`

- `TraceId traceIdHolder` 필드
    - 기존엔 `TraceId 객체` 를 메서드 내부에서만 사용했지만, 이것을 계속 들고 있을 수 있도록 필드에 저장한다.

- `syncTraceId()` 메서드
    - `begin(...)` 메서드를 통해 로그를 시작할 때, `TraceId 객체` 를 동기화하는 역할을 수행한다.
    - 기존 `TraceId 객체` 가 `traceIdHolder` 필드에 없을 경우 (컨트롤러에서 호출한 경우)
        - 처음으로 로그 추적을 시작하는 경우이므로
        - 새로운 `TraceId 객체` 를 생성하고 필드에 저장한다.
    - 기존 `TraceId 객체` 가 `traceIdHolder` 필드에 있을 경우 (컨트롤러 이후 로직에서 호출하는 경우)
        - `id값`과 `level`을 맞춰야 하므로
        - 기존 `TraceId 객체` 의 level(depth)를 증가시킨다.

- `releaseTraceId()` 메서드
    - `complete(...)` 메서드를 통해 로그를 종료할 때, `TraceId 객체` 를 해제하는 역할을 수행한다.
    - `TraceId 객체` 의 `level` 이 0인 경우 (컨트롤러에서 호출한 경우)
        - 모든 비즈니스 로직이 끝나고 로깅이 완전히 종료되는 시점이므로
        - 기존의 `TraceId 객체` 를 없앤다. (null)
    - `TraceId 객체` 의 `level` 이 1 이상인 경우 (컨트롤러 이후)
        - 기존 `TraceId 객체` 의 level(depth)를 감소시킨다.

- 아래는 실제로 출력되는 로그마다, 동작원리를 설명한 것이다.
    
    ```text
    [c80f5dbb] OrderController.request()                //syncTraceId(): 최초 호출 level=0
    [c80f5dbb] |-->OrderService.orderItem()             //syncTraceId(): 직전 로그 있음 level=1 증가
    [c80f5dbb] | |-->OrderRepository.save()             //syncTraceId(): 직전 로그 있음 level=2 증가
    [c80f5dbb] | |<--OrderRepository.save() time=1005ms //releaseTraceId(): level=2->1 감소
    [c80f5dbb] |<--OrderService.orderItem() time=1014ms //releaseTraceId(): level=1->0 감소
    [c80f5dbb] OrderController.request() time=1017ms    //releaseTraceId(): level==0, traceId 제거
    ```
    

### 테스트코드

테스트코드를 작성하여, 정상적으로 동작하는지 알아보자.

```java
import hello.advanced.trace.TraceStatus;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class FieldLogTraceTest {
  FieldLogTrace trace = new FieldLogTrace();

	/**
	 * 정상 동작 시
	 */
  @Test
  void begin_end_level2() {
    TraceStatus status1 = trace.begin("hello1"); //depth=1
    TraceStatus status2 = trace.begin("hello2"); //depth=2
    trace.end(status2); //depth=1 <- depth=2
    trace.end(status1); //depth=0 <- depth=1
  }

	/**
	 * 예외 동작 시
	 */
  @Test
  void begin_exception_level2() {
    TraceStatus status1 = trace.begin("hello1"); //depth=1
    TraceStatus status2 = trace.begin("hello2"); //depth=2
    trace.exception(status2, new IllegalStateException()); //depth=1 <- depth=2 (exception)
    trace.exception(status1, new IllegalStateException()); //depth=0 <- depth=1 (exception)
  }
}
```

- 동작 동작 결과
    
    ```text
    [ed72b67d] hello1
    [ed72b67d] |-->hello2
    [ed72b67d] |<--hello2 time=2ms
    [ed72b67d] hello1 time=6ms
    ```
    
- 비정상 동작 결과
    
    ```text
    [59770788] hello
    [59770788] |-->hello2
    [59770788] |<X-hello2 time=3ms ex=java.lang.IllegalStateException
    [59770788] hello time=8ms ex=java.lang.IllegalStateException
    ```
    

이제 불필요하게 `TraceId` 를 파라미터로 전달하지 않아도 되고, 애플리케이션의 메서드 파라미터도 변경하지 않아도 된다.

<br/>

## 필드 동기화 - 적용

지금까지 만든 `FieldLogTrace` 를 실제 애플리케이션에 적용해보자.

### `LogTraceConfig`

- `FieldLogTrace` 를 수동으로 스프링 빈으로 등록하자.

> 수동으로 등록하면, 나중에 구현체를 편리하게 변경할 수 있다.  
(계속해서 발전시켜나가며 구현체를 변경할 예정)

```java
@Configuration
public class LogTraceConfig {
  @Bean
  public LogTrace logTrace() {
    return new FieldLogTrace();
  }
}
```

### `OrderControllerV3`

```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV3 {

  private final OrderServiceV3 orderService;
  private final LogTrace trace; //인터페이스 사용

  @GetMapping("/v3/request")
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

### `OrderServiceV3`

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV3 {
  private final OrderRepositoryV3 orderRepository;
  private final LogTrace trace; //인터페이스 사용

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

### `OrderRepositoryV3`

```java
@Repository
@RequiredArgsConstructor
public class OrderRepositoryV3 {

  private final LogTrace trace; //인터페이스 사용

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

### 결과

- 정상 실행 결과: `/v3/request?itemId=hello`

  ```text
  [3cfc6878] OrderController.request()
  [3cfc6878] |-->OrderServiceV1.orderItem()
  [3cfc6878] |    |-->OrderRepository.save()
  [3cfc6878] |    |<--OrderRepository.save() time=1005ms
  [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
  [3cfc6878] OrderController.request() time=1014ms
  ```

- 예외 실행 결과: `/v3/request?itemId=ex`

  ```text
  [1c444352] OrderController.request()
  [1c444352] |-->OrderServiceV1.orderItem()
  [1c444352] |    |-->OrderRepository.save()
  [1c444352] |    |<X-OrderRepository.save() time=0ms ex=java.lang.IllegalStateException: 예외 발생!
  [1c444352] |<X-OrderServiceV1.orderItem() time=10ms ex=java.lang.IllegalStateException: 예외 발생!
  [1c444352] OrderController.request() time=13ms ex=java.lang.IllegalStateException: 예외 발생!
  ```

<br/>

## 필드 동기화 - 동시성 문제

위에서 만든 `FieldLogTrace` 를 테스트할 때는 문제가 없는 것처럼 보인다. **사실 `FieldLogTrace` 는 심각한 동시성 문제를 가지고 있다.**

### 동시성 문제 확인

- `/v3/request?itemId=hello` 을 1초 안에 2번 실행해보자.
- 이때 우리가 기대하는 결과는 아래와 같다.
    
    ```text
    [nio-8080-exec-3] [3cfc6878] OrderController.request()
    [nio-8080-exec-3] [3cfc6878] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [123asd45] OrderController.request()
    [nio-8080-exec-4] [123asd45] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [3cfc6878] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-4] [123asd45] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-3] [3cfc6878] OrderController.request() time=1014ms
    [nio-8080-exec-4] [123asd45] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] OrderController.request() time=1014ms
    ```
    
    - `[nio-8080-exec-3]` , `[nio-8080-exec-4]` 는 쓰레드의 번호를 나타낸다.
    - 동시에 여러 사용자가 요청하면, 여러 쓰레드가 동시에 애플리케이션 로직을 호출하게 된다. 따라서 로그가 섞여서 출력된다. 이것을 쓰레드 별로 다시 정리해보면 아래와 같다.
    
    ```text
    [nio-8080-exec-3] [3cfc6878] OrderController.request()
    [nio-8080-exec-3] [3cfc6878] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [3cfc6878] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-3] [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-3] [3cfc6878] OrderController.request() time=1014ms
    
    [nio-8080-exec-4] [123asd45] OrderController.request()
    [nio-8080-exec-4] [123asd45] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [123asd45] |    |-->OrderRepository.save()
    [nio-8080-exec-4] [123asd45] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-4] [123asd45] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] OrderController.request() time=1014ms
    ```
    
    - **이와 같이, 우리는 서로 다른 쓰레드가 로그를 개별적으로 처리하길 기대했다.**
- 그 결과는 아래와 같다.
    
    ```text
    [nio-8080-exec-3] [5c49ff2c] OrderController.request()
    [nio-8080-exec-3] [5c49ff2c] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [5c49ff2c] |    |-->OrderRepository.save()
    [nio-8080-exec-4] [5c49ff2c] |    |    |-->OrderController.request()
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |    |-->OrderRepository.save()
    [nio-8080-exec-3] [5c49ff2c] |    |<--OrderRepository.save() time=1004ms
    [nio-8080-exec-3] [5c49ff2c] |<--OrderServiceV1.orderItem() time=1010ms
    [nio-8080-exec-3] [5c49ff2c] OrderController.request() time=1014ms
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |    |<--OrderRepository.save() time=1002ms
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |<--OrderServiceV1.orderItem() time=1013ms
    [nio-8080-exec-4] [5c49ff2c] |    |    |<--OrderController.request() time=1017ms
    ```
    
    - 이것을 다시 쓰레드별로 정리해보면 아래와 같다.
    
    ```text
    [nio-8080-exec-3] [5c49ff2c] OrderController.request()
    [nio-8080-exec-3] [5c49ff2c] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [5c49ff2c] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [5c49ff2c] |    |<--OrderRepository.save() time=1004ms
    [nio-8080-exec-3] [5c49ff2c] |<--OrderServiceV1.orderItem() time=1010ms
    [nio-8080-exec-3] [5c49ff2c] OrderController.request() time=1014ms
    
    [nio-8080-exec-4] [5c49ff2c] |    |    |-->OrderController.request()
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |    |-->OrderRepository.save()
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |    |<--OrderRepository.save() time=1002ms
    [nio-8080-exec-4] [5c49ff2c] |    |    |    |<--OrderServiceV1.orderItem() time=1013ms
    [nio-8080-exec-4] [5c49ff2c] |    |    |<--OrderController.request() time=1017ms
    ```
    
    - 기대한 것과는 전혀 다르게 출력된다.
    - **서로 다른 쓰레드가 같은 `TraceId` 를 사용하고, `level` 도 많이 이상해졌다.**

### 동시성 문제

- 위에서 살펴본 것과 같은 문제를 동시성 문제라고 한다.
- `FieldLogTrace` 는 싱글톤으로 등록된 스프링 빈이다. 이 객체의 인스턴스가 애플리케이션에 1개만 존재한다는 뜻이다.
- **이렇게 하나만 있는 인스턴스의 `FieldLogTrace.traceIdHolder` 필드를 여러 쓰레드가 동시에 접근하기 때문에 문제가 발생하게 된다.**

<br/>

## 동시성 문제 - 예시

예시를 통해, 동시성 문제가 어떻게 발생하게 되는지 알아보자.

### `build.gradle`

```text
dependencies {
  ...

  //테스트에서 lombok 사용
  testCompileOnly 'org.projectlombok:lombok'
  testAnnotationProcessor 'org.projectlombok:lombok'
}
```

- 테스트 코드에서 `@Slf4j` 를 사용하기 위해 추가하자.

### 테스트코드 `FieldService`

```java
@Slf4j
public class FieldService {

  private String nameStore;

  public String logic(String name) {
    log.info("저장 name={} -> nameStore={}", name, nameStore);
    nameStore = name;
    sleep(1000);
    log.info("조회 nameStore={}", nameStore);
    return nameStore;
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

- 파라미터로 넘어온 `name` 을 필드 `nameStore` 에 저장한다.
- 그리고 1초간 쉰 다음 필드에 저장된 `nameStore` 를 반환한다.

### 테스트코드 `FieldServiceTest`

```java
@Slf4j
public class FieldServiceTest {

  private FieldService fieldService = new FieldService();

  @Test
  void field() {
    log.info("main start");

    //쓰레드에서 수행할 로직 A
    Runnable userA = () -> {
      fieldService.logic("userA");
    };

    //쓰레드에서 수행할 로직 B
    Runnable userB = () -> {
      fieldService.logic("userB");
    };

    //쓰레드 A 생성
    Thread threadA = new Thread(userA);
    threadA.setName("thread-A");

    //쓰레드 B 생성
    Thread threadB = new Thread(userB);
    threadB.setName("thread-B");

    threadA.start(); //쓰레드 A 실행
    sleep(2000); //동시성 문제 발생 X (메인쓰레드가 2초 대기)
//     sleep(100); //동시성 문제 발생 O (메인쓰레드가 0.1초 대기)
    threadB.start(); //쓰레드 B 실행 (메인쓰레드가 대기한 후, 호출되는 코드)

    sleep(3000); //쓰레드 B가 끝날때까지, 메인 쓰레드가 대기
    log.info("main exit");
  }

  private void sleep(long millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

- `sleep(2000)` 덕분에 동시성 문제가 발생하지 않는다.
    - **왜냐하면 `FieldService.logic()` 메서드는 내부에서 `sleep(1000)` 을 호출하여 1초 지연을 하기 때문이다. 따라서 1초 이후에 쓰레드B를 실행하면 정상적으로(순서대로) 실행할 수 있다.**
- 결과
    
    ```text
    [Test worker] main start
    [thread-A] 저장 name=userA -> nameStore=null
    [thread-A] 조회 nameStore=userA
    [thread-B] 저장 name=userB -> nameStore=userA
    [thread-B] 조회 nameStore=userB
    [Test worker] main exit
    ```
    
    1. `thread-A` 는 `userA` 를 `nameStore` 에 저장했다.
    2. `thread-A` 는 `userA` 를 `nameStore` 에서 조회했다.
    3. `thread-B` 는 `userB` 를 `nameStore` 에 저장했다.
    4. `thread-B` 는 `userB` 를 `nameStore` 에서 조회했다.
    
    <br/>
    
    이것을 시각적으로 표현하면 아래와 같다.
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%201.png)
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%202.png)
    

이번에는 동시성 문제가 발생하도록 `FieldServiceTest` 코드를 아래처럼 바꿔보자.

```java
// sleep(2000); //동시성 문제 발생 X (메인쓰레드가 2초 대기)
sleep(100); //동시성 문제 발생 O (메인쓰레드가 0.1초 대기)
```

- 결과
    
    ```text
    [Test worker] main start
    [thread-A] 저장 name=userA -> nameStore=null
    [thread-B] 저장 name=userB -> nameStore=userA
    [thread-A] 조회 nameStore=userB
    [thread-B] 조회 nameStore=userB
    [Test worker] main exit
    ```
    
    - 저장하는 부분은 문제가 없지만, 조회할 때 문제가 생긴다.
    
    <br/>
    
    이것을 시각적으로 표현하면 아래와 같다.
    
    1. `thread-A` 는 `userA` 를 `nameStore` 에 저장했다.
        
        ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%203.png)
        
    2. `thread-B` 는 `userB` 를 `nameStore` 에 저장했다.
        
        ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%204.png)
        
    3. `thread-A` 와 `thread-B` 는 `userB` 를 `nameStore` 에서 조회했다.
        
        ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%205.png)
        
- 정리
    - **결과적으로 `thread-A` 입장에서는 저장한 데이터와 조회한 데이터가 다른 문제가 발생한다.**
    - 이처럼 여러 쓰레드가 동시에 같은 인스턴스의 필드 값을 변경하면서 발생하는 문제를 **동시성 문제**라고 한다.
    - 특히 스프링 빈처럼 싱글톤 객체의 필드를 변경하며 사용할 때, 이러한 동시성 문제를 조심해야 한다.
    - 이러한 문제를 해결할 수 있는 것이 바로, **쓰레드 로컬**이다.

<br/>

## ThreadLocal - 소개

### ThreadLocal 이란?

- **쓰레드 로컬은 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다.**
- 일반적인 변수 필드
    - 여러 쓰레드가 같은 인스턴스의 필드에 접근하면 처음 쓰레드가 보관한 데이터가 사라질 수 있다.
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%206.png)
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%207.png)
    

<br/>

- 쓰레드 로컬
    - **쓰레드 로컬을 사용하면 각 쓰레드마다 별도의 내부 저장소를 제공한다. 따라서 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제없다.**
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%208.png)
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%209.png)
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%2010.png)
    

> 자바는 언어차원에서 쓰레드 로컬을 지원하기 위한 `java.lang.ThreadLocal` 클래스를 제공한다.

<br/>

## ThreadLocal - 예시

테스트 코드를 통해, 쓰레드 로컬 사용방법을 알아보자.

### 테스트 코드 `ThreadLocalService`

```java
@Slf4j
public class ThreadLocalService {

  //쓰레드 로컬 설정
  private ThreadLocal<String> nameStore = new ThreadLocal<>();

  public String logic(String name) {
    log.info("저장 name={} -> nameStore={}", name, nameStore.get());
    nameStore.set(name);
    sleep(1000);
    log.info("조회 nameStore={}", nameStore.get());
    return nameStore.get();
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

- **기존의 `FieldService` 와 거의 같은 코드인데, `nameStore` 필드가 일반 `String` 타입에서 `ThreadLocal` 을 사용하도록 변경되었다.**
- ThreadLocal 사용법
    - 값 저장: `ThreadLocal.set(xxx)`
    - 값 조회: `ThreadLocal.get()`
    - 값 제거: `ThreadLocal.remove()`

> 해당 쓰레드가 쓰레드 로컬을 모두 사용하고 나면, `ThreadLocal.remove()` 를 호출해서 쓰레드 로컬에 저장된 값을 반드시 제거해주어야 한다!!  
자세한 것은 나중에 설명하겠다.

### 테스트 코드 `ThreadLocalServiceTest`

```java
@Slf4j
public class ThreadLocalServiceTest {

  private ThreadLocalService threadLocalService = new ThreadLocalService();

  @Test
  void threadLocal() {
    log.info("main start");

    //쓰레드에서 수행할 로직 A
    Runnable userA = () -> {
      threadLocalService.logic("userA");
    };

    //쓰레드에서 수행할 로직 B
    Runnable userB = () -> {
      threadLocalService.logic("userB");
    };

    //쓰레드 A 생성
    Thread threadA = new Thread(userA);
    threadA.setName("thread-A");

    //쓰레드 B 생성
    Thread threadB = new Thread(userB);
    threadA.setName("thread-B");

    threadA.start(); //쓰레드 A 실행
//    sleep(2000); //동시성 문제 발생 X (메인쓰레드가 2초 대기)
     sleep(100); //동시성 문제 발생 O (메인쓰레드가 0.1초 대기)
    threadB.start(); //쓰레드 B 실행 (메인쓰레드가 대기한 후, 호출되는 코드)

    sleep(3000); //쓰레드 B가 끝날때까지, 메인 쓰레드가 대기
    log.info("main exit");
  }

  private void sleep(long millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

### 실행 결과

```text
[Test worker] main start
[thread-A] 저장 name=userA -> nameStore=null
[Thread-B] 저장 name=userB -> nameStore=null
[thread-A] 조회 nameStore=userA
[Thread-B] 조회 nameStore=userB
[Test worker] main exit
```

- 쓰레드 로컬 덕분에 쓰레드마다 각각 별도의 데이터 저장소를 가지게 되었고, 결과적으로 동시성 문제도 해결되었다!

<br/>

## 쓰레드 로컬 동기화 - 개발

- `FieldLogTrace` 에서 발생했던 동시성 문제를 `ThreadLocal` 로 해결할 수 있다.
- **`TraceId traceIdHolder` 필드를 `ThreadLocal<TraceId> traceIdHolder` 로 변경하여, 쓰레드 로컬을 사용하도록 하면 된다.**

### `ThreadLocalLogTrace`

```java
@Slf4j
public class ThreadLocalLogTrace implements LogTrace {
  private static final String START_PREFIX = "-->";
  private static final String COMPLETE_PREFIX = "<--";
  private static final String EX_PREFIX = "<X-";

//  private TraceId traceIdHolder; //traceId 동기화, 동시성 이슈 발생, traceId를 계속 들고 있는다.
  private ThreadLocal<TraceId> traceIdHolder = new ThreadLocal<>();

  @Override
  public TraceStatus begin(String message) {
    syncTraceId(); //traceId를 동기화한다.

    TraceId traceId = traceIdHolder.get(); //쓰레드 로컬에서 값을 가져온다.
    Long startTimeMs = System.currentTimeMillis();
    log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);

    return new TraceStatus(traceId, startTimeMs, message);
  }

  @Override
  public void end(TraceStatus status) {
    complete(status, null);
  }

  @Override
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

    releaseTraceId(); //동기화를 해제할 수 있다면, 해제한다.
  }

  private void syncTraceId() {
    TraceId traceId = traceIdHolder.get(); //쓰레드 로컬에서 값을 가져온다.
    if (traceId == null) {
      traceIdHolder.set(new TraceId()); //쓰레드 로컬에 값을 저장한다.
    } else {
      traceIdHolder.set(traceId.createNextId()); //쓰레드 로컬에 값을 저장한다.
    }
  }

  private void releaseTraceId() {
    TraceId traceId = traceIdHolder.get(); //쓰레드 로컬에서 값을 가져온다.
    if (traceId.isFirstLevel()) {
      traceIdHolder.remove(); //쓰레드 로컬에 저장된 값을 제거한다.
    } else {
      traceIdHolder.set(traceId.createPreviousId()); //쓰레드 로컬에 값을 저장한다.
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

- 쓰레드 로컬을 사용하도록 변경했다.
- `ThreadLocal.remove()`
    - **추가로 쓰레드 로컬을 모두 사용하고 나면, 반드시 `ThreadLocal.remove()` 를 호출해서 쓰레드 로컬에 저장된 값을 제거해주어야 한다.**
    
    > 로그를 찍는 경우엔, 마지막 로그를 출력하고 나면 쓰레드 로컬의 값 제거
    
    ```text
    [3cfc6878] OrderController.request() //쓰레드 로컬에 값(TraceId) 할당
    [3cfc6878] |-->OrderServiceV1.orderItem()
    [3cfc6878] |    |-->OrderRepository.save()
    [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [3cfc6878] OrderController.request() time=1014ms //이때 쓰레드 로컬 값 제거
    ```
    
<br/>

## 쓰레드 로컬 동기화 - 적용

### `LogTraceConfig` 수정

```java
@Configuration
public class LogTraceConfig {
  @Bean
  public LogTrace logTrace() {
    //return new FieldLogTrace();
    return new ThreadLocalLogTrace();
  }
}
```

- 컨테이너에 등록할 빈 객체를 바꾸기만 하면 된다.

### 결과

- 정상 실행 결과: `/v3/request?itemId=hello`
    
    ```text
    [3cfc6878] OrderController.request()
    [3cfc6878] |-->OrderServiceV1.orderItem()
    [3cfc6878] |    |-->OrderRepository.save()
    [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [3cfc6878] OrderController.request() time=1014ms
    ```
    
- 예외 실행 결과: `/v3/request?itemId=ex`
    
    ```text
    [1c444352] OrderController.request()
    [1c444352] |-->OrderServiceV1.orderItem()
    [1c444352] |    |-->OrderRepository.save()
    [1c444352] |    |<X-OrderRepository.save() time=0ms ex=java.lang.IllegalStateException: 예외 발생!
    [1c444352] |<X-OrderServiceV1.orderItem() time=10ms ex=java.lang.IllegalStateException: 예외 발생!
    [1c444352] OrderController.request() time=13ms ex=java.lang.IllegalStateException: 예외 발생!
    ```
    
- 동시 요청 결과: `/v3/request?itemId=hello` 연속 2번 실행
    
    ```text
    [nio-8080-exec-3] [3cfc6878] OrderController.request()
    [nio-8080-exec-3] [3cfc6878] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [123asd45] OrderController.request()
    [nio-8080-exec-4] [123asd45] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [3cfc6878] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-4] [123asd45] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-3] [3cfc6878] OrderController.request() time=1014ms
    [nio-8080-exec-4] [123asd45] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] OrderController.request() time=1014ms
    ```
    
    쓰레드 별로 로그를 분리해보면 아래와 같다.
    
    ```text
    [nio-8080-exec-3] [3cfc6878] OrderController.request()
    [nio-8080-exec-3] [3cfc6878] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-3] [3cfc6878] |    |-->OrderRepository.save()
    [nio-8080-exec-3] [3cfc6878] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-3] [3cfc6878] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-3] [3cfc6878] OrderController.request() time=1014ms
    
    [nio-8080-exec-4] [123asd45] OrderController.request()
    [nio-8080-exec-4] [123asd45] |-->OrderServiceV1.orderItem()
    [nio-8080-exec-4] [123asd45] |    |-->OrderRepository.save()
    [nio-8080-exec-4] [123asd45] |    |<--OrderRepository.save() time=1005ms
    [nio-8080-exec-4] [123asd45] |<--OrderServiceV1.orderItem() time=1009ms
    [nio-8080-exec-4] [123asd45] OrderController.request() time=1014ms
    ```
    
    - 드디어 정상적으로 동작하게 된다!

<br/>

## 쓰레드 로컬 - 주의사항

**WAS(톰캣)처럼 쓰레드 풀을 사용하는 경우, 쓰레드 로컬의 값을 사용하고 제거하지 않고 내버려두면 심각한 문제가 발생할 수 있다.**

아래는 그 예시이다.

- 사용자A 저장 요청
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%2011.png)
    
    1. 사용자A가 저장 HTTP를 요청했다.
    2. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다.
    3. 쓰레드 `thread-A` 가 할당되었다.
    4. `thread-A` 는 사용자A의 데이터를 쓰레드 로컬에 저장한다.
    5. 쓰레드 로컬의 `thread-A` 전용 보관소에 데이터를 보관한다.
- 사용자A 저장 요청 종료
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%2012.png)
    
    1. 사용자A의 HTTP 응답이 끝난다.
    2. WAS는 사용이 끝난 `thread-A` 를 쓰레드 풀에 반환한다.
        
        > 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고, 보통 쓰레드 풀을 통해서 쓰레드를 재사용한다.
    3. `thread-A` 는 쓰레드 풀에 아직 살아있다. **따라서 쓰레드 로컬의 `thread-A` 전용 보관소에 사용자A의 데이터도 함께 살아있게 된다.**
- 사용자B 조회 요청
    
    ![Untitled](/assets/img/2022-07-21-SPRING_Advanced_ThreadLocal/Untitled%2013.png)
    
    1. 사용자B가 조회를 위한 새로운 HTTP 요청을 한다.
    2. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다.
    3. 쓰레드 `thread-A` 가 할당되었다. (물론 다른 쓰레드가 할당될 수 도 있다.)
    4. 이번에는 조회하는 요청이다. `thread-A` 는 쓰레드 로컬에서 데이터를 조회한다.
    5. **쓰레드 로컬은 `thread-A` 전용 보관소에 있는 사용자A의 데이터를 반환한다.**
    6. **결과적으로 사용자A의 값이 반환된다. 따라서 사용자B는 사용자A의 정보를 조회하게 된다.**
- 정리
    - 결과적으로 사용자B는 사용자A의 데이터를 확인하게 되는 심각한 문제가 발생하게 된다.
    - **이런 문제를 예방하려면 사용자A의 요청이 끝날때 쓰레드 로컬의 값을 `ThreadLocal.remove()` 로 꼭 제거해야 한다.**

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*