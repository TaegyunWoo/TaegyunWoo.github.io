---
category: Spring-ADV
tags: [스프링]
title: "[스프링-ADV] 전략 패턴"
date:   2022-07-22 16:10:00 
lastmod : 2022-07-22 16:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 전략 패턴

## 전략 패턴 - 시작

### 개요

- 이전 게시글에서 템플릿 메서드 패턴에 대해 다뤄보았다.
    - [[스프링-ADV] 템플릿 메서드 패턴](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_TemplateMethodPattern)
- 템플릿 메서드 패턴이 갖는 문제점은 아래와 같다.
    - 상속을 사용하기 때문에 자식 클래스와 부모 클래스가 강하게 결합된다.
    - 또한 별도의 클래스나 익명 내부 클래스를 만들어야 한다.
- 이러한 문제를 해결하기 위해 전략패턴을 사용할 수 있다.

### 예시 코드(테스트 코드 `ContextV1Test`)

- 템플릿 메서드 패턴을 통해 개선했던 예시 코드를 다시 살펴보자. 이것을 전략패턴으로 개선해볼 것이다.

> 이전(템플릿 메서드 패턴)에 사용했던 예시와 같다.

```java
@Slf4j
public class ContextV1Test {
  @Test
  void strategyV0() {
    logic1();
    logic2();
  }

  private void logic1() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    log.info("비즈니스 로직1 실행");
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }

  private void logic2() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    log.info("비즈니스 로직2 실행");
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }
}
```

- 실행 결과는 아래와 같다.
    
    ```text
    비즈니스 로직1 실행
    resultTime=4
    비즈니스 로직2 실행
    resultTime=0
    ```
    
<br/>

## GOF 전략 패턴

### 전략 패턴 vs 템플릿 메서드 패턴

- 동일한 문제를 전략 패턴을 사용해서 해결해보자.
- 템플릿 메서드 패턴에서의 해결 방법
    - 부모 클래스: 변하지 않는 템플릿을 작성
    - 자식 클래스: 변하는 부분을 작성
- 전략 패턴에서의 해결 방법
    - `Context` : 변하지 않는 부분을 작성
    - `Strategy` 인터페이스 : 변하는 부분이 구현하도록 작성
- **즉, 전략 패턴에서는 상속(`extends`)이 아니라, 위임(`implements`)으로 문제를 해결한다.**
    - `Context` 가 변하지 않는 템플릿 역할을 한다.
    - `Strategy` 가 변하는 알고리즘 역할을 한다.

### GOF 디자인 패턴에서 정의한 전략 패턴의 의도

“알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.”

![Untitled](/assets/img/2022-07-22-SPRING_Advanced_StrategyPattern/Untitled.png)

<br/>

## 전략 패턴 - 예시 1

### `Strategy` 인터페이스

```java
public interface Strategy {
  void call();
}
```

- 이 인터페이스는 변하는 알고리즘 역할을 한다.

### 테스트 코드 `StrategyLogic1`

```java
@Slf4j
public class StrategyLogic1 implements Strategy {
  @Override
  public void call() {
    log.info("비즈니스 로직1 실행");
  }
}
```

- 변하는 알고리즘은 `Strategy` 인터페이스를 구현하면 된다.
- 여기서는 비즈니스 로직 1을 구현했다.

### 테스트 코드 `StrategyLogic2`

```java
@Slf4j
public class StrategyLogic2 implements Strategy {
  @Override
  public void call() {
    log.info("비즈니스 로직2 실행");
  }
}
```

- 여기서는 비즈니스 로직 2을 구현했다.

### 테스트 코드 `ContextV1`

```java
@Slf4j
public class ContextV1 {
  private Strategy strategy;

  public ContextV1(Strategy strategy) {
    this.strategy = strategy;
  }

  public void execute() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    strategy.call(); //위임
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }
}
```

- `ContextV1` 은 변하지 않는 로직을 가지고 있는 템플릿 역할을 하는 코드이다.
    - 전략 패턴에서는 이것을 컨텍스트(문맥)이라고 한다.
    - **컨텍스트(문맥)는 크게 변하지 않지만, 그 문맥 속에서 `strategy` 를 통해 일부 전략이 변경된다고 생각하자.**
- `Context` 는 내부에 `Strategy strategy` 필드를 가지고 있다.
    - 이 필드에 변하는 부분인 `strategy` 의 구현체를 주입하면 된다.
- 전략 패턴의 핵심
    - **`Context` 는 `Strategy` 인터페이스에만 의존한다. 덕분에 `Strategy` 의 구현체를 변경하거나 새로 만들어도 `Context` 코드에는 영향을 주지 않는다.**
- **이 전략 패턴이 바로 “스프링에서 의존관계 주입으로 사용하는 방식이다.**

### 테스트 코드 `ContextV1Test` 에 메서드 추가

```java
@Test
void strategyV1() {
  Strategy strategyLogic1 = new StrategyLogic1(); //변하는 부분
  ContextV1 context1 = new ContextV1(strategyLogic1); //'변하는 부분' 주입
  context1.execute(); //실행

  Strategy strategyLogic2 = new StrategyLogic2(); //변하는 부분
  ContextV1 context2 = new ContextV1(strategyLogic2); //'변하는 부분' 주입
  context2.execute(); //실행
}
```

- 의존관계 주입을 통해, `ContextV1` 에 `Strategy` 의 구현체인 `strategyLogic1` 을 주입한다.
    - 이렇게 해서 `Context` 안에 원하는 전략을 주입한다.
- 실행 결과는 아래와 같다.
    
    ```text
    비즈니스 로직1 실행
    resultTime=4
    비즈니스 로직2 실행
    resultTime=0
    ```
    

### 전략 패턴 실행 그림

![Untitled](/assets/img/2022-07-22-SPRING_Advanced_StrategyPattern/Untitled%201.png)

1. 클라이언트(여기에선 테스트 메서드 `strategyV1()`)가 `Context` 에 원하는 `Strategy` 구현체를 주입한다.
2. 클라이언트가 `context` 의 `execute()` 메서드를 호출하여 실행한다. 
3. `context` 는 `context` 로직을 시작한다.
4. **`context` 로직 중간에 `strategy.call()` 을 호출해서 주입 받은 `strategy` 로직을 실행한다.**
5. `context` 는 나머지 로직을 실행한다.

### 템플릿 메서드 패턴과 전략 패턴의 차이점

얼핏 보기엔, 템플릿 메서드 패턴과 전략 패턴 간의 차이점이 명확하게 안보일 수 있다.

아래 그림을 통해 정리해보자.

![Untitled](/assets/img/2022-07-22-SPRING_Advanced_StrategyPattern/Untitled%202.png)

- 템플릿 메서드 패턴
    - 추상 클래스에 포함되는 것
        - 변하지 않는 부분을 구현한 메서드
        - 변하는 부분은 구현되지 않은 추상 메서드
    - 구현 클래스에 포함되는 것
        - 추상 클래스를 상속받아, 추상 메서드를 오버라이딩하여 변하는 부분 구현
- 전략 패턴
    - `Context` 에 포함되는 것
        - 변하지 않는 부분을 구현한 메서드
        - **단, 변하는 부분을 실행해야 할 땐 인터페이스(`Strategy`)에 위임**
    - `Strategy` 구현 클래스에 포함되는 것
        - `Strategy` 인터페이스를 구현하여 변하는 부분 작성

<br/>

## 전략 패턴 - 예시 2

전략 패턴도 익명 내부 클래스를 사용할 수 있다.

### 테스트 코드 `ContextV1Test` 에 메서드 추가 1

```java
@Test
void strategyV2() {
  //익명 내부 클래스로 변하는 부분 구현
  Strategy strategyLogic1 = new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직1 실행");
    }
  };
  log.info("strategyLogic1={}", strategyLogic1.getClass());
  ContextV1 context1 = new ContextV1(strategyLogic1); //익명 내부 클래스로 구현한 변하는 부분 주입
  context1.execute();

  //익명 내부 클래스로 변하는 부분 구현
  Strategy strategyLogic2 = new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직2 실행");
    }
  };
  log.info("strategyLogic2={}", strategyLogic2.getClass());
  ContextV1 context2 = new ContextV1(strategyLogic2); //익명 내부 클래스로 구현한 변하는 부분 주입
  context2.execute();
}
```

- 직접 클래스를 만들어서 `strategy` 를 구현한 것과는 다르게, 익명 내부 클래스를 만들어서 구현하였다.
- 실행 결과는 아래와 같다.
    
    ```text
    strategyLogic1=class hello.advanced.trace.strategy.ContextV1Test$1
    비즈니스 로직1 실행
    resultTime=1
    strategyLogic2=class hello.advanced.trace.strategy.ContextV1Test$2
    비즈니스 로직2 실행
    resultTime=0
    ```
    
    - `ContextV1Test$1` , `ContextV1Test$2` 와 같이 익명 내부 클래스가 생성되었다.

### 테스트 코드 `ContextV1Test` 에 메서드 추가 2

아예 익명 내부 클래스를 바로 `Context` 생성자에 전달하여, 좀 더 간략하게 표현할 수 있다.

```java
@Test
void strategyV3() {
  //익명 내부 클래스를 생성자에 바로 전달함
  ContextV1 context1 = new ContextV1(new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직1 실행");
    }
  });
  context1.execute();

  //익명 내부 클래스를 생성자에 바로 전달함
  ContextV1 context2 = new ContextV1(new Strategy() {
    @Override
    public void call() {
      log.info("비즈니스 로직2 실행");
    }
  });
  context2.execute();
}
```

### 테스트 코드 `ContextV1Test` 에 메서드 추가 3

람다식을 사용하면 훨씬 더 간결해진다.

```java
@Test
void strategyV4() {
  ContextV1 context1 = new ContextV1(() -> log.info("비즈니스 로직1 실행"));
  context1.execute();

  ContextV1 context2 = new ContextV1(() -> log.info("비즈니스 로직2 실행"));
  context2.execute();
}
```

> 람다로 변경하려면, 인터페이스에 메서드가 1개만 있으면 된다.

### 정리

- 지금까지 **일반적으로** 이야기하는 전략 패턴에 대해 알아보았다.
- 변하지 않는 부분을 `Context` 에 두고, 변하는 부분을 `Strategy` 를 구현해서 만든다. 그리고 `Context` 의 내부 필드에 `Strategy` 를 주입해서 사용했다.

### 선 조립, 후 실행

- **이렇게 `Context` 의 내부 필드에 `Strategy` 를 두고 사용하면, 선 조립 후 실행 방식에서 유용하다.**
    - `Context` 와 `Strategy` 를 한번 조립하고 나면 이후로는 `Context` 를 실행하기만 하면 된다.
    - 스프링으로 애플리케이션을 개발할 때, 애플리케이션 로딩 시점에 의존관계 주입을 하여 필요한 의존관계를 모두 맺어두고 난 다음, 실제 요청을 처리하는 것과 같은 원리이다.
- **이 방식의 단점은 `Context` 와 `Strategy` 를 조립한 이후에는 전략을 변경하기가 번거롭다는 것이다.**
    - `Context` 에 `setter` 메서드를 두고 `strategy` 를 받아와서 변경하게 된다면, `Context` 를 싱글톤으로 두고 사용할 때 동시성 이슈가 발생할 수 있다.

따라서 보다 유연한 방식으로 전략 패턴을 구현해보자.

<br/>

## 전략 패턴 - 예시 3

이번에는 전략을 실행할 때 직접 파라미터로 전달해서 사용해보자.

### 테스트 코드 `ContextV2`

```java
@Slf4j
public class ContextV2 {

  /**
   * 전략을 파라미터로 전달 받는 방식
   */
  public void execute(Strategy strategy) {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    strategy.call(); //위임
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }

}
```

- `ContextV2` 는 전략을 필드로 가지지 않는다.
    - 대신에 전략을 `execute(..)` 가 호출될 때마다 항상 파라미터로 전달받는다.

### 테스트 코드 `ContextV2Test`

```java
@Slf4j
public class ContextV2Test {

  /**
   * 전략 패턴 적용
   */
  @Test
  void strategyV1() {
    ContextV2 context = new ContextV2();
    context.execute(new StrategyLogic1());
    context.execute(new StrategyLogic2());
  }

  /**
   * 전략 패턴 익명 내부 클래스
   */
  @Test
  void strategyV2() {
    ContextV2 context = new ContextV2();
    context.execute(new Strategy() {
      @Override
      public void call() {
        log.info("비즈니스 로직1 실행");
      }
    });
    context.execute(new Strategy() {
      @Override
      public void call() {
        log.info("비즈니스 로직2 실행");
      }
    });
  }

  /**
   * 전략 패턴 익명 내부 클래스2, 람다
   */
  @Test
  void strategyV3() {
    ContextV2 context = new ContextV2();
    context.execute(() -> log.info("비즈니스 로직1 실행"));
    context.execute(() -> log.info("비즈니스 로직2 실행"));
  }
}
```

- `Context` 와 `Strategy` 를 ‘선 조립 후 실행’하는 방식이 아니라, `Context` 를 실행할 때마다 전략을 인수로 전달한다.
- **따라서 클라이언트는 `Context` 를 실행하는 시점에 원하는 `Strategy` 를 전달할 수 있다.**
    - 이전 방식과 비교해서 원하는 전략을 더욱 유연하게 변경할 수 있다.
- 또한 `Context` 를 하나만 생성하여, 실행 시점에 여러 전략을 인수로 전달해서 유연하게 실행할 수 있다.

### 전략 패턴 파라미터 실행 그림

![Untitled](/assets/img/2022-07-22-SPRING_Advanced_StrategyPattern/Untitled%203.png)

1. 클라이언트(여기에선 테스트 메서드)는 `Context` 를 실행하면서 인수로 `Strategy` 를 전달한다.
2. `Context` 는 `execute()` 로직을 실행한다.
3. `Context` 는 파라미터로 넘어온 `strategy.call()` 로직을 실행한다.
4. `Context` 의 `execute()` 로직이 종료된다.

<br/>

## 정리

### `ContextV1` vs `ContextV2`

- `ContextV1` 은 필드에 `Strategy` 를 저장하는 방식으로 전략 패턴을 구사했다.
    - 선 조립, 후 실행 방법에 적합하다.
    - `Context` 를 실행하는 시점에는 이미 조립이 끝났기 때문에 전략을 신경쓰지 않고 단순히 실행만 하면 된다.
- `ContextV2` 는 파라미터에 `Strategy` 를 전달받는 방식으로 전략 패턴을 구사했다.
    - 실행할 때마다 전략을 유연하게 변경할 수 있다.
    - 단점 역시 실행할 때마다 전략을 계속 지정해주어야 한다는 점이다.
- `ContextV2` 가 `ContextV1` 보다 더 적합한 이유
    - 지금 우리가 해결하고 싶은 문제는 변하는 부분과 변하지 않는 부분을 분리하는 것이다.
        - 변하는 부분 = 템플릿
        - **그 템플릿 안에서 변하는 부분에 약간 다른 코드 조각을 넘겨서 실행하는 것이 목적이다.**
    - **우리가 원하는 것은 선 조립, 후 실행이 아니다. 단순히 코드를 실행할 때 변하지 않는 템플릿이 있고, 그 템플릿 안에서 원하는 부분만 살짝 다른 코드를 실행하고 싶을 뿐이므로, `ContextV2` 가 더 적합하다.**

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*