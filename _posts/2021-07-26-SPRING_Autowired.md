---
category: Spring
tags: [스프링, 핵심원리, autowired, 충돌]
title: "[스프링 - 핵심원리] @Autowired 와 충돌문제"
date:   2021-07-26 12:30:00 
lastmod : 2021-07-26 12:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# `@Autowired` - 조회 빈이 2개 이상일때

## 문제 상황

- `@Autowired` 는 타입(Type)을 기반으로 조회한다.

- 그렇다면, 타입이 같은 빈 객체들이 존재한다면 어떻게 될까?
    - 충돌이 발생하여, `NoUniqueBeanDefinitionException` 예외를 발생시킨다.
- 이 문제를 어떻게 해결할까.

<br><br>

## 조회 대상 빈이 2개 이상일 때의 해결 방안

- `@Autowired` 필드명 매칭
- `@Qualifier` → `@Qualifier` 끼리 매칭 → 빈 이름 매칭
- `@Primary` 사용

위와 같이 총 3가지의 해결 방법이 있다. 하나씩 알아보자.

<br><br>

## `@Autowired` 필드명 매칭

- `@Autowired` 는 타입 매칭을 시도하고, **이때 여러 빈이 있으면 '필드 이름'과 '파라미터 이름'으로 빈 이름을 추가 매칭** 한다.
- **필드에 직접 주입하는 방식으로 수행한다.**

<br>

### 타입이 같은 빈 객체 등록 1

- 빈 객체 이름을 `rateDiscountPolicy` 로 등록한다.
- 타입은 `DiscountPolicy` 이다.

```java
@Component
public class RateDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 타입이 같은 빈 객체 등록 2

- 빈 객체 이름을 `fixDiscountPolicy` 로 등록한다.
- 타입은 `DiscountPolicy` 이다. 위와 **동일한 타입** 이다.

```java
@Component
public class FixDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 의존관계 주입

- `DiscountPolicy` 타입인 `rateDiscountPolicy` 빈과 `fixDiscountPolicy` 빈 중 `rateDiscountPolicy` 빈을 `OrderServiceImpl` 에 주입한다.

```java
@Component
public class OrderServiceImpl implements OrderService {

	//의존관계 주입 (rateDiscountPolicy 빈을 OrderServiceImpl에 주입한다.)
	/*
		타입이 같은 빈이 2개이므로, 빈 이름이 rateDiscountPolicy인 것(필드명과 빈이름이 같은 것)을
		찾아 주입한다.
	*/
	@Autowired
	private DiscountPolicy rateDiscountPolicy;

	// .. 이하 생략
}
```

<br><br>

## `@Qualifier` 사용

- `@Qualifier` 는 추가 구분자를 붙여주는 방법이다.

- 주입시 추가적인 방법을 제공하는 것이지, **빈 이름을 변경하는 것은 아니다.**

<br>

### 타입이 같은 빈 객체 등록 1

- 빈 객체 이름을 `rateDiscountPolicy` 로 등록한다.

- 타입은 `DiscountPolicy` 이다.
- **추가 구분자를 "mainDiscountPolicy"로 설정한다.**

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 타입이 같은 빈 객체 등록 2

- 빈 객체 이름을 `fixDiscountPolicy` 로 등록한다.

- 타입은 `DiscountPolicy` 이다. 위와 **동일한 타입** 이다.
- **추가 구분자를 "fixDiscountPolicy"로 설정한다.**

```java
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 생성자 주입

- 매개변수 `discountPolicy` 에 '추가 구분자를 통해 선택된 빈 객체 `rateDiscountPolicy` '를 넘긴다.

```java
@Component
public class OrderServiceImpl implements OrderService {

	private final DiscountPolicy discountPolicy;
	
	@Autowired
	public OrderServiceImpl(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
	
	// .. 이하 생략
}
```

<br>

### setter 주입

- 매개변수 `discountPolicy` 에 '추가 구분자를 통해 선택된 빈 객체 `rateDiscountPolicy` '를 넘긴다.

```java
@Component
public class OrderServiceImpl implements OrderService {

	private DiscountPolicy discountPolicy;
	
	@Autowired
	public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
		return discountPolicy;
	}
	
	// .. 이하 생략
}
```

<br>

### `@Qualifier` 에서 추가 구분자를 못찾을 경우

- `@Qualifier` 로 설정된 추가 구분자 값을 못찾을 때는 어떻게 할까?

- 이때는 설정된 추가 구분자 값을 **이름으로 갖는 빈을 찾는다.**
- 예시
    1. `@Qualifier("MyValue")` 를 통해 "MyValue" 라는 추가 구분자를 갖는 빈을 찾는다.
    2. 하지만, `@Qualifier("MyValue")` 로 설정된 빈 객체가 없다.
    3. 그렇다면, "MyValue" 라는 이름을 갖는 빈 객체를 찾는다.
    4. 만약, "MyValue" 라는 이름을 갖는 빈 객체 역시 없다면, `NoSuchBeanDefinitionException` 이 발생한다.

<br><br>

## `@Primary` 사용

- `@Primary` 는 우선순위를 정하는 방법이다.

- `@Autowired` 시에 여러 빈이 매칭되면 `@Primary` 가 우선권을 가진다.

<br>

### 타입이 같은 빈 객체 등록 1

- 빈 객체 이름을 `rateDiscountPolicy` 로 등록한다.
- 타입은 `DiscountPolicy` 이다.
- **우선권을 가진다.**

```java
@Component
@Primary
public class RateDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 타입이 같은 빈 객체 등록 2

- 빈 객체 이름을 `fixDiscountPolicy` 로 등록한다.
- 타입은 `DiscountPolicy` 이다. 위와 **동일한 타입** 이다.
- **우선권을 갖지 않는다.**

```java
@Component
public class FixDiscountPolicy implements DisocuntPolicy {
	//... 이하 생략
}
```

<br>

### 생성자 주입

- 매개변수 `discountPolicy` 에 '우선권을 가진 빈 객체 `rateDiscountPolicy` '를 넘긴다.

```java
@Component
public class OrderServiceImpl implements OrderService {

	private final DiscountPolicy discountPolicy;
	
	//생성자
	@Autowired
	public OrderServiceImpl(DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
		
	// .. 이하 생략
}
```

<br>

### setter 주입

- 매개변수 `discountPolicy` 에 '우선권을 가진 빈 객체 `rateDiscountPolicy` '를 넘긴다.

```java
@Component
public class OrderServiceImpl implements OrderService {

	private DiscountPolicy discountPolicy;
	
	@Autowired
	public DiscountPolicy setDiscountPolicy(DiscountPolicy discountPolicy) {
		return discountPolicy;
	}
	
	// .. 이하 생략
}
```

<br><br>

## `@Qualifier` vs `@Primary`

### 우선권

- 만약 `@Qualifier` 와 `@Primary` 가 모두 붙었다면 어떻게 작동할까?
- **더 구체적으로 설정된** `@Qualifier` **가** `@Primary` **보다 우선권을 갖는다.**

### 활용

- `@Qualifier`
    - 가끔 사용되는 빈일 때 사용
- `@Primary`
    - 자주 사용되는 빈일 때 사용

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*