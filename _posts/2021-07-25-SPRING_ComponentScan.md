---
category: Spring
tags: [스프링, 핵심원리, 컴포넌트스캔, Autowired]
title: "[스프링 - 핵심원리] 컴포넌트 스캔"
date:   2021-07-25 14:30:00 
lastmod : 2021-07-25 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 컴포넌트 스캔과 의존관계 자동 주입

- 기존에는 `@Bean` 애너테이션이나 XML의 <bean> 태그를 통해서 스프링 빈 객체를 등록하였다.

- 이러한 방식은 빈의 개수가 증가하면 번거롭다.
- 따라서, 스프링이 제공하는 기능인 **컴포넌트 스캔** 을 사용하여 좀더 편리하게 빈을 등록할 수 있다.
- 또한, 의존관계도 자동으로 주입하는 `@Autowired` 라는 기능도 제공한다.

<br><br>

## 애너테이션 정리

- `@ComponentScan`
    - `@Component` 가 붙은 클래스를 빈 객체로 등록한다.
- `@Component`
    - 해당 클래스를 스프링 빈으로 등록할 수 있도록 유도한다.
- `@Autowired`
    - 해당 생성자의 매개변수에 자동으로 의존관계를 주입한다(DI).

<br><br>

## 컴포넌트 스캔 적용 코드

### 컴포넌트 스캔이 적용된 `AppConfig`

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import static org.springframework.context.annotation.ComponentScan.*;

@Configuration
@ComponentScan(
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)

public class AutoAppConfig {

}
```

- `@ComponentScan` 애너테이션을 설정 정보 ( `AutoAppConfig` )에 붙여주면 된다.

- 컴포넌트 스캔은 `@Component` **애너테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록** 한다.

> `excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class)` 해당 옵션은 이전 게시글에서 작성한 예제 코드 ( `AppConfig` )를 남기기 위해 추가한 옵션이다.

<br>

### 컴포넌트로 지정한 `MemoryMemberRepository` 클래스

> 기존의 `MemoryMemberRepository` 클래스의 코드는 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고.

```java
@Component
public class MemoryMemberRepository implements MemberRepository {

}
```

- `@Component` 애너테이션을 사용하여, `@ComponentScan` 애너테이션이 붙은 설정 정보 ( `AutoAppConfig` )가 `MemoryMemberRepository` 를 스캔하여 빈으로 등록할 수 있도록 한다.

<br>

### 컴포넌트로 지정한 `RateDiscountPolicy` 클래스

> `RateDiscountPolicy` 클래스는 할인정책 관련 구체 클래스이다. `DiscountPolicy` 인터페이스를 구체화한 것이다.

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {

}
```

- `@Component` 애너테이션을 사용하여, `@ComponentScan` 애너테이션이 붙은 설정 정보 ( `AutoAppConfig` )가 `MemoryMemberRepository` 를 스캔하여 빈으로 등록할 수 있도록 한다.

<br>

### 의존관계가 자동으로 주입되는 `MemberServiceImpl` 클래스

> 기존의 `MemberServiceImpl` 클래스의 코드는 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP) 참고.

```java
@Component
public class MemberServiceImpl implements MemberService {
	
	private final MemberRepository memberRepository;

	@Autowired
	public MemberServiceImpl(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}

}
```

- `@Autowired` 애너테이션을 통해 의존관계를 자동으로 주입해준다.

<br>

### 의존관계가 자동으로 주입되는 `OrderServiceImpl` 클래스

> 기존의 `OrderServiceImpl` 클래스의 코드는 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_Configuration) 참고.

```java
@Component
public class OrderServiceImpl implements OrderService {
	private final MemberRepository memberRepository;
	private final DiscountPolicy discountPolicy;

	@Autowired
	public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
	}

}
```

- `@Autowired` 애너테이션을 통해 의존관계를 자동으로 주입해준다.
- 여러 의존관계 역시 한번에 주입받을 수 있다.

<br>

### 테스트 코드

```java
import hello.core.AutoAppConfig;
import hello.core.member.MemberService;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.*;

public class AutoAppConfigTest {

	@Test
	void basicScan() {
		ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
		MemberService memberService = ac.getBean(MemberService.class);
		assertThat(memberService).isInstanceOf(MemberService.class);
	}

}
```

- `MemberService` 의 구체 객체가 정상적으로 빈으로 생성된 것을 확인할 수 있다.

<br><br>

## 상세 동작 원리

### `@ComponentScan`

![@ComponentScan 동작 원리](/assets/img/2021-07-25-SPRING_ComponentScan/Untitled%2013.png)

- `@ComponentScan` 은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 이때 스프링 빈의 기본 이름은 클래스명을 사용하되, **맨 앞글자만 소문자를 사용** 하여 빈을 등록한다.
    - Ex) MemberServiceImpl 클래스 ⇒ memberServiceImpl
- 빈 이름을 직접 설정하고 싶다면, `@Component("원하는빈이름")` 을 사용하면 된다.

<br>

### `@Autowired` 의존관계 자동 주입

![@Autowired](/assets/img/2021-07-25-SPRING_ComponentScan/Untitled%2014.png)

- 생성자에 `@Autowired` 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 기본적인 조회 전략은 **타입이 같은 빈을 찾아서 주입하는 것** 이다.
    - 즉, 위 그림에서 빈 객체 `memoryMemberRepository` 의 타입이 `MemoryMemberRepository` 이자, `MemberRepository` 이므로 해당 빈이 주입되게 된다.
- 여러 파라미터가 있더라도 다 찾아서 자동으로 주입한다.

<br><br><br>

# 탐색 위치와 기본 스캔 대상

## 탐색할 패키지의 시작 위치 지정

- 스프링 컨테이너가 모든 자바 클래스를 대상으로 컴포넌트 스캔을 수행하면 시간이 오래 걸리게 된다.
- 따라서, 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

<br>

### 탐색 시작 위치 지정 설정

```java
@ComponentScan(
	basePackages = {"원하는 패키지명1", "원하는 패키지명2", .. , "원하는 패키지명n"},
)
```

- `basePackages`
    - 탐색할 패키지의 시작 위치를 지정한다.
    - 해당 패키지를 포함한 모든 하위 패키지를 탐색한다.
- `basePackageClasses`
    - 해당 속성은 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
- 만약 시작 위치를 지정하지 않으면, `@ComponentScan` 이 붙은 설정 정보 클래스의 패키지가 시작 위치로 설정된다.

<br>

### 탐색 시작 위치를 설정하는 좋은 방법

- 탐색 시작 패키지를 따로 지정하기보단, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 방법을 권장한다.

<br><br>

## 컴포넌트 스캔 기본 대상

- `@ComponentScan` 은 `@Component` 뿐만 아니라 아래의 내용도 추가로 대상에 포함한다.
    - `@Component` : 컴포넌트 스캔에서 사용
    - `@Controller` : 스프링 MVC 컨트롤러에서 사용
    - `@Service` : 스프링 비즈니스 로직에서 사용
    - `@Repository` : 스프링 데이터 접근 계층에서 사용
    - `@Configuration` : 스프링 설정 정보에서 사용

<br><br>

## 애너테이션들의 부가 기능

- `@Controller` : 스프링 MVC 컨트롤러로 인식
- `@Service` : 아무런 처리를 하지는 않는다. 단지, 개발자들에게 핵심 비즈니스 로직이 여기 있다는 것을 명시해준다.
- `@Repository` : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
- `@Configuration` : 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다.

<br><br><br>

# 중복 등록과 충돌

## 컴포넌트 스캔에서 발생할 수 있는 상황

컴포넌트 스캔시 같은 빈 이름을 등록했을 때, 발생할 수 있는 상황은 아래와 같다.

- 자동 빈 등록 vs 자동 빈 등록
- 수동 빈 등록 vs 자동 빈 등록

<br><br>

## 자동 빈 등록 vs 자동 빈 등록

컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, **그 이름이 같은 경우 스프링은 오류를 발생** 시킨다.

<br><br>

## 수동 빈 등록 vs 자동 빈 등록

### 예시 코드

```java
//빈 객체로 등록할 클래스

@Component
public class MemoryMemberRepository implements MemberRepository {

}
```

- 해당 클래스에 `@Component` 가 붙어있으므로, `@ComponentScan` 에 의해 자동으로 빈 등록이 된다.

- 즉, 자동 빈 등록이다.

```java
//설정 정보 클래스

@Configuration
@ComponentScan
public class AutoAppConfig {

	//빈 수동 등록	
	@Bean(name = "memoryMemberRepository")
	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}

}
```

- `AutoAppConfig` 클래스 내부에 `@Bean` 애너테이션을 사용하여 memberRepository 빈 객체를 수동으로 등록한다.

- 즉, 수동 빈 등록이다.

<br>

### 중복 결과

이 경우 **수동 빈 등록이 우선권** 을 가진다. 즉, **수동 빈이 자동 빈을 오버라이딩** 해버린다.

> 이러한 결과를 의도적으로 설계했다면 괜찮지만, 그것이 아니라면 이러한 문제 (빈 객체가 오버라이딩 되는 문제)를 디버깅하기란 쉽지 않다.

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*