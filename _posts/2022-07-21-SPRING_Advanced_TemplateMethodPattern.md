---
category: Spring-ADV
tags: [스프링]
title: "[스프링-ADV] 템플릿 메서드 패턴"
date:   2022-07-21 22:30:00 
lastmod : 2022-07-21 22:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 템플릿 메서드 패턴

## 템플릿 메서드 패턴 개요

- 이전 포스팅에서 쓰레드 로컬을 사용하여, 매개변수를 넘기는 수고를 덜 수 있는 로그 추적기를 만들었다.
    - 이전 게시글: [[스프링-ADV] 쓰레드 로컬](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_ThreadLocal)
    
    > 꼭 이전 글을 참고한 뒤, 본 글을 읽자.

### 기존 로그 추적기의 문제점

- 이전 포스팅에서 요구사항을 만족하는 로그 추적기를 잘 만들었다. 또한 파라미터를 넘기는 불편함까지 제거했다.
    - [요구사항 참고 링크](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_Project_Initialize#11)
- 하지만 여전히 불편한 점이 있다. 로그 추적기 도입 전과 도입 후의 코드를 비교해보자.
    - 로그 추적기 도입 전 - `OrderControllerV0` , `OrderServiceV0`
        
        ```java
        //OrderControllerV0 코드
        @GetMapping("/v0/request")
        public String request(String itemId) {
          orderService.orderItem(itemId);
          return "ok";
        }
        ```
        
        ```java
        //OrderServiceV0 코드
        public void orderItem(String itemId) {
            orderRepository.save(itemId);
        }
        ```
        
    - 로그 추적기 도입 후 - `OrderControllerV3` , `OrderServiceV3`
        
        ```java
        //OrderControllerV3 코드
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
        ```
        
        ```java
        //OrderServiceV3 코드
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
        ```
        
    - `OrderControllerV0` 과 `OrderServiceV0` 은 해당 메서드가 실제 처리해야 하는 핵심 기능만 깔끔하게 남아있다.
    - **반면에 `OrderControllerV3` 과 `OrderServiceV3` 은 핵심 기능보다 로그를 출력해야 하는 부가 기능 코드가 훨씬 더 많고 복잡하다.**

### 핵심 기능 vs 부가 기능

앞으로 코드를 설명할 때 핵심 기능과 부가 기능으로 구분해서 설명하겠다.

- 핵심 기능
    - 해당 객체가 제공하는 고유의 기능(핵심로직)이다.
    - 예시: `OrderService` 의 핵심 기능은 레포지토리 클래스의 메서드를 호출하는 것이다.
- 부가 기능
    - 핵심 기능을 보조하기 위해 제공되는 기능이다.
    - 서브로직이라고도 한다.
    - 예시: 로깅

`OrderControllerV0` , `OrderServiceV0` 에는 핵심 기능만 있지만, `OrderControllerV3` , `OrderServiceV3` 에는 핵심 기능과 부가 기능이 함께 섞여 있다.

**즉, `OrderControllerV3` , `OrderServiceV3` 에는 핵심 기능 코드보다 부가 기능 코드가 더 많다. 이 문제를 해결해보자.**

### 부가 기능의 패턴

`OrderControllerV3` , `OrderServiceV3` , `OrderRepositoryV3` 를 유심히 살펴보면 아래와 같이 동일한 패턴이 있다.

```java
TraceStatus status = null;
try {
	status = trace.begin("message");
	
	//핵심 기능 호출
	
	trace.end(status);
} catch (Exception e) {
	trace.exception(status, e);
	throw e;
}
```

- **다시 말해, 로그 추적기를 사용하는 구조는 모두 동일하다. 중간에 핵심 기능을 사용하는 코드만 다를 뿐이다.**
- 코드가 중복되는 부분을 따로 메서드로 뽑아내서 사용하게끔 한다면 해결이 될까?
    - **그렇지 않다. `try-catch` 구문이 존재하고, 핵심 기능 부분이 중간에 있어서 단순하게 메서드로 추출하는 것은 어렵다.**

### 변하는 것과 변하지 않는 것을 분리

- 좋은 설계는 **변하는 것과 변하지 않는 것을 분리하는 것**이다.
- 핵심 기능 = 변하는 것
- 부가 기능 (로그 추적기) = 변하지 않는 것

**템플릿 메서드 패턴을 통해 이 문제를 해결할 수 있다.**

<br/>

## 템플릿 메서드 패턴 - 예시1

예시 코드를 통해, 템플릿 메서드 패턴을 이해해보자.

### 테스트 코드 `TemplateMethodTest`

```java
@Slf4j
public class TemplateMethodTest {

  @Test
  void templateMethodV0() {
    logic1();
    logic2();
  }

  private void logic1() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    log.info("비즈니스 로직1 실행");
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }

  private void logic2() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    log.info("비즈니스 로직2 실행");
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }
}
```

- 실행 결과는 아래와 같다.
    
    ```text
    비즈니스 로직1 실행
    resultTime=3
    비즈니스 로직2 실행
    resultTime=0
    ```
    
- `logic1()` 과 `logic2()` 는 시간을 측정하는 부분과 비즈니스 로직을 실행하는 부분이 함께 존재한다.
    - 변하는 부분: 비즈니스 로직
    - 변하지 않는 부분: 시간 측정

이제 템플릿 메서드 패턴을 사용해서 변하는 부분과 변하지 않는 부분을 분리해보자.

<br/>

## 템플릿 메서드 패턴 - 예시2

- 템플릿 메서드 패턴은 이름 그대로 템플릿을 사용하는 방식이다.
    - 템플릿은 기준이 되는 거대한 틀이다.
- 템플릿이라는 틀에 변하지 않는 부분을 몰아둔다. 그리고 일부 변하는 부분을 별도로 호출해서 해결한다.

### 템플릿 메서드 패턴 구조 그림

아래 그림과 같은 구조를 갖는 예시 코드를 작성해보자.

![Untitled](/assets/img/2022-07-21-SPRING_Advanced_TemplateMethodPattern/Untitled%2014.png)

### 테스트 코드 `AbstractTemplate` 추상 클래스

```java
@Slf4j
public abstract class AbstractTemplate {

  /**
   * 구체 메서드 - 실행 메서드
   */
  public void execute() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    call(); //상속
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }

  /**
   * 추상 메서드
   */
  protected abstract void call();

}
```

- **변하지 않는 부분인 시간 측정 로직을 `execute()` 메서드에 몰아두었다. 이제 이것이 하나의 템플릿이 된다.**
- **그리고 템플릿 안에서 변하는 부분은 `call()` 메서드를 호출해서 처리한다.**

![Untitled](/assets/img/2022-07-21-SPRING_Advanced_TemplateMethodPattern/Untitled%2015.png)

- 클래스 별 코드 유형을 정리하면 아래와 같다.
    - 부모 클래스 (추상 클래스) : 변하지 않는 코드
    - 자식 클래스 : 변하는 코드

계속해서 자식 클래스를 살펴보자.

### 테스트 코드 `SubClassLogic1` 구현 클래스

```java
@Slf4j
public class SubClassLogic1 extends AbstractTemplate {
  @Override
  protected void call() {
    log.info("비즈니스 로직1 실행");
  }
}
```

- 템플릿이 호출하는 대상인 `call()` 메서드를 오버라이딩한다.

### 테스트 코드 `SubClassLogic2` 구현 클래스

```java
@Slf4j
public class SubClassLogic2 extends AbstractTemplate {
  @Override
  protected void call() {
    log.info("비즈니스 로직2 실행");
  }
}
```

- 이 클래스 역시 `call()` 메서드를 오버라이딩한다.

### 테스트 코드 `TemplateMethodTest` 에 메서드 추가

```java
/**
 * 템플릿 메서드 패턴 적용
 */
@Test
void templateMethodV1() {
  AbstractTemplate template1 = new SubClassLogic1();
  template1.execute();

  AbstractTemplate template2 = new SubClassLogic2();
  template2.execute();
}
```

- 실행 결과는 아래와 같다.
    
    ```text
    비즈니스 로직1 실행
    resultTime=3
    비즈니스 로직2 실행
    resultTime=1
    ```
    

### 템플릿 메서드 패턴 인스턴스 호출 그림

![Untitled](/assets/img/2022-07-21-SPRING_Advanced_TemplateMethodPattern/Untitled%2016.png)

- 클라이언트(이 경우, 테스트 메서드)에서 `template1.execute()` 를 호출하면, 템플릿 로직인 `AbstractTemplate.execute()` 를 실행한다.
- **여기서 중간에 `call()` 메서드를 호출하는데, 이 부분이 오버라이딩 되어있다.**
- **따라서 현재 인스턴스인 `SubClassLogic1` 인스턴스의 `SubClassLogic1.call()` 메서드가 호출된다.**

<br/>

## 템플릿 메서드 패턴 - 예시 3

### 익명 내부 클래스 사용하기

- 템플릿 메서드 패턴은 `SubClassLogic1` , `SubClassLogic2` 처럼 클래스를 계속 만들어야 하는 단점이 있다.
- **익명 내부 클래스를 사용하면 이런 단점을 보완할 수 있다.**

> 익명 내부 클래스에 대한 자세한 내용은 자바 기본 문법을 따로 찾아보길 바란다.

### 테스트 코드 `TemplateMethodTest` 에 메서드 추가

```java
/**
 * 템플릿 메서드 패턴, 익명 내부 클래스 사용
 */
@Test
void templateMethodV2() {
  //익명 내부 클래스
  AbstractTemplate template1 = new AbstractTemplate() {
    @Override
    protected void call() {
      log.info("비즈니스 로직1 실행");
    }
  };
  log.info("클래스 이름1 = {}", template1.getClass());
  template1.execute();

  //익명 내부 클래스
  AbstractTemplate template2 = new AbstractTemplate() {
    @Override
    protected void call() {
      log.info("비즈니스 로직2 실행");
    }
  };
  log.info("클래스 이름2 = {}", template2.getClass());
  template2.execute();
}
```

- 실행 결과는 아래와 같다.
    
    ```text
    클래스 이름1 = class hello.advanced.trace.template.TemplateMethodTest$1
    비즈니스 로직1 실행
    resultTime=0
    클래스 이름2 = class hello.advanced.trace.template.TemplateMethodTest$2
    비즈니스 로직2 실행
    resultTime=2
    ```
    
<br/>

## 템플릿 메서드 패턴 - 적용

이제 우리가 만든 애플리케이션의 로그 추적기 로직에 템플릿 메서드 패턴을 적용해보자.

### `AbstractTemplate` 추상 클래스

```java
public abstract class AbstractTemplate<T> {

  private final LogTrace trace;

  public AbstractTemplate(LogTrace trace) {
    this.trace = trace;
  }

  /**
   * 템플릿
   */
  public T execute(String message) {
    TraceStatus status = null;
    try {
      status = trace.begin(message);

      //로직 호출
      T result = call();

      trace.end(status);
      return result;
    } catch (Exception e) {
      trace.exception(status, e);
      throw e;
    }
  }

  protected abstract T call();
}
```

- `AbstractTemplate` 추상 클래스는 템플릿 메서드 패턴에서 부모 클래스이며, 템플릿 역할을 한다.
- `<T>` 제네릭을 통해 반환 타입을 정의한다.
- `abstract T call()` 은 변하는 부분을 처리하는 메서드이다. 이 부분은 상속으로 구현해야 한다.

### `OrderControllerV4`

```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV4 {

  private final OrderServiceV4 orderService;
  private final LogTrace trace;

  @GetMapping("/v4/request")
  public String request(String itemId) {

    AbstractTemplate<String> template = new AbstractTemplate<String>(trace) {
      @Override
      protected String call() {
        orderService.orderItem(itemId);
        return "ok";
      }
    };

    return template.execute("OrderController.request()");
  }
}
```

- `AbstractTemplate<String>`
    - 제네릭을 `String` 으로 설정했다. 따라서 `AbstractTemplate` 의 반환 타입은 `String` 이 된다.
- 익명 내부 클래스
    - 객체를 생성하면서 `AbstractTemplate` 을 상속받은 자식 클래스를 정의했다.
    - 따라서 별도의 자식 클래스를 직접 만들지 않아도 된다.
- `template.execute("OrderController.request()")`
    - 템플릿을 실행하면서 로그로 남길 `message` 를 전달한다.

### `OrderServiceV4`

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV4 {
  private final OrderRepositoryV4 orderRepository;
  private final LogTrace trace;

  public void orderItem(String itemId) {
    AbstractTemplate<Void> template = new AbstractTemplate<Void>(trace) {
      @Override
      protected Void call() {
        orderRepository.save(itemId);
        return null;
      }
    };

    template.execute("OrderService.orderItem()");
  }

}
```

> 제네릭에서 반환 타입이 필요한데, 반환할 내용이 없으면 `Void` 타입을 사용하고 `null` 을 반환하면 된다.

### `OrderRepositoryV4`

```java
@Repository
@RequiredArgsConstructor
public class OrderRepositoryV4 {

  private final LogTrace trace;

  public void save(String itemId) {
    AbstractTemplate<Void> template = new AbstractTemplate<Void>(trace) {
      @Override
      protected Void call() {
        if (itemId.equals("ex")) {
          throw new IllegalStateException("예외 발생!");
        }
        sleep(1000);
        return null;
      }
    };

    template.execute("OrderRepository.save()");
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

- 정상 실행 결과: `/v4/request?itemId=hello`
    
    ```text
    [aaaaaaaa] OrderController.request()
    [aaaaaaaa] |-->OrderService.orderItem()
    [aaaaaaaa] |   |-->OrderRepository.save()
    [aaaaaaaa] |   |<--OrderRepository.save() time=1004ms
    [aaaaaaaa] |<--OrderService.orderItem() time=1006ms
    [aaaaaaaa] OrderController.request() time=1007ms
    ```
    
<br/>

## 이것이 더 나은 구조인 이유

### 좋은 설계란?

- 좋은 설계란, **변경**이 일어날 때 자연스럽게 드러난다.
- **만약 로그를 남기는 로직을 변경해야 해서, `AbstractTemplate` 코드를 변경해야 한다고 해보자.**  
**이땐 단순히 `AbstractTemplate` 코드만 변경하면 된다.**
    - 지금까지 로그를 남기는 부분을 모아서 하나로 모듈화하고, 비즈니스 로직 부분을 분리했기 때문이다.
- 만약 템플릿이 없는 `V3` 상태에서 로그를 남기는 로직을 변경해야 한다고 할 때, 모든 클래스를 다 찾아서 고쳐야 한다.
    - `V3` 에 대해서는 이전 게시글을 참고하자. [[스프링-ADV] 쓰레드 로컬](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_ThreadLocal#6)

### 단일 책임 원칙 (SRP)

- `V4` 는 **로그를 남기는 부분에 단일 책임 원칙을 지킨 것**이다. 변경 지점을 하나로 모아서 변경에 쉽게 대처할 수 있는 구조를 만든 것이다.
    - 이전 `V3` 상태에서는 `컨트롤러` , `서비스` , `레포지토리` 가 ‘핵심 기능(비즈니스 로직)’과 ‘부가 기능(로깅)’을 모두 가지고 있었다. 따라서 SRP를 위배한다.

<br/>

## 템플릿 메서드 패턴 - 정의

### GOF 디자인 패턴에서의 정의

“작업에서 알고리즘의 골격을 정의하고 일분 단계를 하위 클래스로 연기합니다. 템플릿 메서드를 사용하면 하위 클래스가 알고리즘의 구조를 변경하지 않고도 알고리즘의 특정 단계를 재정의할 수 있습니다.” [GOF]

![Untitled](/assets/img/2022-07-21-SPRING_Advanced_TemplateMethodPattern/Untitled%2017.png)

- 풀어서 말하면 아래와 같다.
    - 부모 클래스: 알고리즘의 골격인 템플릿을 정의한다.
    - 자식 클래스: 일부 변경되는 로직을 정의한다.
- 이렇게 하면 자식 클래스가 알고리즘의 전체 구조를 변경하지 않고, 특정 부분만 재정의할 수 있다.
- **결국 상속과 오버라이딩을 통한 다형성으로 문제를 해결하는 것이다.**

### 템플릿 메서드 패턴의 단점

- 템플릿 메서드 패턴은 상속을 사용하기 때문에, **상속에서 오는 단점들을 그대로 안고간다.**
    - **특히 자식 클래스가 부모 클래스와 컴파일 시점에 강하게 결합되는 문제가 있다.**
    
    > 자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않는데도 불구하고 말이다.
- 자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않는데, 부모 클래스를 알아야 한다. 이것은 좋은 설계가 아니다.  
그리고 이런 잘못된 의존관계 때문에 부모 클래스를 수정하면, 자식 클래스에도 영향을 줄수 있다.
    - 예를 들어, 부모 클래스(`AbstractTemplate`)에 새로운 추상 메서드를 추가했다고 해보자. 이때 모든 구현 클래스들이 그 메서드를 구현해주어야 한다.
- 추가적으로 템플릿 메서드 패턴은 상속 구조를 사용하기 때문에, **별도의 클래스나 익명 내부 클래스를 만들어야 하는 부분도 복잡하다.**

템플릿 메서드 패턴과 비슷한 역할을 하면서 상속의 단점을 제거할 수 있는 디자인 패턴이 바로 “전략 패턴"이다.

전략 패턴은 다음 포스팅에서 다룬다.

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*