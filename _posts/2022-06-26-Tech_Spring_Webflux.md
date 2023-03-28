---
category: Tech
tags: [Tech]
title: "Spring Webflux 찍먹하기"
date:   2022-06-26 18:10:00 
lastmod : 2022-06-26 18:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# About Spring Webflux

> [참고한 발표영상: [10분 테코톡] 호돌의 Spring Webflux](https://www.youtube.com/watch?v=4x1QRyMIjGU)

<br/>

## Spring Webflux 배경

### Spring Webflux 특징

- 적은 수의 쓰레드로 동시성 처리
- 논블로킹
- Functional Programming
- Reactive Programming

위와 같은 특징을 토대로, Spring Webflux에 대해 설명할 예정이다.

### 기존 Spring MVC의 쓰레드풀 방식

![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled.png)

아래는 기존의 Spring MVC가 갖는 멀티쓰레드 방식의 문제점이다.

- CPU·메모리가 충분하지만 **쓰레드가 모자라서 처리율이 저하**될 수 있다.
- 그래서 쓰레드를 과도하게 늘리면 이번엔 **메모리·CPU에 부하**로 성능이 저하된다.
- 컨텍스트 스위칭 과정에서 엄청난 오버헤드 발생
- **즉, 쓰레드를 무조건 늘린다고 문제를 해결할 수 있는 것은 아니다.**

그렇다면 이 문제를 Netflix는 어떻게 해결했을까?

### Netflix의 개선방식

- 기존방식
    
    ![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%201.png)
    
    - 한 명의 유저가 접속 시, 여러 API를 호출한다.
    - **이때 네트워크 지연율이 증가한다.**
        - 요청 트래픽이 많기 때문에

- 개선된 방식
    
    ![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%202.png)
    
    - 해당 문제를 개선하기 위해, 하나의 API로 줄여버렸다.
    - 이때 서버는 **기존 로직 순서대로 처리**해줘야 한다.
    - 그래서 동시에 많은 작업을 서버에서 처리해줘야 한다.
    - 따라서 **‘하나의 쓰레드에서 병렬적으로 처리하자!’** 라는 취지로 `RxJava` 를 도입했다.
        
        > `RxJava` 에 대해선, 글 후반부에 다룬다.

<br/>

## 동시성과 논블로킹

### 기존 Spring MVC는 어떻게 동시성을 처리하는가?

![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%203.png)

- Application에서 I/O 요청을 한 후, 완료되기 전까지는 Application이 Block되어 다른 작업을 수행할 수 없다.
- 하지만 우리는 Blocking이 되지 않는 것처럼 느껴진다.
- **왜냐하면 Multi Thread 기반으로 동작하기 때문이다.**
    - Blocking이 되는 순간, 다른 Tread가 동작함으로써 Blocking의 문제를 해소하였다.
- **하지만 쓰레드간의 전환(Context Switching)이 발생하면 오버헤드(비용)이 발생하므로, 여러 개의 I/O 작업을 처리하기 위해 여러 쓰레드를 사용하는 비효율적일 수 있다.**

### 그렇다면 어떻게 Webflux는 ‘적은 수의 쓰레드’로 동시성을 처리하는가?

Spring Webflux는 이벤트루프를 활용하여, 동시성을 처리한다.

이벤트루프란 아래와 같다.

![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%204.png)

- Webflux는 이벤트루프 기반으로 동작하여, 적은 수의 쓰레드로도 동시성 작업을 수행할 수 있다.
- 이벤트루프?
    1. 작업 큐(Queue)에 쌓인 작업을 처리하기 위해, 동기적으로 동작한다. ⇒ 이벤트루프
        
        아래는 ‘이벤트루프’라는 장치를 이해하기 위한 코드이다.
        
        ```java
        while (queue.waitForMessage()) {
          queue.processNextMessage();
        }
        ```
        
    2. 이때 클라이언트가 요청을 보낸다.
    3. 이 요청을 처리할 작업(Context, Message)가 큐에 저장된다.
    4. 큐에 저장된 작업을 처리한다.

> 이벤트루프에 대한 보다 자세한 내용은 아래 자료를 참고하자.  
1. [[10분 테코톡] 피터의 이벤트루프](https://www.youtube.com/watch?v=wcxWlyps4Vg)  
2. [NHN Cloud - 자바스크립트와 이벤트 루프](https://meetup.toast.com/posts/89)  
3. [MDN - JavaScript 이벤트루프](https://developer.mozilla.org/ko/docs/Web/JavaScript/EventLoop)

<br/>

이러한 이벤트루프를 토대로, 논블로킹 방식으로 동작한다.

아래는 논블로킹 방식의 한 예이다.

![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%205.png)

- Application이 I/O 요청을 한 후, 다시 제어권이 Application으로 돌아온다.
    - 즉 Non-Blocking I/O 와 마찬가지로 즉시 리턴된다.
- 데이터 준비가 완료되면 이벤트를 발생시켜 알려주거나, 미리 등록해놓은 callback을 통해서 이후 작업이 진행된다.
- **Blocking이 없기 때문에 자원을 보다 더 효율적으로 사용할 수 있다.**

> ‘동기·비동기 / 블로킹·논블로킹’ 에 대한 보다 상세한 설명은 아래 동영상을 참고하자.  
[[10분 테코톡] 우의 Block vs Non-Block & Sync vs Async](https://www.youtube.com/watch?v=IdpkfygWIMk)

### 그렇다면 Webflux가 MVC보다 무조건 빠른가?

- 결론적으로 말하면 아니다.
- Webflux는 적은 쓰레드와 메모리를 효율적으로 사용하는 것일 뿐이다.
- **’속도가 빠르다' 보다는 ‘적은 리소스로 많은 트래픽을 감당할 수 있다'라고 해야 한다.**

<br/>

## Reactive Programming

### Reactive Programming 이란?

- 사전적 정의
    - Non-Blocking IO에 “Reactive Stream”과 “Backpressure”를 곁들인 프로그래밍 기법
    - Publisher가 Subscriber를 압도하지 못하게 하는 목적이 핵심인 Backpressure를 가지고, 작은 수의 쓰레드로 확장성 있는 Non-Blocking / Event Driven 개발을 지향한다.
- 다소 부정확할 수 있지만, 쉽게 풀어본다면…
    - **Reactive Stream**
        - 비동기, 논블로킹으로 작동하는 `Stream` 이다.
        - 또한 그 `Stream` 의 기능이 매우 많다.
        - `Publisher`(웹클라이언트, DB)에서 변경이 생기면, 변경된 데이터들을 `Subscriber` 에 `Stream` 으로 전달한다.
        - **바로 이 `Stream` 으로 프로그래밍하는 패러다임이 Reactive Programming 이다.**
            
            > 모든 것이 `Stream` !
        - 예시) 엑셀로 표현한다면…
            
            ![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%206.png)
            
    - **Backpressure**
        - `Subscriber` 로 들어오는 `Stream` 의 양을 조절하는 것이다.
        - 즉 적은 컴퓨팅 자원으로 일을 처리하기 가능한 정도씩만 받고자 하는 것이다.
            - 한번에 너무 많은 `Stream` 이 들어오면, 모두 처리할 수 없으므로…

<br/>

## Reactive 라이브러리

### 리액터 & RxJava

리액터와 RxJava는 리액티브 스트림이라는 JDK 스펙의 인터페이스를 구현한 라이브러리이다.

### **리액터(Reactor)**

- 리액티브 라이브러리(구현체)이다.
- Spring Webflux가 사용하는 라이브러리이다.

### **RxJava**

- 또다른 리액티브 라이브러리이다.
- Netflix가 사용하는 라이브러리이다.

<br/>

## Webflux와 리액티브 프로그래밍

### Webflux가 대체 무엇인가요?

```java
@Test
void mapFlux() {
	Flux<Crew> crewFlux = Flux.just("Hodol", "Kyle", "DD")
    .map(crew -> new Crew(crew, 27));
  StepVerifier.create(crewFlux.log())
    .expectNext(new Crew("Hodol", 27))
    .expectNext(new Crew("Kyle", 27))
    .expectNext(new Crew("DD", 27))
    .verifyComplete();
}
```

위 코드는 Webflux에 대한 예시 테스트코드이다.

> 나중에 좀 더 자세히 설명할 예정이니, ‘이렇게 생겼구나~’ 정도만 참고하자.
> 
- Spring Webflux란 쉽게 말해, Java8의 `Stream` 과 유사한데 비동기적으로 작동할 수 있는 것이다.
- 기존의 `Stream` API와는 다르게, 단일 항목(Collection이 아닌)에 대해서도 사용이 가능하다!

<br/>

## Webflux의 또다른 기능

### Annotated Controller

- 스프링 MVC에서 제공되는 일반적인 컨트롤러 형태와 일치하다.
- 하지만 ‘리액티브 `@RequestBody` 인수를 사용할 수 있다.
- 예시
    
    ```java
    //기존 Spring MVC 컨트롤러 메서드 형식
    @PostMapping("/accounts")
    public void handle(@RequestBody Account account) {
      //...
    }
    
    //Spring Webflux
    @PostMapping("/accounts")
    public void handle(@RequestBody Mono<Account> account) {
      //...
    }
    ```
    

### Functional Endpoints

- 람다 기반의 가벼운 함수형 프로그래밍 모델
- **애플리케이션이 요청을 라우팅하고 처리하는데 사용할 수 있는 작은 라이브러리·유틸리티 집합이라고 생각할 수 있다.**
- 예시
    
    아래 코드는 요청을 처리하는 비즈니스 로직이 담긴 리액티브 핸들러이다.
    
    ```java
    public class PersonReactiveHandler {
      public Mono<ServerResponse> listPeople(ServerRequest request) {
        //...
      }
    
      public Mono<ServerResponse> createPeople(ServerRequest request) {
        //...
      }
    
      public Mono<ServerResponse> getPeople(ServerRequest request) {
        //...
      }
    }
    ```
    
    아래 코드는 핸들러를 라우팅하는 코드이다.
    
    ```java
    PersonRepository repository = ...
    PersonHandler handler = new PersonHandler(repository); //핸들러 객체
    
    RouterFunction<ServerResponse> otherRoute = ...
    
    //핸들러 라우팅
    RouterFunction<ServerResponse> route = route()
      .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
      .GET("/person", accept(APPLICATION_JSON), handler::listPerson)
      .POST("/person", handler::createPerson)
      .add(otherRoute)
      .build();
    ```

<br/>

## 정리

### Webflux 사용법

1. 핸들러 Function 작성
2. 라우터 Function 작성
3. Configuration에 Bean으로 등록
    - 일반적인 스프링 컨테이너 Configuration임

### Spring MVC vs Spring Webflux

![Untitled](/assets/img/2022-06-26-Tech_Spring_Webflux/Untitled%207.png)

- 왜 Webflux에는 `JPA` , `JDBC` 등이 없는가?
    - 왜냐하면 `JPA` , `JDBC` 등은 블로킹 방식으로 동작하기 때문에, Webflux의 의미가 없어지기 때문이다.
    - 따라서 Redis 와 같이 리액티브가 지원되는 DB를 사용해야 한다.
    - **즉 Spring Webflux를 사용하려면, 리액티브가 지원되는 모듈을 End-to-End로 함께 사용해야 한다.**

### Spring Webflux 적용 시, 고려사항

- 제대로 동작하는 스프링 MVC 애플리케이션이라면 변경할 필요 없다.
    - 명령형 프로그래밍은 코드를 작성하고, 이해하고, 디버깅하는 가장 쉬운 방법이다.
    - 대부분 라이브러리가 블로킹 방식이기 때문에, 오히려 스프링 MVC를 사용할 때 선택권이 넓다.
- 논 블로킹 웹 스택을 고려하고 있다면, 스프링 Webflux는 다른 웹 스택과 동일한 실행 모델 이점을 제공하며, 서버, 프로그래밍 모델, 리액티브 라이브러리에 대한 선택지를 제공한다.
- Java8의 람다 또는 코틀린과 함께 사용할 함수형 웹 프레임워크를 찾는다면, 스프링 Webflux 함수형 웹 엔드포인트를 사용할 수 있다.
    - 소규모의 애플리케이션이나 더 큰 명료함과 제어를 더 적은 복잡도로 제공하는 마이크로 서비스에 좋은 선택이 될 수 있다.

### 결론

- Spring 5에서는 기존의 Servlet Stack과 다른 Reactive Stack을 만들어냈다.
- Spring Webflux는 Spring Reactive Stack의 Web Framework 역할을 맡는다. (Spring MVC의 포지션)
- Reactive Programming은 컴퓨팅 자원을 효율적으로 운용하기 위해 고안된 프로그래밍 패러다임이다.
- Reactive Programming은 Asynchronous Non-Blocking IO, Data Stream에 기반해, 메서드 체이닝과 람다식으로 이루어진다.
- Reactive Stream에서 `Publisher` 는 `Subscriber` 에 Data Stream을 제한없이 보내고, 이는 Backpressure로 제한한다.
- `Mono` , `Flux` 는 단일 항목이던 복수 항목이던 `Stream` 처럼 사용하되, Reactive하게 작동할 수 있다.
- 새로운 Functional Endpoints를 통해, 조금 더 함수형 프로그래밍스럽게 `Controller` 를 대체할 수 있다.
- End-to-End Reactive 하지 않다면, 의미가 없다.
    - `Spring Security Reactive` , `ReactiveRepository` 를 함께 사용하자.
- Spring MVC가 멀쩡히 잘 돌아가면, Spring Webflux로 반드시 갈아탈 필요는 없다.

### 실제 프로젝트 예시 만들기
실제로 Spring Webflux를 활용한 프로젝트 예시는 아래 링크를 참고하자.
[DEVKUMA - Spring WebFlux의 간단한 사용법](https://www.devkuma.com/docs/spring-webflux/)