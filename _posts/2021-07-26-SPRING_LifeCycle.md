---
category: Spring-Core
tags: [스프링, 핵심원리, 빈생명주기, 초기화, 종료]
title: "[스프링 - 핵심원리] 빈 생명주기 콜백과 관련 기능들"
date:   2021-07-26 15:30:00 
lastmod : 2021-07-26 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>


# 스프링 빈의 라이프사이클

## 스프링 빈이 가지는 라이프사이클

- **객체 생성 ⇒ 의존관계 주입**

스프링 빈은 위와 같은 라이프사이클을 갖는다. 스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다. **따라서 초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출해야 한다.** 그렇다면 스프링이 의존관계 주입을 완료했는지 어떻게 알 수 있을까?

> 초기화 작업?  
'DB 커넥션 풀'과 같은 기능을 구현할 때, 필요한 객체 초기화 작업을 뜻한다.

**스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공** 한다. 또한 **스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백**을 준다.

<br><br>

## 스프링 빈의 이벤트 라이프사이클

1. **스프링 컨테이너 생성**
2. **스프링 빈 생성**
3. **의존관계 주입**
4. **초기화 콜백**
5. **빈 객체 사용**
6. **소멸전 콜백**
7. **스프링 종료**

스프링 빈의 이벤트 라이프사이클은 위와 같다. 해당 라이프사이클에 따라 호출되는 콜백함수는 다음과 같다.

- **초기화 콜백**

    - 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출되는 콜백 함수
- **소멸전 콜백**
    - 빈이 소멸되기 직전에 호출되는 콜백 함수

<br>

### 참고: 객체의 생성과 초기화를 분리해야한다.

- **생성자** 는 필수 정보(파라미터)를 받고, 메모리를 할당해서 객체를 생성하는 책임을 가진다.

    - 생성자: 오직 객체생성을 위한 값세팅
- 반면에 **초기화** 는 이렇게 생성된 값들을 활용해사 외부 커넥션을 연결하는 등의 무거운 동작을 수행한다.
- **따라서, 생성자 안에서 무거운 초기화 작업을 함께 하는 것 보다는 객체를 생성하는 부분과 초기화 하는 부분을 명확하게 나누는 것이 유지보수 관점에서 좋다.**

<br><br><br>

# 스프링의 빈 생명주기 콜백 기능

## 빈 생명주기 콜백 지원의 방식

- 인터페이스(`InitializingBean` , `DisposableBean`) 방식
- 설정 정보에 초기화 메서드, 종료 메서드 지정하는 방식
- `@PostConstructor` , `@PreDestroy` 애노테이션 방식

스프링은 위와 같이 총 3가지 방법으로 빈 생명주기 콜백을 지원한다. 지금부터 하나씩 알아보자.

<br><br>

## 인터페이스(`InitializingBean` , `DisposableBean`) 방식

### 예시코드

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

//빈 생명주기와 관련된 두가지 인터페이스를 implements
public class NetworkClient implements InitializingBean, DisposableBean {

	private String url;
	
	public NetworkClient() {
		System.out.println("생성자 호출, url = " + url);
	}

	public void setUrl(String url) {
		this.url = url;
	}

	//서비스 시작시 호출하고자 하는 메서드
	public void connect() {
		System.out.println("connect: " + url);
	}

	//비즈니스 로직으로서 호출하고자 하는 메서드
	public void call(String message) {
		System.out.println("call: " + url + " message = " + message);
	}

	//서비스 종료시 호출하고자 하는 메서드
	public void disConnect() {
		System.out.println("close + " + url);
	}

	//의존관계 주입 완료 후 동작할 내용을 작성하는 메서드
	@Override
	public void afterPropertiesSet() throws Exception {
		//DI 완료후 동작할 내용 작성
		connect();
		call("초기화 연결 메시지"); //비즈니스 로직 호출
	}

	//빈 종료시 동작할 내용을 작성하는 메서드
	@Override
	public void destroy() throws Exception {
		//빈 종료시 동작할 내용 작성
		disConnect();
	}
}
```

- `InitializingBean` 인터페이스
    - `afterPropertiesSet()` 메서드로 초기화를 지원한다.
- `DisposableBean` 인터페이스
    - `destroy()` 메서드로 소멸을 지원한다.

<br>

### 테스트 코드

```java
public class BeanLifeCycleTest {

	@Test
	public void lifeCycleTest() {
		ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
		NetworkClient client = ac.getBean(NetworkClient.class);
		ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
	}

	@Configuration
	static class LifeCycleConfig {
		@Bean
		public NetworkClient networkClient() {
			//이때, 생성자가 동작한다.
			NetworkClient networkClient = new NetworkClient();
			networkClient.setUrl("http://hello-spring.dev");
			return networkClient;
		}
		//위 메서드가 실행된 후, 빈이 등록된다.
		//이때, afterPropertiesSet 메서드가 동작한다.
	}

}
```

<br>

### 초기화, 소멸 인터페이스 단점

- 스프링 전용 인터페이스이기에, 해당 코드가 스프링 전용 인터페이스에 의존한다.
- 초기화, 소멸 메서드의 이름을 변경할 수 없다.
- 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.
- **따라서, 권장되지 않는 방법이다.**

<br><br>

## 빈 등록 초기화, 소멸 메서드 지정

### 예시 코드

```java
public class NetworkClient {
	private String url;
	
	public NetworkClient() {
		System.out.println("생성자 호출, url = " + url);
	}

	public void setUrl(String url) {
		this.url = url;
	}

	//서비스 시작시 호출하고자 하는 메서드
	public void connect() {
		System.out.println("connect: " + url);
	}

	//비즈니스 로직으로서 호출하고자 하는 메서드
	public void call(String message) {
		System.out.println("call: " + url + " message = " + message);
	}

	//서비스 종료시 호출하고자 하는 메서드
	public void disConnect() {
		System.out.println("close + " + url);
	}

	//빈 초기화할때 호출할 메서드로 설정할 메서드 (개발자가 직접 작성한 임의의 메서드)
	public void init() {
		System.out.println("NetworkClient.init");
		connect();
		call("초기화 연결 메시지");
	}

	//빈 종료할때 호출할 메서드로 설정할 메서드 (개발자가 직접 작성한 임의의 메서드)
	public void close() {
		System.out.println("NetworkClient.close");
		disConnect();
	}

}
```

<br>

### 테스트 코드

```java
public class BeanLifeCycleTest {

	@Test
	public void lifeCycleTest() {
		ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
		NetworkClient client = ac.getBean(NetworkClient.class);
		ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
	}

	@Configuration
	static class LifeCycleConfig {

		//초기화, 종료 메서드 설정
		@Bean(initMethod = "init" , destroyMethod = "close")
		public NetworkClient networkClient() {
			NetworkClient networkClient = new NetworkClient();
			networkClient.setUrl("http://hello-spring.dev");
			return networkClient;
		}
		//위 메서드가 실행된 후, 빈이 등록된다.
		//이때, close 메서드가 동작한다.
	}

}
```

- `@Bean( initMethod = "초기화할 내용이 담겨있는 메서드 이름" , destroyMethod = "종료할 내용이 담겨있는 메서드 이름" )`
    - 해당 애너테이션을 이용하여 초기화, 종료 메서드를 지정한다.

<br>

### 설정 정보 사용 특징

- 메서드 이름을 자유롭게 줄 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 코드가 아니라 설정 정보를 사용하게 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.

<br>

### 종료 메서드 추론

- 라이브러리는 대부분 `close` , `shutdown` 이라는 이름의 종료 메서드를 사용한다.

- `@Bean` 의 `destroyMethod` 는 기본값이 `(inferred)` (추론)으로 등록되어 있다.
    - 이러한 추론 기능은 `close` , `shutdown` 라는 이름의 메서드를 자동으로 호출해준다.
    - 말그대로 추론기능이다.
- 따라서 **직접 스프링 빈으로 등록하면 종료메서드는 따로 적어주지 않아도 잘 동작한다.**

<br><br>

## `@PostConstructor` , `@PreDestroy` 애노테이션 방식 - 권장방식

### 예시 코드

```java
public class NetworkClient {
	private String url;
	
	public NetworkClient() {
		System.out.println("생성자 호출, url = " + url);
	}

	public void setUrl(String url) {
		this.url = url;
	}

	//서비스 시작시 호출하고자 하는 메서드
	public void connect() {
		System.out.println("connect: " + url);
	}

	//비즈니스 로직으로서 호출하고자 하는 메서드
	public void call(String message) {
		System.out.println("call: " + url + " message = " + message);
	}

	//서비스 종료시 호출하고자 하는 메서드
	public void disConnect() {
		System.out.println("close + " + url);
	}

	//빈 초기화할때 호출할 메서드로 설정할 메서드 (개발자가 직접 작성한 임의의 메서드)
	//@PostConstructor 를 통해 설정한다.
	@PostConstructor
	public void init() {
		System.out.println("NetworkClient.init");
		connect();
		call("초기화 연결 메시지");
	}

	//빈 종료할때 호출할 메서드로 설정할 메서드 (개발자가 직접 작성한 임의의 메서드)
	//@PreDestroy 를 통해 설정한다.
	@PreDestroy
	public void close() {
		System.out.println("NetworkClient.close");
		disConnect();
	}

}
```

<br>

### 테스트 코드

```java
public class BeanLifeCycleTest {

	@Test
	public void lifeCycleTest() {
		ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
		NetworkClient client = ac.getBean(NetworkClient.class);
		ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
	}

	@Configuration
	static class LifeCycleConfig {

		@Bean
		public NetworkClient networkClient() {
			NetworkClient networkClient = new NetworkClient();
			networkClient.setUrl("http://hello-spring.dev");
			return networkClient;
		}
		//위 메서드가 실행된 후, 빈이 등록된다.
		//이때, close 메서드가 동작한다.
	}

}
```

- `@PostConstructor` , `@PreDestroy` 을 사용하면 가장 편리하게 초기화와 종료를 실행할 수 있다.

<br>

### `@PostConstructor` , `@PreDestroy` 애노테이션 특징

- 최신 스프링에서 **가장 권장하는 방법** 이다.
- 애노테이션 하나만 붙이면 되므로 매우 편리하다.
- 자바 표준이기 때문에, 다른 컨테이너에서도 잘 동작한다.
- 컴포넌트 스캔과 잘 어울린다.
- 외부 라이브러리에는 적용할 수 없다. (이땐 `@Bean` 을 통해 설정하자.)

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*