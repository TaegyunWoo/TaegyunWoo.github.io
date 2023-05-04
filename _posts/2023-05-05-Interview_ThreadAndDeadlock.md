---
category: Interview
tags: [코딩인터뷰 완전분석]
title: "[코딩인터뷰 완전분석] 면접 대비 정리 - 쓰레드와 락(Lock)"
date:   2023-05-05 00:05:00 +0900
lastmod : 2023-05-05 00:05:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

> 이 글은 게일 라크만 맥도웰 저자의 ‘코딩인터뷰 완전분석’을 읽으며, 깨달은 사실과 내용을 정리하는 글입니다.  
자세한 내용이 궁금하다면 [http://www.yes24.com/Product/Goods/44305533](http://www.yes24.com/Product/Goods/44305533) 에서 책을 구매하시면 좋겠습니다.
> 

빅테크 기업에서 쓰레드를 활용한 알고리즘 구현 문제는 잘 출제되지 않는다고 저자가 이야기합니다.

하지만 여기에 해당되는 기업은 미국 기업(아마존, MS, Google 등)이며, 한국의 상황은 다소 다를 수 있습니다.

또한 네이버 면접 후기를 찾아보던 중, 쓰레드 프로그래밍 구현을 해보라는 요청을 받았다는 내용이 있었습니다.

따라서 잘 출제되지 않는다고 저자가 설명하더라도, 쓰레드 개념과 실제로 어떻게 쓰레드를 활용해서 프로그래밍하는지 잘 알아둘 필요가 있다고 생각합니다.

추가로 쓰레드의 개념과 데드락에 대해서는 꽤나 빈번하게 만날 수 있다는 점을 기억하고 있는 것이 좋겠습니다.

*이 포스팅은 Java 언어를 기준으로 쓰레드에 대해 설명합니다. 이 점 참고해주세요!*

<br/><br/>

## Java의 쓰레드

Java의 모든 쓰레드는 `java.lang.Thread` 클래스 객체에 의해 생성되고 제어됩니다.

`main()` 메서드를 실행하면, 자동으로 사용자 쓰레드가 하나 생성되어 프로그램을 실행하는데 이것을 **주 쓰레드 (Main Thread)** 라고 합니다. (메인 쓰레드가 종료되면 프로세스가 종료되어버립니다.)

Java에서 쓰레드를 구현하는 방법으로는 아래 두가지가 있습니다.

- `java.lang.Runnable` 인터페이스를 구현하는 방법
- `java.lang.Thread` 클래스를 상속받아 구현하는 방법

그럼 각 방법에 대해 자세히 알아볼까요?

<br/>

### `Runnable` 인터페이스를 구현하는 방법

`Runnable` 인터페이스는 아래와 같이 단순하게 생겼습니다.

```java
public interface Runnable {
	void run(); //이것을 구현해서, 쓰레드 작업을 정의할 수 있다.
}
```

아래 단계는 어떻게 쓰레드를 만들고 사용하는지를 나타냅니다.

1. `Runnable` 인터페이스를 구현하는 클래스를 만든다.
2. `Thread` 타입 객체를 생성할 때, `Thread` 생성자의 파라미터에 `Runnable` 객체를 전달한다.
3. 2번 단계에서 생성한 `Thread` 객체의 `start()` 메서드를 통해, 쓰레드를 동작시킨다.

실제 구현 코드를 살펴보면 아래와 같습니다.

```java
public class Example {
	
	//Runnable 인터페이스 구현 클래스
  public static class RunnableExample implements Runnable {

    @Override
    public void run() {
      System.out.println("RunnableThread starting : " + Thread.currentThread().getName());

      try {
        Thread.sleep(5000); //쓰레드 실행 후 5초를 기다린다.
      } catch (InterruptedException e) {
        e.printStackTrace();
      }

      System.out.println("RunnableThread terminating : " + Thread.currentThread().getName());
    }
  }

  public static void main(String[] args) throws InterruptedException {
    RunnableExample instance = new RunnableExample(); //Runnable 객체 생성
    Thread thread = new Thread(instance); //새로운 Thread 객체 생성

    System.out.println("MainThread started : " + Thread.currentThread().getName());

    thread.start(); //새로운 쓰레드 실행

    thread.join(); //해당 쓰레드의 작업이 끝날 때까지 현재 쓰레드 (메인 쓰레드) 대기

    System.out.println("MainThread terminating : " + Thread.currentThread().getName());
  }
}
```

위 코드를 실행한 결과는 아래와 같습니다.

```
MainThread started : main
RunnableThread starting : Thread-0
RunnableThread terminating : Thread-0
MainThread terminating : main
```

- 가장 먼저 메인 쓰레드가 실행됩니다.
- 그 다음 다른 사용자 쓰레드를 생성하고 실행한 뒤, 메인 쓰레드는 해당 쓰레드가 종료될 때까지 대기합니다. (`join()`)
- 새로 생성된 쓰레드가 종료되면, 메인 쓰레드가 이어서 종료됩니다.

<br/>

### `Thread` 클래스를 상속받아 구현하는 방법

위처럼 `Runnable` 인터페이스를 구현하는 대신, 직접 `Thread` 클래스를 상속받아서 구현할 수도 있습니다.

구현 방법은 아래와 같습니다.

1. `Thread` 클래스를 상속받는 클래스를 만든다.
2. `run()` 메서드를 오버라이딩하여, 쓰레드가 수행할 작업을 작성한다.
3. 해당 클래스의 객체를 생성하고, `start()` 메서드를 사용해 실행한다.

코드는 위에서 상속 부분만 변경되었습니다. 그외는 모두 동일합니다.

```java
public class Example {
  public static class ThreadInheritExample extends Thread {

    @Override
    public void run() {
      System.out.println("RunnableThread starting : " + Thread.currentThread().getName());

      try {
        Thread.sleep(5000); //쓰레드 실행 후 5초를 기다린다.
      } catch (InterruptedException e) {
        e.printStackTrace();
      }

      System.out.println("RunnableThread terminating : " + Thread.currentThread().getName());
    }
  }

  public static void main(String[] args) throws InterruptedException {
    Thread thread = new ThreadInheritExample(); //새로운 Thread 객체 생성

    System.out.println("MainThread started : " + Thread.currentThread().getName());

    thread.start(); //새로운 쓰레드 실행

    thread.join(); //해당 쓰레드의 작업이 끝날 때까지 현재 쓰레드 (메인 쓰레드) 대기

    System.out.println("MainThread terminating : " + Thread.currentThread().getName());
  }
}
```

결과는 위와 동일합니다.

<br/>

### `Thread` 상속 방식 vs `Runnable` 구현 방식

지금까지 Java에서 지원하는 두 가지 쓰레드 프로그래밍 방법에 대해 알아봤습니다.

그렇다면 어떤 방식이 더 나을까요? 답은 **`Runnable` 인터페이스를 구현하는 방식**입니다. 그 이유는 아래와 같습니다.

- Java는 다중 상속을 지원하지 않기 때문에, `Thread` 클래스를 상속받으면 다른 클래스는 상속받을 수 없게 된다.  
하지만 인터페이스를 구현하는 것은 관계가 없기 때문에 `Runnable` 인터페이스 방식이 더 권장된다.
- `Thread` 클래스를 상속받으면 불필요한 다른 것들까지 모두 상속받게 된다. 따라서 `Runnable` 이 더 낫다.

<br/><br/>

## 동기화와 락

이번에는 쓰레드 간의 동기화 방법에 대해 알아보겠습니다.

한 프로세스 안에서 생성된 쓰레드들은 모두 같은 메모리 공간을 공유합니다.

> 한 프로세스 내의 모든 쓰레드는 메모리 영역 중 stack 영역을 제외한 모든 영역을 공유한다.  
쓰레드마다 서로 다른 함수를 실행해야하기 때문에, stack 영역은 쓰레드마다 독립적으로 존재한다.
> 

이 덕분에 멀티 쓰레드가 멀티 프로세스에 비해, 메모리 공간을 효율적으로 사용하고, 정보를 공유하는데 발생하는 오버헤드가 적습니다.  
**(같은 메모리 공간 공유 → 메모리 공간 적게 사용, 쓰레드간 통신 비용 ↓)**

하지만 **동시성 문제가 발생**할 수 있습니다.

공유자원에 대한 동시성 문제를 해결하기 위해, **Java에서는 `synchronized` 키워드와 `Lock` 클래스를 통해 동기화 처리를 지원합니다.**

각 동기화 방식에 대해 자세히 알아보겠습니다.

<br/>

### `synchronized` : 인스턴스 메서드 동기화

Java에서 지원하는 `synchronized` 키워드를 사용해서 **공유 자원에 대한 쓰레드의 접근을 제어**할 수 있습니다.

이 키워드는 메서드 단위와 블록 단위로 설정할 수 있습니다. 먼저 메서드 단위로 적용하는 방법에 대해 알아보겠습니다.

먼저 동기화 처리를 안한 메서드를 여러 쓰레드에서 동시에 호출하면 어떻게 될까요?

```java
public class Example {
	//비즈니스 로직 클래스
  public static class MyBusiness {
    public int count = 0;

		//동기화 처리하지 않은 메서드
    public void doNormal() {
      count++;
      for (int i = 0; i < 100000; i++) {} //대충 복잡한 로직
      System.out.println("[" + Thread.currentThread().getName() + "] count = " + count);
    }

		//동기화 처리한 메서드
    public synchronized void doSync() {
      count++;
      for (int i = 0; i < 100000; i++) {} //대충 복잡한 로직
      System.out.println("[" + Thread.currentThread().getName() + "] count = " + count);
    }
  }

	//쓰레드 구현 클래스
  public static class MyThread extends Thread {
    private MyBusiness business;

    public MyThread(MyBusiness business) {
      this.business = business;
    }

    @Override
    public void run() {
      System.out.println("[" + Thread.currentThread().getName() + "] MyThread started");
      business.doNormal(); //동기화 처리되지 않은 메서드 호출
    }
  }

  public static void main(String[] args) throws InterruptedException {
    MyBusiness business = new MyBusiness();
    Thread threadA = new MyThread(business); //새로운 Thread 객체 A 생성 (동일한 MyBusiness 객체 사용)
    Thread threadB = new MyThread(business); //새로운 Thread 객체 B 생성 (동일한 MyBusiness 객체 사용)

    threadA.start();
    threadB.start();
  }
}
```

`threadA` 와 `threadB` 를 순서대로 호출해서 `MyBusiness` 를 실행했기 때문에, `threadA` 의 작업이 먼저 끝나고 `threadB` 의 작업이 그 다음에 끝나기를 기대했습니다.

하지만 실제 결과는 아래와 같습니다.

```text
[Thread-0] MyThread started
[Thread-1] MyThread started
[Thread-1] job done.
[Thread-0] job done.
```

첫번째 쓰레드가 먼저 실행됐지만, 두번째 쓰레드가 먼저 작업을 종료했습니다.

이것이 바로 동시성 문제입니다.

<br/>

그렇다면 아래처럼 `synchronized` 키워드를 붙여서 동기화 처리한 메서드를 호출하면 어떻게 될까요?

```java
public class Example {
	//비즈니스 로직 클래스
  public static class MyBusiness {
    public int count = 0;

		//동기화 처리하지 않은 메서드
    public void doNormal() {
      count++;
      for (int i = 0; i < 100000; i++) {} //대충 복잡한 로직
      System.out.println("[" + Thread.currentThread().getName() + "] count = " + count);
    }

		//동기화 처리한 메서드
    public synchronized void doSync() {
      count++;
      for (int i = 0; i < 100000; i++) {} //대충 복잡한 로직
      System.out.println("[" + Thread.currentThread().getName() + "] count = " + count);
    }
  }

	//쓰레드 구현 클래스
  public static class MyThread extends Thread {
    private MyBusiness business;

    public MyThread(MyBusiness business) {
      this.business = business;
    }

    @Override
    public void run() {
      System.out.println("[" + Thread.currentThread().getName() + "] MyThread started");
      business.doSync(); //동기화 처리한 메서드 호출
    }
  }

  public static void main(String[] args) throws InterruptedException {
    MyBusiness business = new MyBusiness();
    Thread threadA = new MyThread(business); //새로운 Thread 객체 A 생성 (동일한 MyBusiness 객체 사용)
    Thread threadB = new MyThread(business); //새로운 Thread 객체 B 생성 (동일한 MyBusiness 객체 사용)

    threadA.start();
    threadB.start();
  }
}
```

이 경우, 예상대로 첫 쓰레드의 작업이 먼저 끝나고 두번째 쓰레드의 작업이 그 다음에 끝납니다.

```text
[Thread-0] MyThread started
[Thread-1] MyThread started
[Thread-0] job done.
[Thread-1] job done.
```

<br/>

하지만 **기억해야 하는 것은 `synchronized` 키워드는 동일한 객체에서만 동작한다는 것**입니다.

**즉 `threadA` 와 `threadB` 가 서로 다른 `MyBusiness` 객체를 사용한다면, `synchronized` 메서드라도 동시에 호출할 수 있습니다.**  
아래처럼 호출하면 말이죠!

```java
public static void main(String[] args) throws InterruptedException {
    MyBusiness business1 = new MyBusiness();
    MyBusiness business2 = new MyBusiness();
    Thread threadA = new MyThread(business1); //새로운 Thread 객체 A 생성 (서로 다른 MyBusiness 객체 사용)
    Thread threadB = new MyThread(business2); //새로운 Thread 객체 B 생성 (서로 다른 MyBusiness 객체 사용)

    threadA.start();
    threadB.start();
}
```

이렇게 서로 다른 `MyBusiness` 객체를 사용하는 경우, 그 결과는 아래와 같습니다.

```text
[Thread-0] MyThread started
[Thread-1] MyThread started
[Thread-1] job done.
[Thread-0] job done.
```

<br/>

### `synchronized` : static 메서드 동기화

정적 메서드에도 `synchronized` 키워드를 사용할 수 있습니다. 이때 동기화된 메서드는 모든 인스턴스에서 공유하는 클래스 메서드이기 때문에, **모든 인스턴스에 대해서 동기화**됩니다.

정적 메서드, 즉 static 메서드는 **‘클래스 락’** 에 의해서 동기화됩니다.

클래스 락이란, **클래스 단위로 락이 걸리는 동기화 방식**입니다. **특정 클래스의 모든 static 메서드를 함께 동기화 처리하게 됩니다.**  
**즉, 해당 클래스의 모든 static 메서드는 여러 쓰레드에서 동시에 실행될 수 없습니다.**

```java
public class Example {
  public static class MyBusiness {
		//첫번째 static synchronized 메서드
    public static synchronized void doSync1() {
      for (int i = 0; i < 100000; i++) {}
      System.out.println("[" + Thread.currentThread().getName() + "] doSync1 job done.");
    }
		
		//두번째 static synchronized 메서드
    public static synchronized void doSync2() {
      for (int i = 0; i < 100000; i++) {}
      System.out.println("[" + Thread.currentThread().getName() + "] doSync2 job done.");
    }
  }

  public static class MyThread extends Thread {
    private int methodNum;

    public MyThread(int methodNum) {
      this.methodNum = methodNum;
    }

    @Override
    public void run() {
      System.out.println("[" + Thread.currentThread().getName() + "] MyThread started");
      if (methodNum == 1) {
        MyBusiness.doSync1();
      } else if (methodNum == 2) {
        MyBusiness.doSync2();
      }
    }
  }

  public static void main(String[] args) throws InterruptedException {
    Thread threadA = new MyThread(1); //첫번째 static synchronized 메서드 실행
    Thread threadB = new MyThread(2); //두번째 static synchronized 메서드 실행

    threadA.start(); //동시 실행 불가
    threadB.start(); //동시 실행 불가
  }
}
```

**서로 다른 static 메서드라고 해도, 클래스 단위로 락이 걸리기 때문에 동시에 수행될 수 없습니다.**

결과는 아래와 같습니다.

```text
[Thread-0] MyThread started
[Thread-1] MyThread started
[Thread-0] doSync1 job done.
[Thread-1] doSync2 job done.
```

<br/>

### `synchronized` : 블록단위 동기화

메서드 단위가 아닌, 코드 블록을 단위로 동기화 처리할 수도 있습니다.

```java
public class MyClass {
	public void myMethod() {
		//some jobs...
		synchronized(this) {
			//synchronized jobs...
		}
		//some jobs...
	}
}
```

여기서 주목할 것은 `(this)` 입니다. 이것은 **동기화 처리를 할 기준**을 의미합니다.

`this` 를 기준으로 설정했으므로, **각 쓰레드가 동일한 `MyClass` 객체를 가지고 있다면 동기화 처리가 됩니다.**

<br/>

### 락 (Lock)

`synchronized` 의 단점은 동기화 작업을 세밀하게 조작할 수 없다는 것입니다.

이럴 경우, `Lock` 클래스를 사용하면 됩니다.

**락(’모니터’라고도 합니다)을 공유 자원에 붙이면 해당 자원에 대한 접근을 동기화할 수 있고, 세밀하게 조작할 수 있습니다.**

- `Lock` vs `synchronized`
    - `Lock` : 동기화 처리에 대한 다양한 기능을 지원한다.
        - 대기시간 제한
        - WAITING 상태의 쓰레드들을 INTERRUPT 처리 등…
    - `synchronized`
        - 세밀한 쓰레드 조작 불가능

**쓰레드가 해당 자원에 접근하려면, 우선 그 자원에 붙어 있는 락을 획득해야 합니다.**

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Example {
	//비즈니스 로직 클래스
  public static class MyBusiness {
    private Lock lock;

    public MyBusiness() {
      this.lock = new ReentrantLock();
    }

    public void doSync() {
      lock.lock(); //현재 쓰레드가 락을 건다.

      for (int i = 0; i < 1000000; i++) {}
      System.out.println("[" + Thread.currentThread().getName() + "] doSync job done.");

      lock.unlock(); //현재 쓰레드가 락을 반환한다.
    }
  }

	//쓰레드 클래스
  public static class MyThread extends Thread {
    private MyBusiness business;

    public MyThread(MyBusiness business) {
      this.business = business;
    }

    @Override
    public void run() {
      System.out.println("[" + Thread.currentThread().getName() + "] MyThread started");
      business.doSync();
    }
  }

  public static void main(String[] args) throws InterruptedException {
    MyBusiness business = new MyBusiness();
    Thread threadA = new MyThread(business); //새로운 Thread 객체 A 생성 (동일한 MyBusiness 객체 사용)
    Thread threadB = new MyThread(business); //새로운 Thread 객체 A 생성 (동일한 MyBusiness 객체 사용)

    threadA.start();
    threadB.start();
  }
}
```

- `lock.lock()`
    - 이를 호출한 쓰레드가 lock을 가져간다.
    - 만약 lock이 없다면 대기한다.
- `lock.unlock()`
    - lock을 반환한다.

출력 결과는 아래와 같습니다.

```text
[Thread-0] MyThread started
[Thread-1] MyThread started
[Thread-0] doSync job done.
[Thread-1] doSync job done.
```

<br/><br/>

## 데드락(교착상태)과 해결방법

### 데드락이란?

데드락이란, **첫번째 쓰레드는 두번째 쓰레드가 들고 있는 객체의 락이 풀리기를 기다리고, 두번째 쓰레드는 첫번째 쓰레드가 들고 있는 객체의 락이 풀리기를 기다리는 상황**을 의미합니다.

**모든 쓰레드가 락이 풀리기를 기다리고 있기 때문에, 무한 대기 상태에 빠지게 됩니다.**

<br/>

### 데드락 발생 조건

- **상호배제**
    - 한 번에 한 프로세스(쓰레드)만 공유 자원을 사용할 수 있는 것
- **들고 기다리기 (점유대기)**
    - 공유 자원에 대한 접근 권한을 갖고 있는 프로세스(쓰레드)가 자신의 권한을 양보하지 않은 상태에서, 다른 다른 자원에 대한 권한을 요구하는 상태
- **선취 불가능 (비선점)**
    - 다른 프로세스(쓰레드)의 자원 접근 권한을 강제로 취소할 수 없는 것
- **대기 상태의 사이클 (순환대기)**
    - 두 개 이상의 프로세스(쓰레드)가 자원 접근을 서로 기다리는 상태

마지막 조건을 방지하는 것이 가장 쉽기 때문에, 대부분의 데드락 방지 알고리즘은 4번 조건(순환대기)을 막는데 초점이 맞춰져 있습니다.

<br/>

### 데드락 해결 방안

- **무시**
    - 데드락 발생 확률이 낮은 시스템에서 발생한 데드락은 무시한다.
- **예방**
    - 데드락 발생 조건 중 하나를 예방한다.
- **회피**
    - 미리 사용할 자원에 대한 정보를 받아서, 순환대기를 방지한다.
- **탐지-회복**
    - 데드락이 발생한 사실을 탐지해서, 해당 부분을 해결한다.