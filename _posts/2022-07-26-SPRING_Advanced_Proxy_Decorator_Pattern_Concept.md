---
category: Spring-ADV
tags: [스프링]
title: "[스프링-ADV] 프록시 패턴, 데코레이터 패턴 개념"
date:   2022-07-26 01:00:00 
lastmod : 2022-07-26 01:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 프록시, 프록시 패턴, 데코레이터 패턴 개념

## 프록시란

- 이전 게시글에서 프록시 패턴과 데코레이터 패턴을 설명하기 위한 프로젝트를 설정하고, 새로운 요구사항을 추가했다.
    - 이전 게시글: [[스프링-ADV] 프록시 패턴, 데코레이터 패턴 예시 프로젝트 설정](https://taegyunwoo.github.io/spring-adv/SPRING_Advanced_Proxy_Decorator_Pattern_ExProject)
- 본격적으로 프록시 패턴과 데코레이터 패턴에 대해 알아보기 전에, 먼저 프록시가 무엇인지 알아보자.

### 클라이언트와 서버

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%205.png)

- 클라이언트와 서버는 `웹브라우저` 와 `서버 컴퓨터` 에 국한된 용어가 아니다.
- 클라이언트 = 의뢰인
    - **서버에 필요한 것을 요청하는 주체**
- 서버 = 서비스나 상품을 제공하는 사람이나 물건
    - **클라이언트의 요청을 처리하는 주체**
- 이 개념을 컴퓨터 네트워크에 도입하면 우리가 아는 `웹브라우저` 와 `서버 컴퓨터` 가 된다.
- **이 개념을 객체에 도입하면 아래와 같이 된다.**
    - **클라이언트 = `요청하는 객체`**
    - **서버 = `요청을 처리하는 객체`**

### 직접 호출

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%206.png)

- 클라이언트가 서버를 직접 호출하고, 처리 결과를 직접 받는다.
- 이것을 **직접 호출** 이라고 한다.

### 간접 호출

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%207.png)

- 위처럼 클라이언트가 어떤 대리자를 통해서 대신 간접적으로 서버에 요청할 수 있다.
- 예시
    - 내가 엄마(대리자)한테 장을 봐달라고 부탁한다.
- **여기서 이 대리자가 바로 프록시이다.**

### 프록시의 일반적인 역할

- **접근 제어, 캐싱**
    - 클라이언트가 프록시에게 특정 작업을 요청한다. 하지만 프록시가 이미 그 작업 결과를 알고 있어서, 서버에 접근하지 않고 자신의 것을 응답한다.
- **부가 기능 추가**
    - 프록시가 서버로부터 작업 결과를 받은 후, 프록시 스스로 추가적인 작업을 해서 클라이언트에게 응답한다.
- **프록시 체인**
    - 프록시가 또다른 프록시를 호출한다.
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%208.png)
    

### 객체에서 프록시의 역할

- 위에선 일반적으로 프록시가 하는 기능에 대해 설명한 것이다. 하지만 객체에서의 프록시는 좀 더 특별한 특성을 갖는다.
- 객체에서 프록시가 되려면, **클라이언트는 서버에게 요청을 한 것인지, 프록시에게 요청을 한 것인지 조차 몰라야 한다!**
- **즉, 서버와 프록시는 같은 인터페이스를 사용해야 한다. 그리고 클라이언트가 사용하는 서버 객체를 프록시 객체로 변경해도 클라이언트 코드를 변경하지 않고 동작할 수 있어야 한다.**

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%209.png)

- 클라이언트는 서버 인터페이스(`ServerInterface`)에만 의존한다.
- 그리고 서버와 프록시가 같은 인터페이스를 사용한다.
- **따라서 DI를 사용해서 대체 가능하다.**
- 런타임 시, 객체 의존 관계
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2010.png)
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2011.png)
    
    - 런타임(App 실행 시점)에 클라이언트 객체에 DI를 사용했다.
    - **`client ➡️ server` 에서 `client ➡️ proxy` 로 객체 의존관계를 변경해도 클라이언트 코드를 전혀 변경하지 않아도 된다.**
    - 즉 DI를 사용하면 클라이언트 코드의 변경없이 유연하게 프록시를 주입할 수 있다.

### 프록시의 주요 기능

다시 프록시를 통해 할 수 있는 일에 대해 설명하겠다.

- 접근 제어
    - 권한에 따른 접근 차단
    - 캐싱
    - 지연 로딩
- 부가 기능 추가
    - 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다.
    - Ex) 요청 값이나, 응답 값을 중간에 변형한다.
    - Ex) 실행 시간을 측정해서 추가 로그를 남긴다.

**프록시 객체가 중간에 있으면 대표적으로 ‘접근 제어'와 ‘부가 기능 추가'를 수행할 수 있다.**

### GOF 디자인 패턴에서의 프록시 사용법

- 프록시를 사용하는 디자인 패턴은 아래 두 가지가 있다.
    - 프록시 패턴
    - 데코레이터 패턴
- 프록시 패턴과 데코레이터 패턴을 구분하는 기준은 ‘의도(intent)’이다.
    - 프록시 패턴: 접근 제어가 목적
    - 데코레이터 패턴: 새로운 기능 추가가 목적
- **둘 다 프록시를 사용하지만, 의도가 다르다는 점이 핵심이다. 용어가 프록시 패턴이라고 해서 이 패턴만 프록시를 사용하는 것이 아니다. 데코레이터 패턴도 프록시를 사용한다.**

이제 본격적으로 예시 코드를 통해 프록시 패턴을 학습해보자.

> 이전 게시글에서 작성한 프로젝트를 사용한다.

<br/>

## 프록시 패턴 - 예시 1

### `build.gradle` 수정

테스트 코드를 기반으로 예시를 설명할 예정이니, Lombok을 테스트 코드에서 사용할 수 있도록 설정하자.

```text
dependencies {
  ...
  //테스트에서 lombok 사용
  testCompileOnly 'org.projectlombok:lombok'
  testAnnotationProcessor 'org.projectlombok:lombok'
}
```

위와 같이 해야 테스트 코드에서 `@Slf4j` 애너테이션이 작동한다.

### 예시 코드 구조

프록시 패턴을 이해하지 위한 예시 코드를 작성하기 전에, 코드 구조를 시각적으로 확인하자.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2012.png)

- `Client` 클래스가 `Subject` 인터페이스를 의존한다.
- `RealSubject` 는 `Subject` 인터페이스의 구현체이다.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2013.png)

### 테스트 코드 `Subject` 인터페이스

```java
package hello.proxy.pureproxy.code;

public interface Subject {
  String operation();
}
```

- 단순히 `operation()` 메서드 하나만 가지고 있다.

### 테스트 코드 `RealSubject`

```java
package hello.proxy.pureproxy.code;

@Slf4j
public class RealSubject implements Subject {
  @Override
  public String operation() {
    log.info("실제 객체 호출");
    sleep(1000);
    return "data";
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

- `RealSubject` 는 `Subject` 인터페이스를 구현했다.
- `operation()` 는 데이터 조회를 시뮬레이션 하기 위해 1초 쉬도록 했다.
    - 호출할 때마다 시스템에 큰 부하를 주는 데이터 조회라고 가정하자.

### 테스트 코드 `ProxyPatternClient`

```java
package hello.proxy.pureproxy.code;

public class ProxyPatternClient {

  private Subject subject;

  public ProxyPatternClient(Subject subject) {
    this.subject = subject;
  }

  public void execute() {
    subject.operation();
  }
}
```

- `Subject` 인터페이스에 의존하고, `Subject` 를 호출하는 클라이언트 코드이다.
- `execute()` 를 실행하면 `subject.operation()` 를 호출한다.

### 테스트 코드 `ProxyPatternTest`

```java
package hello.proxy.pureproxy.proxy;

public class ProxyPatternTest {
  
  @Test
  void noProxyTest() {
    RealSubject realSubject = new RealSubject(); //실제 서버 객체
    ProxyPatternClient client = new ProxyPatternClient(realSubject); //서버 객체 주입
    client.execute(); //클라이언트 실행
    client.execute();
    client.execute();
  }

}
```

- 테스트 코드에서는 `client.execute()` 를 3번 호출한다.
- 데이터를 조회하는데 1초가 소모되므로 총 3초의 시간이 걸린다.

### 실행 결과

- 아래는 실제 실행 결과이다.
    
    ```text
    실제 객체 호출
    실제 객체 호출
    실제 객체 호출
    ```
    
- `client.execute()` 를 3번 호출하면, 다음과 같이 처리된다.
    1. `client → realSubject` 를 호출해서 값을 조회한다. (1초 소요)
    2. `client → realSubject` 를 호출해서 값을 조회한다. (1초 소요)
    3. `client → realSubject` 를 호출해서 값을 조회한다. (1초 소요)
- **이 데이터가 한번 조회하면 변하지 않는 데이터라면, 어딘가에 보관해두고 이미 조회한 데이터를 사용하는 것이 성능상 좋다.**
    - 이것을 **캐시** 라고 한다.
- **프록시 패턴의 주요 기능인 접근 제어를 하는 방법이 바로 캐시이다.**

이제 이미 개발된 로직을 수정하지 않고, 프록시 객체를 통해서 캐시를 적용해보자.

<br/>

## 프록시 패턴 - 예시 2

### 예시 코드 구조

- 클래스 의존 관계에는 크게 변화가 없고, Proxy 클래스가 추가되었다.
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2014.png)
    
- 런타임 객체 의존 관계에서는 `client` 와 `realSubject` 사이에 `proxy` 객체가 중개를 한다.
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2015.png)
    

### 테스트 코드 `CacheProxy`

```java
package hello.proxy.pureproxy.code;

@Slf4j
public class CacheProxy implements Subject {

  private Subject target; //실제 객체
  private String cacheValue; //캐시하여 저장할 값

  public CacheProxy(Subject target) {
    this.target = target;
  }

  @Override
  public String operation() {
    log.info("프록시 호출");

    //캐시된 것이 없다면
    if (cacheValue == null) {
      cacheValue = target.operation();
    }

    return cacheValue;
  }
}
```

- **프록시 객체도 실제 객체(`realSubject`)과 모양이 같아야 하기 때문에, `Subject` 인터페이스를 구현해야 한다.**
- `private Subject target`
    - 클라이언트가 프록시를 호출하면 프록시가 최종적으로 실제 객체를 호출해야 한다.
    - **따라서 내부에 실제 객체의 참조를 가지고 있어야 한다.**
    - 이렇게 프록시가 호출하는 대상을 `target` 이라고 한다.
- `operation()`
    - `cacheValue` 필드에 값이 없으면, 실제 객체(`target` / `realSubject`)를 호출해서 값을 구한다. 그리고 구한 값을 `cacheValue` 필드에 저장하고 반환한다.
    - **만약 `cacheValue` 에 값이 있으면 실제 객체를 호출하지 않고, 캐시값을 그대로 반환한다. 따라서 처음 조회 이후에는 캐시(`cacheValue`)에서 매우 빠르게 데이터를 조회할 수 있다.**

### 테스트 코드 `ProxyPatternTest` 에 메서드 추가

```java
@Test
void cacheProxyTest() {
  Subject realSubject = new RealSubject(); //실제 서버 객체
  Subject cacheProxy = new CacheProxy(realSubject); //프록시 객체에 실제 객체 주입
  ProxyPatternClient client = new ProxyPatternClient(cacheProxy); //클라이언트에 프록시 객체 주입
  client.execute();
  client.execute();
  client.execute();
}
```

- `realSubject` 와 `cacheProxy` 를 생성하고 둘을 연결한다. 결과적으로 `cacheProxy` 가 `realSubject` 를 참조하는 런타임 객체 의존관계가 완성된다. 그리고 `client` 에 `realSubject` 가 아닌 `cacheProxy` 를 주입한다.
- **이 과정을 통해, `client → cacheProxy → realSubject` 런타임 의존 관계가 완성된다.**
- 클라이언트가 실제 `realSubject` 를 호출하는 것이 아니라, `cacheProxy` 를 호출하게 된다.

### 실행 결과

- 아래는 실제 실행 결과이다.
    
    ```text
    프록시 호출
    실제 객체 호출
    프록시 호출
    프록시 호출
    ```
    
- `client.execute()` 를 3번 호출하면, 다음과 같이 처리된다.
    1. `client` 의 `cacheProxy` 호출 → `cacheProxy` 에 캐시값이 없다. → `realSubject` 를 호출하고 결과를 캐시에 저장한다. (1초 소요)
    2. `client` 의 `cacheProxy` 호출 → `cacheProxy` 에 캐시값이 있다. → `cacheProxy` 에서 즉시 반환 (0초 소요)
    3. `client` 의 `cacheProxy` 호출 → `cacheProxy` 에 캐시값이 있다. → `cacheProxy` 에서 즉시 반환 (0초 소요)

<br/>

## 프록시 패턴 정리

### 결론

결과적으로 캐시 프록시를 도입하기 전에는 3초가 걸렸지만, **캐시 프록시 도입 이후에는 최초에 한번만 1초가 걸리고, 이후에는 거의 즉시 반환한다.**

### 정리

- 프록시 패턴의 핵심
    - `RealSubject` 코드와 클라이언트 코드를 전혀 변경하지 않고, 프록시를 도입해서 접근 제어를 했다는 것
    - 클라이언트 코드의 변경 없이 자유롭게 프록시를 넣고 뺄 수 있다. (DI를 통해!)
    - 실제 클라이언트 입장에서는 프록시 객체가 주입되었는지, 실제 객체가 주입되었는지 알지 못한다.
- 프록시 패턴 호출 흐름
    
    ![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2016.png)
    
<br/>

## 데코레이터 패턴 - 예시 1

### 예시 코드 구조

- 이제 데코레이터 패턴에 대해 알아보자.
- 역시나 데코레이터 패턴을 이해하기 위한 테스트 코드의 구조를 먼저 살펴보자.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2017.png)

이전에 살펴본 프록시 패턴 적용 전과 거의 같다.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2018.png)

### 테스트 코드 `Component` 인터페이스

```java
package hello.proxy.pureproxy.decorator.code;

public interface Component {
  String operation();
}
```

- 이것 역시 단순히 `operation()` 메서드를 갖는다.

### 테스트 코드 `RealComponent`

```java
package hello.proxy.pureproxy.decorator.code;

@Slf4j
public class RealComponent implements Component {
  @Override
  public String operation() {
    log.info("RealComponent 실행");
    return "data";
  }
}
```

- `RealComponent` 는 `Component` 인터페이스를 구현한다.
- `operation()`
    - 단순히 로그를 남기고 `"data"` 문자를 반환한다.

### 테스트 코드 `DecoratorPatternClient`

```java
package hello.proxy.pureproxy.decorator.code;

@Slf4j
public class DecoratorPatternClient {
  private Component component;

  public DecoratorPatternClient(Component component) {
    this.component = component;
  }

  public void execute() {
    String result = component.operation();
    log.info("result={}", result);
  }
}
```

- 클라이언트 코드는 단순히 `Component` 인터페이스를 의존한다.
- `execute()` 를 실행하면 `component.operation()` 을 호출하고, 그 결과를 출력한다.

### 테스트 코드 `DecoratorPatternTest`

```java
package hello.proxy.pureproxy.decorator;

@Slf4j
public class DecoratorPatternTest {

  @Test
  void noDecorator() {
    Component realComponent = new RealComponent();
    DecoratorPatternClient client = new DecoratorPatternClient(realComponent); //실제 서버 객체 주입

    client.execute();
  }

}
```

- `client → realComponent` 의존관계를 설정하고, `client.execute()` 를 호출한다.

### 실행 결과

```text
RealComponent - RealComponent 실행
DecoratorPatternClient - result=data
```

여기까지는 앞선 내용과 크게 다르지 않다.

<br/>

## 데코레이터 패턴 - 예시 2

### 부가 기능 추가

앞서 설명한 것처럼 ‘프록시 패턴'과 ‘데코레이터 패턴'을 구분하는 기준은 의도이다.

즉, ‘프록시 패턴'은 그 의도가 접근 제어(캐시)이고, ‘데코레이터 패턴'은 의도가 부가 기능 추가이다.

- 예시
    - 요청 값이나, 응답 값을 중간에 변형한다.
    - 실행 시간을 측정해서 추가 로그를 남긴다.

먼저 응답 값을 꾸며주는 데코레이터 프록시를 만들어보자.

### 예시 코드 구조

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2019.png)

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2020.png)

### 테스트 코드 `MessageDecorator`

```java
package hello.proxy.pureproxy.decorator.code;

@Slf4j
public class MessageDecorator implements Component {

  //실제 target(realComponent)나 또다른 프록시/데코레이터(프록시체인)를 주입받을 필드
  private Component component;

  public MessageDecorator(Component component) {
    this.component = component;
  }

  @Override
  public String operation() {
    log.info("MessageDecorator 실행");

    String result = component.operation();
    String decoResult = "*****" + result + "*****";
    log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);

    return decoResult;
  }
}
```

- 프록시가 호출해야 하는 대상을 `component` 에 저장한다.
- `operation()`
    - 프록시와 연결된 대상을 호출(`component.operation()`) 하고, 그 응답 값에 `*****` 를 더해서 꾸며준 다음 반환한다.
    - 꾸미기 전: `data`
    - 꾸민 후: `*****data*****`

### 테스트 코드 `DecoratorPatternTest` 에 메서드 추가

```java
@Test
void decorator1() {
  Component realComponent = new RealComponent(); //실제 서버 객체
  Component messageDecorator = new MessageDecorator(realComponent); //데코레이터 프록시 객체에 실제 서버 객체 주입
  DecoratorPatternClient client = new DecoratorPatternClient(messageDecorator); //클라이언트에 데코레이터 프록시 객체 주입

  client.execute();
}
```

`client → messageDecorator → realComponent` 객체 의존관계를 만들고, `client.execute()` 를 호출한다.

### 실행 결과

```text
MessageDecorator 실행
RealComponent 실행
MessageDecorator 꾸미기 적용 전=data, 적용 후=*****data*****
result=*****data*****
```

`MessageDecorator` 가 `RealComponent` 를 호출하고 반환한 응답 메시지를 꾸며서 반환한 것을 확인할 수 있다.

<br/>

## 데코레이터 패턴 - 예시 3

### 실행 시간을 측정하는 데코레이터

이번에는 기존 데코레이터에 더해서, ‘실행 시간을 측정하는 기능'까지 추가해보자.

### 예시 코드 구조

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2021.png)

`TimeDecorator` 데코레이터 프록시 클래스가 하나 더 추가된 것을 확인할 수 있다.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2022.png)

프록시 체인을 하여 `timeDecorator` 객체가 `messageDecorator` 객체를 의존하는 구조를 갖는다.

### 테스트 코드 `TimeDecorator`

```java
package hello.proxy.pureproxy.decorator.code;

@Slf4j
public class TimeDecorator implements Component {

  private Component component; //프록시 체인을 위해, messageDecorator가 저장되어야 한다.

  public TimeDecorator(Component component) {
    this.component = component;
  }

  @Override
  public String operation() {
    log.info("TimeDecorator 실행");
    long startTime = System.currentTimeMillis();
    String result = component.operation();
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("TimeDecorator 종료 resultTime={}ms", resultTime);
    return result;
  }
}
```

- `TimeDecorator` 는 실행 시간을 측정하는 부가 기능을 제공한다.
- 대상을 호출하기 전에 시간을 가지고 있다가, 대상의 호출이 끝나면 호출 시간을 로그로 남겨준다.

### 테스트 코드 `DecoratorPatternTest` 에 메서드 추가

```java
@Test
void decorator2() {
  Component realComponent = new RealComponent(); //실제 서버 객체
  Component messageDecorator = new MessageDecorator(realComponent); //실제 객체 주입
  Component timeDecorator = new TimeDecorator(messageDecorator); //데코레이터 객체 주입
  DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator); //클라이언트에 데코레이터 객체 주입

  client.execute();
}
```

`client → timeDecorator → messageDecorator → realComponent` 객체 의존관계를 설정하고, 실행한다.

### 실행 결과

```text
TimeDecorator 실행
MessageDecorator 실행
RealComponent 실행
MessageDecorator 꾸미기 적용 전=data, 적용 후=*****data*****
TimeDecorator 종료 resultTime=3ms
result=*****data*****
```

- `TimeDecorator` 가 `MessageDecorator` 를 실행하고, 실행시간을 측정해서 출력한 것을 확인할 수 있다.

<br/>

## 프록시 패턴과 데코레이터 패턴 정리

### GOF 데코레이터 패턴

실제 GOF에서 말하는 데코레이터 패턴은 아래와 같은 구조를 갖는다.

![Untitled](/assets/img/2022-07-26-SPRING_Advanced_Proxy_Decorator_Pattern_Concept/Untitled%2023.png)

- 우리가 기존에 작성한 `TimeDecorator` 와 `MessageDecorator` 에선 **항상 호출 대상인 `component` 를 가지고 있어야 하고, 또 호출해야 한다.**
    - 바로 이 부분이 코드의 중복이다.
    - **따라서 `component` 를 속성(필드)로 가지고 있는 `Decorator` 추상 클래스를 만드는 방법을 고안할 수 있다.**
    - **이렇게 하면 클래스 다이어그램에서 어떤 것이 실제 컴포넌트인지, 데코레이터인지 명확하게 구분할 수 있는 장점도 있다.**
- 여기까지가 GOF에서 말하는 데코레이터 패턴이다.

### 프록시 패턴 vs 데코레이터 패턴

프록시 패턴과 데코레이터 패턴은 그 모양이 거의 같고, 상황에 따라 정말 똑같을 때도 있다.

디자인 패턴에서 중요한 것은 해당 패턴의 겉모양이 아니라, **그 패턴을 만든 의도가 더 중요하다.**

**따라서 의도에 따라 패턴을 구분한다.**

- 프록시 패턴의 의도
    - 다른 개체에 대한 **“접근을 제어”**하기 위해 대리자를 제공
- 데코레이터 패턴의 의도
    - **”객체에 추가 책임(기능)을 동적으로 추가”** 하고, 기능 확장을 위한 유연한 대안 제공

### 정리

프록시를 사용하고 해당 프록시가 접근 제어가 목적이라면 프록시 패턴이고, 새로운 기능을 추가하는 것이 목적이라면 데코레이터 패턴이 된다.

다음 포스팅에선 이 패턴들을 활용하여 로그 추적기를 개선해볼 것이다.

<br>

---

<br>

<a href="https://inf.run/ru7S"><img src="/assets/img/Inflearn_Spring_Advanced/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*