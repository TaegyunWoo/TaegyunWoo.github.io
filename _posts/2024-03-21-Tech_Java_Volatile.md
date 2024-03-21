---
category: Tech
tags: [Tech]
title: "Java의 volatile 키워드 이해하기"
date: 2024-03-21 17:10:00 +0900
lastmod: 2024-03-21 17:10:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

Java에서 등장하는 volatile 키워드를 컴퓨터 동작 구조와 함께 이해해보는 시간을 가져보겠습니다.

<br/><br/>

## volatile 이란?

가장 먼저 volatile을 왜 사용하는지 먼저 알아볼까요?

- 멀티쓰레딩 환경에서 변수 값의 일관성을 보장하기 위해 사용한다.
- volatile이 적용된 변수의 값을 쓰레드 A에서 변경했을 때, 다른 쓰레드도 즉시 그 변경 값을 읽을 수 있게 된다.

여기서 우리는 ‘멀티쓰레딩 환경에서 변수 값의 일관성이 보장되지 않을 수 있다!’라는 것을 알아차려야 합니다.

<br/><br/>

## 멀티쓰레딩 환경에서의 변수 값 일관성

만약 여러 쓰레드가 동일한 변수에 접근하여 R/W 작업을 한다면, 각 쓰레드는 서로 다른 값을 인식할 가능성이 높습니다.

아래 코드를 볼까요?

```java
public class Main {

  static boolean flag = true;

  public static void main(String[] args) throws InterruptedException {
    Runnable runnable = () -> {
      boolean b = false;
      while (flag) {
        b = !b;
      }
    };

    Thread threadA = new Thread(runnable);

    threadA.start();
    ThreadA.sleep(1); //바로 쓰레드 A 실행

    flag = false;

    threadA.join(); //쓰레드 A가 끝날 때까지 기다림
    
    System.out.println("All process is done!");
  }
}
```

해당 코드를 실행시켰을 때, 어떤 결과가 예상되시나요?

우리의 동작 의도는 아래와 같습니다.

1. 메인 쓰레드가 `main 메서드` 를 실행시킨다.
2. 새로운 쓰레드 `threadA` 를 생성한다.
3. 메인 쓰레드가 1ms 동안 sleep 하므로, 바로 쓰레드 `threadA` 가 실행된다.
4. 쓰레드 `threadA` 에 의해 while 문을 반복한다.
5. 1ms 가 지난 후 메인 쓰레드가 `flag` 변수의 값을 false로 변경한다.
6. 쓰레드 `threadA` 는 `flag` 변수의 값이 false임을 확인해 while 문을 탈출하고 종료된다.
7. 메인 쓰레드가 종료된다.

하지만 직접 실행시켜보신 분은 아시겠지만, **쓰레드 `threadA` 가 종료되지 않고 while문을 무한히 반복**하는 것을 알 수 있습니다.

즉, 메인 쓰레드에서 변경한 `flag` 변수의 값을 쓰레드 `threadA` 가 알아차리지 못했다는 것이죠.

그 이유는 아래와 같습니다.

<br/>

### 컴퓨터 캐시 구조

![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled.png)

위 그림을 보면 각 CPU 코어가 서로 다른 Cache를 사용하고 있는 것을 알 수 있습니다.

그리고 바로 이 부분 때문에, **변수 값의 일관성이 깨지는 것**이죠.

다시 코드를 가지고 와볼까요?

```java
public class Main {

  static boolean flag = true;

  public static void main(String[] args) throws InterruptedException {
    Runnable runnable = () -> {
      boolean b = false;
      while (flag) {
        b = !b;
      }
    };

    Thread threadA = new Thread(runnable);

    threadA.start();
    ThreadA.sleep(1); //바로 쓰레드 A 실행

    flag = false;

    threadA.join(); //쓰레드 A가 끝날 때까지 기다림
    
    System.out.println("All process is done!");
  }
}
```

이 경우 각 쓰레드는 아래처럼 동작하게 됩니다.

1. `flag` 변수 초기화
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%201.png)
    
2. 쓰레드 `threadA` 실행
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%202.png)
    
    메인 메모리로부터 가져온 `flag` 변수의 값 true를 Cache에 저장한 뒤, 코드를 실행합니다.
    
    Cache에 저장된 `flag` 변수의 값이 true이므로, while 문을 계속 반복합니다.
    
3. 메인 쓰레드가 `flag` 변수 업데이트
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%203.png)
    
    메인 쓰레드가 `flag` 변수를 false로 업데이트 합니다.
    
    여기서 중요한 것은 **쓰레드 `threadA` (CPU Core B)에 캐싱된 `flag` 값은 여전히 true라는 것**입니다.
    
    이 때문에 `threadA` 는 여전히 while 문을 벗어나지 못합니다.
    

이 문제를 해결하는 방법에는 크게 두 가지가 있습니다.

- 매 while 반복마다, Cache를 Flush하는 방법
- 바로 Main Memory에서 데이터를 읽어오는 방법 (volatile)

각 방법에 대해 좀 더 자세히 알아볼까요?

<br/>

### 매 while 반복마다, Cache를 Flush하는 방법

이번에는 코드를 아래처럼 수정해봅시다.

```java
public class Main {

  static boolean flag = true;

  public static void main(String[] args) throws InterruptedException {
    Runnable runnable = () -> {
      while (flag) {
        System.out.println("Print to console!"); //변경사항
      }
    };

    Thread thread = new Thread(runnable);

    thread.start();
    Thread.sleep(1);

    flag = false;

    thread.join();
  }
}
```

위 코드를 실행해보면, **while문이 무한 반복하지 않고 정상적으로 종료**됩니다.

변경된 것은 `b = !b` 라는 계산 대신, `println` 메서드를 호출한 것 밖에 없는데 말이죠.

그 이유는 **콘솔 출력 등의 I/O 작업 같은 시스템 API 호출 시, Cache가 Flush 되기 때문**입니다.

이를 그림으로 다시 표현해보면 아래와 같습니다.

1. 변수 `flag` 초기화
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%204.png)
    
2. 쓰레드 `threadA` 실행
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%205.png)
    
3. 쓰레드 `threadA` 에서 `System.out.println()` (시스템 API) 호출
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%206.png)
    
    시스템 API를 호출함으로써 Cache가 Flush 되었습니다.
    
4. 메인 쓰레드가 `flag` 변수 업데이트
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%207.png)
    
5. 쓰레드 `threadA` 의 다음 반복문 수행 (Cache Miss)
    
    ![Untitled](/assets/img/2024-03-21-Tech_Java_Volatile/Untitled%208.png)
    
    **쓰레드 `threadA` 의 캐시가 Flush 되었으므로, 메인 메모리에서 `flag` 변수의 값을 가져옵니다.**
    
    이를 통해, 업데이트된 최신 값을 확인할 수 있고, **while문이 종료**됩니다.
    

이를 통해, 변수의 일관성이 깨지는 문제를 해결할 수 있습니다.

그렇다면 변수의 최신 값이 필요할 때마다 항상 시스템 API를 호출해야 할까요?

바로 이때, volatile을 사용하면 됩니다.

<br/>

### 바로 Main Memory에서 데이터를 읽어오는 방법 (volatile)

이번에는 코드를 아래처럼 수정해봅시다.

```java
public class Main {

  static volatile boolean flag = true; //volatile 키워드 사용

  public static void main(String[] args) throws InterruptedException {
    Runnable runnable = () -> {
      boolean b = false;
      while (flag) {
        b = !b; //기존 연산 (시스템 API 호출 X)
      }
    };

    Thread thread = new Thread(runnable);

    thread.start();
    Thread.sleep(1);

    flag = false;

    thread.join();
  }
}
```

해당 코드를 실행해보면, 쓰레드 `threadA` 가 무한 루프를 돌던 문제를 정상적으로 해결하게 되었음을 확인할 수 있습니다.

**volatile 변수는 항상 그 값을 메인 메모리에서 Read하게 됩니다. 따라서 최신 값을 언제나 유지할 수 있는 것이죠.**

이와 관련된 절차는 그림으로 표현하지 않더라도, 이해하실 수 있을 것입니다.

<br/><br/>

## 마치며

이번 글에서 volatile이 무엇이고, 그 속에 있는 원리까지 살펴봤습니다.

volatile은 언제나 변수의 최신 값을 가져올 수 있다는 장점이 있지만, CPU의 Cache를 사용하지 못하는 만큼 성능적인 측면에서 비교적 불리합니다.

따라서 꼭 최신 값을 읽어야하는 경우에만 사용하셔야 한다는 점을 강조하고, 글을 마치겠습니다.

혹시 글에 오류가 있다면, 댓글 남겨주세요. 감사합니다.