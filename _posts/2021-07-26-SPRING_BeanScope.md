---
category: Spring
tags: [스프링, 핵심원리, 빈스코프, 프로토타입빈]
title: "[스프링 - 핵심원리] 빈 스코프"
date:   2021-07-26 16:30:00 
lastmod : 2021-07-26 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>


Title: "빈 스코프"

# 빈 스코프

## 빈 스코프란?

빈 객체가 존재할 수 있는 범위를 뜻한다.

<br><br>

## 빈 스코프의 종류

- **싱글톤**

    - 기본 스코프
    - 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- 프로토타입
    - 스프링 컨테이너가 **프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하는 않는 매우 짧은 범위의 스코프**
- 웹 관련 스코프
    - request
        - 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
    - session
        - 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
    - application
        - 웹 서블릿 컨텍스와 같은 범위로 유지되는 스코프
	> 다음 게시글에서 다룬다.

<br>

## 빈 스코프 지정하기

`@Scope("빈스코프")` 애너테이션을 통해 빈 스코프를 설정할 수 있다.

<br>

### 컴포넌트 스캔을 활용한 자동 등록시, 지정하는 방법

```java
@Scope("prototype")
@Component
public class HelloBean {}
```

<br>

### 수동 등록시, 지정하는 방법

```java
@Scope("prototype")
@Component
public class HelloBean {}
```

<br><br>

## 프로토타입 스코프

프로토타입 스코프를 **스프링 컨테이너에서 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환** 한다.

<br>

### 싱글톤 조회

![싱글톤 조회](/assets/img/2021-07-26-SPRING_BeanScope/Untitled%2015.png)

<br>

### 프로토타입 빈 요청 - 1

![프로토타입 빈 1](/assets/img/2021-07-26-SPRING_BeanScope/Untitled%2016.png)

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.

<br>

### 프로토타입 빈 요청 - 2

![프로토타입 빈 2](/assets/img/2021-07-26-SPRING_BeanScope/Untitled%2017.png)

1. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
2. 이후에 스프링 컨테이너에 같은 요청이 오면 **항상 새로운 프로토타입 빈을 생성해서 반환** 한다.

> 스프링 컨테이너는 **프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리**한다!!!!  
( 따라서, `@PreDestroy` 같은 종료 메서드가 호출되지 않는다! )

<br><br><br>

# 싱글톤 빈과 프로토타입 빈 함께 사용하기

## '프로토타입 빈', '싱글톤 빈' 함께 사용시 문제점

만약 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용한다면 어떻게 될까?

**'프로토타입 빈'은 '싱글톤 빈'에 주입될 때만 생성되고, 이후로는 생성되지 않는다.** 왜냐하면, 클라이언트(고객)는 항상 '싱글톤 빈'을 통해 '프로토타입 빈'을 만나기 때문이다. 즉, **'프로토타입 빈'은 주입 시점에 스프링 컨테이너에 요청해서 새로 생성되어 주입된 것이고, '싱글톤 빈'이 이 빈을 계속 가지고있다.** '프로토타입 빈'을 사용할 때마다 새로 생성되는 것은 아니다!

> 참고로, 여러 '싱글톤 빈'이 같은 '프로토타입 빈'을 주입받으면, 주입 받는 시점에 각각 새로운 '프로토타입 빈'이 생성되어 주입된다.

그러면, 프로토타입 빈을 주입시점 뿐만 아니라, 사용할 때마다 새로 생성하는 방법은 없을까?

<br><br>

## ObjectFactory, ObjectProvider

- 프로토타입 빈을 사용할 때마다 새로 생성받으려면, **스프링 컨테이너에 새로 요청** 하면 된다.

    - 즉, 프로토타입 빈을 조회하면 프로토타입 빈이 새로 생성되므로, **싱글톤 빈을 사용할 때마다 프로토타입 빈을 조회하면 된다.**
- `ObjectProvider`
    - 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider` 이다.
    - `ObjectFactory` 에 편의 기능을 추가한 것이다.

    > DL (Dependency Lookup) 이란?  
    의존관계를 외부에서 주입(DI) 받는 것이 아닌, 직접 필요한 의존관계를 찾는 것

예제 코드로 확인해보자.

<br>

### PrototypeBean

- `PrototypeBean` 은 프로토타입 빈이다.

```java
@Scope("prototype")
public class PrototypeBean {

	private int count = 0;

	public void addCount() {
		count++;
	}

	public int getCount() {
		return count;
	}

	@PostConstruct
	public void init() {
		System.out.println("PrototypeBean.init " + this);
	}

	@PreDestroy
	public void destroy() {
		System.out.println("PrototypeBean.destroy");
	}

}
```

<br>

### ClientBean

- `ClientBean` 은 싱글톤 빈이며, '프로토타입 빈'을 주입받는다.

```java
public class ClientBean {
	
	//@Autowired
	//private ApplicationContext ac;

	@Autowired
	private ObjectProvider<PrototypeBean> prototypeBeanProvider;
	
	public int logic() {
		// 직접 프로토타입 빈을 조회한다.
		// PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);

		// 위 코드를 대체한다.
		PrototypeBean prototypeBean = prototypeBeanProvider.getObject();

		prototypeBean.addCount();
		int count = prototypeBean.getCount();
		return count;
	}

}
```

- `prototypeBeanProvider.getObject()`
    - 항상 새로운 프로토타입 빈이 생성되어 반환한다.
    - 내부적으로 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (DL)

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*