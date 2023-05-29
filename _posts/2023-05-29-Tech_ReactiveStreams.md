---
category: Tech
tags: [Tech]
title: "Reactive-Streams와 Back Pressure"
date:   2023-05-29 16:15:00 +0900
lastmod : 2023-05-29 16:15:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

최근 Reactive Programming의 핵심 개념 중 하나인 BackPressure(배압)에 대한 이야기가 나와, 이를 정리해보고자 합니다.

비동기 반응형 프로그래밍의 대표적인 예시로는 Spring Webflux이 있습니다.

이 프레임워크를 직접 사용할 기회는 없었지만, 그 기반이 되는 기술의 개념을 알아두면 나중에 실제로 프로젝트를 위해서 Reactive Programming을 활용해야 할 때 도움이 될 수 있을 것 같아, 포스팅을 작성해봅니다.

<br/><br/>

## Reactive-Streams와 Spring WebFlux

Spring WebFlux에 대해 공부하다보면, Reactive-Streams 라는 단어가 등장하곤 합니다.

**Reactive-Streams는 일종의 라이브러리로, Non-Blocking BackPressure를 통해 비동기 스트림 처리를 하기 위한 표준으로 사용됩니다.**

쉽게 말해서, 아래와 같은 특성을 활용해서 프로그래밍할 때 필요한 표준 규범이라고 생각하면 됩니다.

- Non-Blocking, Asynchronous
- Stream, Back Pressure

> 자세한 것은 공식 문서 [https://www.reactive-streams.org/](https://www.reactive-streams.org/) 를 참고하세요.
> 

그렇다면 이 Reactive-Streams와 Spring Webflux는 무슨 관계일까요?

Spring WebFlux는 Reactive Programming을 기반으로 웹 서버 애플리케이션을 만들 수 있는 프레임워크입니다. 따라서 **Spring WebFlux는 Reactive-Streams를 기반으로 구현**됐습니다.

> 참고로 Reactive-Streams는 일종의 표준이기 때문에, 구체적인 구현체는 포함되어 있지 않으며 오직 인터페이스만 존재합니다.  
Reactive-Streams를 구현한 라이브러리에는 `RxJava`와 `Reactor` 가 있습니다.  
Spring WebFlux는 이 중 `Reactor` 라이브러리를 사용합니다.
> 

WebFlux와 Reactive-Streams 간의 관계를 표현하기 위해, 아래와 같이 그림을 그려봤습니다.

![Untitled](/assets/img/2023-05-29-Tech_ReactiveStreams/Untitled.png)

Spring WebFlux는 Reactive-Streams 라는 표준을 따른 Reactor 라이브러리를 기반으로 **‘비동기 반응형 프로그래밍’을 지원**하고, 추가적으로 단일 쓰레드를 활용한 **이벤트 루프 구조를 가졌**습니다.

이 두 가지 조합을 통해, 놀고 있는 쓰레드를 최소화하여 비교적 적은 수의 쓰레드로 최대한 많은 작업을 수행할 수 있게끔 만든 것이 바로 Spring WebFlux입니다.

> 예를 들어 요청당 쓰레드를 할당하는 Servlet MVC의 경우, DB에서 데이터를 조회할 때 DB가 응답할 때까지 해당 쓰레드는 대기하며 놀게 됩니다. (동기식 처리)  
하지만 비동기적으로 이를 처리하게 된다면, DB에 데이터를 요청해두고 다른 작업(다른 요청)을 처리할 수 있게 됩니다.
> 

Spring Webflux를 학습하기 전에, 기반이 되는 기술인 Reactive-Streams를 잘 이해해두는게 우선인 것 같습니다.

그리고 우리가 오늘 살펴볼 개념인 Back Pressure도 Reactive-Streams에서 등장하는 개념이기 때문에, 직접 Reactive-Streams의 구현체를 만들어보면서 개념을 익혀보겠습니다.

<br/><br/>

## Reactive-Streams 구조

![Untitled](/assets/img/2023-05-29-Tech_ReactiveStreams/Untitled%201.png)

Reactive-Streams의 구조를 살펴보면, 위 그림과 같습니다.

주요 요소로는 총 3개(`Publisher(출판자)` , `Subscriber(구독자)` , `Subscription(구독정보)`)가 존재합니다. 이들이 어떤 관계로 어떻게 동작하는지 살펴봅시다.

1. 가장 먼저 **구독자**가 출판자에게 **‘자신이 해당 출판자를 구독할 것’** 이라고 알립니다.  
이때, 구독자가 출판자의 `subscribe` 메서드를 호출합니다.
2. **출판자**는 전달받은 구독자 정보와 자신이 가지고 있는 데이터를 통해서 **‘새로운 구독정보’를 만들**어 냅니다.  
만들어진 구독정보에는 **‘구독자 객체에 대한 참조 변수’** 와 **‘구독할 데이터에 대한 참조 변수’** 가 존재합니다.
3. 그리고 **출판자**가 **이 구독정보를 구독자에게 전달**합니다.  
이때 출판자가 구독자의 `onSubscribe` 메서드를 호출해서 구독정보를 전달합니다.  
여기까지가 구독과정입니다.
4. 이제 구독에 필요한 작업을 완료했으므로, 실제 데이터를 받을 수 있습니다.  
**실제 데이터를 전달받을 때는 출판자가 해당 구독자에게 바로 데이터를 Push하는 것이 아니라, 구독자가 스스로 처리할 수 있는 데이터의 개수만큼 출판자에게 요청하게 됩니다. 그리고 이것이 배압(Back Pressure)을 구현하는 방법이 됩니다.**  
구독자가 구독정보의 `request` 메서드를 호출해서 데이터를 요청합니다.
    - 만약 출판자가 자신의 모든 데이터를 구독자에게 전달하게 되면, 구독자가 전달받은 데이터를 처리하기도 전에 새로운 데이터를 받게 됩니다. (빠른 출판자, 느린 구독자)
    - 이를 해결하기 위해, 큐(버퍼)를 도입해서 데이터를 하나씩 처리할 수도 있지만, 만약 큐의 용량을 넘어서는 수준의 데이터가 전달된다면 결국엔 데이터 누락이 발생합니다.
    - **따라서 구독자가 ‘원하는 데이터 개수 정보’와 함께 데이터를 요청했을 때만 데이터를 전달해주는 방식을 사용합니다.**  
    이 경우엔 별도의 큐(버퍼)가 필요없게 됩니다.
    이 방식을 **‘다이나믹 풀(pull)’** 이라고 합니다.
5. **구독정보가 가지고 있는 ‘출판자의 데이터에 대한 참조 변수’** 를 통해서, 전달할 데이터를 n개 가져옵니다.
6. 구독정보가 가져온 n개의 데이터를 구독자에게 전달합니다.  
이때 구독정보가 구독자의 `onNext` 메서드를 호출해서 전달합니다.

지금까지 Reactive-Streams가 어떤 구조를 갖게 되는지 살펴봤습니다.

그리고 가장 중요한 개념인 Back Pressure가 어떻게 구현되는지까지 알아봤습니다.

<br/><br/>

## 직접 Reactive-Streams 구현해보기

이제 위에서 살펴본 구조를 기반으로, 직접 Reactive-Streams의 구현체를 만들어볼까요?

`RxJava` 나 `Reactor` 와 같은 라이브러리를 우리 손으로 만들어보겠습니다.

> 프로젝트의 모든 소스코드는 아래 Repo를 참고하시면 됩니다.  
[https://github.com/TaegyunWoo/Study-Pratices/tree/main/Reactive-Streams](https://github.com/TaegyunWoo/Study-Pratices/tree/main/Reactive-Streams)
> 

<br/>

### Gradle 의존성 추가

```groovy
dependencies {
	implementation 'org.reactivestreams:reactive-streams:1.0.4'
}
```

위와 같이, 의존성을 추가해서 Reactive-Streams의 인터페이스들을 가져옵니다.

가져온 파일을 살펴보면 아래와 같습니다.

![Untitled](/assets/img/2023-05-29-Tech_ReactiveStreams/Untitled%202.png)

<br/>

### `Publisher` 인터페이스의 구현체 생성 (`MyPublisher`)

이제 Reactive-Streams의 인터페이스들을 하나씩 구현해보겠습니다.

가장 먼저 출판자 역할을 할 `Publisher` 인터페이스를 구현해볼까요?

```java
import org.reactivestreams.Publisher;
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

import java.util.*;

//출판자 클래스
public class MyPublisher implements Publisher<String> {
  private Iterator<String> data; //출판자의 데이터

	//데이터 초기화
  public MyPublisher() {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
      list.add("data[" + i + "]");
    }

    this.data = list.iterator();
  }

  /**
   * 구독자(Subscriber)가 구독할 때 호출할 메서드
   * @param s 구독할 구독자
   */
  @Override
  public void subscribe(Subscriber<? super String> s) {
    System.out.printf("[Publisher] 구독자(%s)로부터 구독이 시작됨.\n", s);

    Subscription subscription = new MySubscription(s, data); //구독정보 생성

    System.out.printf("[Publisher] 구독정보(%s) 생성함.\n", subscription);

    System.out.printf("[Publisher] 생성된 구독정보를 구독자(%s)에게 전달함.\n", s);
    s.onSubscribe(subscription); //구독정보를 구독자에게 전달
  }
}
```

`reactivestreams` 패키지의 `Publisher` 인터페이스를 implements해서, 출판자 클래스를 생성합니다.

- `subscribe(...)` 메서드
    1. 구독자가 구독할 때, 호출하는 메서드입니다. 이때, 구독자는 자신(`this`)의 정보를 파라미터에 넘겨주어야 합니다.
    2. 이 메서드가 호출되면, `전달받은 구독자 정보`와 `출판자 자신이 가지고 있는 데이터`를 통해서 새로운 구독정보(`MySubscription`) 객체를 생성합니다.
    3. 생성된 구독정보(`MySubscription`)을 다시 구독자에게 넘겨주게 됩니다.

<br/>

### `Subscription` 인터페이스의 구현체 생성 (`MySubscription`)

이제 출판자는 완성했으니, 구독정보 클래스를 만들어봅시다.

```java
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

import java.util.Iterator;

//구독 정보 클래스 (구독자, 구독할 데이터 정보)
public class MySubscription implements Subscription {
  private Subscriber subscriber; //구독자
  private Iterator<String> data; //구독할 데이터

	//생성자를 통해서, 구독자와 데이터 정보를 전달받는다.
  public MySubscription(Subscriber subscriber, Iterator<String> data) {
    this.subscriber = subscriber;
    this.data = data;
  }

  /**
   * 구독자(Subscriber)가 데이터를 요청할 때, 호출되는 메서드
   * @param n 한번에 요청할 데이터의 양
   */
  @Override
  public void request(long n) {
    System.out.printf("[Subscription] Subscriber가 %d개의 데이터를 요청함.\n", n);
    while (n-- > 0) { //n번 반복
      if (data.hasNext()) { //만약 데이터가 더 존재한다면
        subscriber.onNext(data.next());

      } else { //만약 데이터가 더 존재하지 않는다면
        subscriber.onComplete();
        break;
      }
    }
  }

  /**
   * 구독자(Subscriber)가 데이터 전송을 중단시킬 때, 호출되는 메서드
   */
  @Override
  public void cancel() {

  }
}
```

구독정보 클래스는 `reactivestreams` 패키지의 `Subscription` 인터페이스를 implements 해서 만듭니다.

- `request(...)` 메서드
    1. 구독자(`Subscriber`)가 데이터를 요청할 때, 호출하게 되는 메서드입니다.
    2. **구독자가 이 메서드를 호출해서 데이터를 가져오기 때문에, 출판자가 구독자를 압도하지 않게 됩니다.  
    이것을 Back Pressure 라고 합니다.**
    3. 한번에 구독자가 전달받을 데이터의 개수 (`파라미터 n`) 만큼 구독자의 `onNext(data)` 메서드를 호출해서 데이터를 전달해줍니다.
    4. 만약 더이상 전달할 데이터가 없다면, 구독자의 `onComplete()` 메서드를 호출해서 구독을 종료시킵니다.

*참고로 `cancel(...)` 메서드의 경우에는 다루지 않습니다. 그래도 전체 구조를 이해하는데에는 문제가 없거든요!*

<br/>

### `Subscriber` 인터페이스의 구현체 생성 (`MySubscriber`)

이제 대망의 구독자 클래스를 구현해보겠습니다.

```java
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

//구독자 클래스
public class MySubscriber implements Subscriber<String> {
  private static final int PULL_SIZE = 2; //한번에 가져올 데이터의 양
  private int bufferSize = 0; //현재 전달받은 데이터의 개수 관리
  private Subscription subscription; //출판자로부터 전달받은 구독정보 객체

  /**
   * 출판자(Publisher)로부터 구독정보(Subscription)을 전달받을 때 호출되는 메서드
   * @param s 전달받을 구독정보
   */
  @Override
  public void onSubscribe(Subscription s) {
    System.out.printf("[Subscriber] 구독정보(%s) 전달받음.\n", s);
    this.subscription = s;

    this.subscription.request(PULL_SIZE); //PULL_SIZE 만큼의 데이터 요청
  }

  /**
   * 구독정보(Subscription)로부터 데이터를 전달받을 때 호출되는 메서드
   * @param s 전달받은 데이터
   */
  @Override
  public void onNext(String s) {
    System.out.printf("[Subscriber] 데이터(%s)를 전달받음\n", s);
    bufferSize++; //하나를 전달받았기 때문에, 버퍼 사이즈 1 증가

    if (bufferSize == PULL_SIZE) {
      bufferSize = 0; //버퍼 사이즈 초기화
      this.subscription.request(PULL_SIZE);
    }
  }

  /**
   * 구독정보(Subscription)로부터 데이터 전달받는 도중, 에러가 발생하면 호출되는 메서드
   * @param t the throwable signaled
   */
  @Override
  public void onError(Throwable t) {

  }

  /**
   * 구독정보(Subscription)로부터 모든 데이터를 전달받으면 호출되는 메서드
   * 이때 모든 데이터는 한번에 전달받을 n개의 데이터가 아닌, 출판자가 가지고 있는 전체 데이터를 의미
   */
  @Override
  public void onComplete() {
    System.out.println("[Subscriber] 모든 데이터를 다 읽었기 때문에, 구독이 종료됨.");
  }
}
```

구독자 클래스는 `reactivestreams` 패키지의 `Subscriber` 인터페이스를 implements 해서 만들면 됩니다.

이 클래스가 다소 복잡해보일 수도 있지만, 하나씩 차근차근 살펴봅시다!

- `PULL_SIZE` 불변 static 필드
    - 이 변수는 구독자가 한번에 요청할 데이터의 개수를 의미합니다.
    - 구독자가 직접 데이터를 요청(`subscription.request(n)`)해야하기 때문에, 구독자 클래스에서 관리됩니다.
- `bufferSize` 필드
    - 이 변수는 구독자가 현재 전달받은 데이터의 개수를 관리합니다.
    - 여기서 현재라고 함은, ‘n개의 데이터를 요청해서, 실제로 n개의 데이터를 전달받는 동안’을 의미합니다.
- `onSubscribe(...)` 메서드
    1. 출판자가 구독을 요청받고(`publisher.subscribe(…)`) 새로운 구독정보를 만든 뒤에, 이 구독정보를 다시 구독자에게 전달해줄 때 사용됩니다.
    2. **전달받은 구독정보를 저장하고, 이 구독정보를 통해서 다음 데이터를 요청(`subscription.request(n)`)합니다.  
    이때 `PULL_SIZE` 만큼의 데이터를 요청하게 됩니다.  
    이것이 바로 Back Pressure 입니다.**
- `onNext(...)` 메서드
    1. 구독정보(`subsciption`)에서 데이터를 전달하기 위해서 호출됩니다.  
    파라미터를 통해서, 데이터를 전달받습니다.
    2. 구독정보에 의해서, `PULL_SIZE` 만큼 호출되어 데이터를 전달받습니다.
    3. 이때 데이터를 전달받을 때마다, `bufferSize` 를 증가시킵니다. 그리고 원하는 개수만큼 데이터를 전달받았다면, 다음 데이터를 `PULL_SIZE` 만큼 재요청합니다.

*역시 `onError(...)` 메서드는 구현하지 않아도, 이해에 어려움은 없습니다.*

<br/>

### 테스트

이제 모든 준비가 끝났으니, 한번 실행해볼까요?

```java
public static void main(String[] args) {
  Publisher<String> publisher = new MyPublisher(); //출판자 생성
  Subscriber<String> subscriber = new MySubscriber(); //구독자 생성

  publisher.subscribe(subscriber); //구독 요청
}
```

위와 같이 메인 메서드를 작성해서 실행해봅시다.

그러면 아래와 같은 결과를 볼 수 있습니다.

```
[Publisher]    구독자(reactive.streams.MySubscriber@74a14482)로부터 구독이 시작됨.
[Publisher]    구독정보(reactive.streams.MySubscription@14ae5a5) 생성함.
[Publisher]    생성된 구독정보를 구독자(reactive.streams.MySubscriber@74a14482)에게 전달함.
[Subscriber]   구독정보(reactive.streams.MySubscription@14ae5a5) 전달받음.
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   데이터(data[0])를 전달받음
[Subscriber]   데이터(data[1])를 전달받음
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   데이터(data[2])를 전달받음
[Subscriber]   데이터(data[3])를 전달받음
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   데이터(data[4])를 전달받음
[Subscriber]   데이터(data[5])를 전달받음
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   데이터(data[6])를 전달받음
[Subscriber]   데이터(data[7])를 전달받음
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   데이터(data[8])를 전달받음
[Subscriber]   데이터(data[9])를 전달받음
[Subscription] Subscriber가 2개의 데이터를 요청함.
[Subscriber]   모든 데이터를 다 읽었기 때문에, 구독이 종료됨.
```

1. 구독을 요청하면, 출판자가 구독정보를 생성하고 구독자에게 전달해주게 됩니다.
2. 구독 정보를 전달받은 구독자가 n개의 데이터를 요청하게 됩니다.
3. 구독 정보로부터 n개의 데이터를 모두 전달받게 되면, 다시 n개의 데이터를 요청하게 됩니다.
4. 모든 데이터를 전달받을 때까지 반복합니다.

<br/><br/>

## 정리하며

지금까지 반응형 프로그래밍의 기본이 되는 개념과 표준 인터페이스 `Reactive-Streams` 에 대해 알아봤습니다.  
그리고 직접 `Reactive-Streams` 인터페이스들을 간단하게나마 구현해보면서, 실제로 어떻게 동작하는지 이해해봤습니다.

Spring WebFlux의 경우, 단순히 `Reactive-Streams` 만을 사용하는 것이 아니라, 단일 쓰레드를 활용한 이벤트 루프 기반의 처리 방식까지 더해진 웹 프레임워크입니다.  
따라서 이번 포스팅만으로 Spring WebFlux까지 완벽히 이해할 수는 없었지만, 그 기본이 되는 개념 중 하나를 잘 익혀둘 수 있어 좋았습니다.

`Reactive-Streams` 와 핵심 개념인 Back Pressure를 이해하시는데 도움이 되었으면 좋겠습니다.

감사합니다.

<br/><br/>

## References

- [메타 코딩 유튜브 - WebFlux](https://www.youtube.com/watch?v=Sz1Ve_1KZII&list=PL93mKxaRDidFH5gRwkDX5pQxtp0iv3guf)
- [Reactive-Streams](https://www.reactive-streams.org/)
- [Baeldung - Backpressure Mechanism in Spring WebFlux](https://www.baeldung.com/spring-webflux-backpressure)