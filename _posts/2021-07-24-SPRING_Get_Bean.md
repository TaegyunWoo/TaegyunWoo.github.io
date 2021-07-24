---
category: Spring
tags: [스프링, 핵심원리, Bean, getBean, getBeansOfType]
title: "[스프링 - 핵심원리] 스프링 빈 조회의 방법들"
date:   2021-07-24 20:30:00 
lastmod : 2021-07-24 20:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

본 게시글은 테스트 코드만을 다룬다. 이미 작성된 코드들 (`AppConfig` , `MemberService` , `MemberServiceImpl` 등)은 [이전 게시글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP)을 참고하면 된다.

또한, JUnit 프레임워크를 통해 테스트 코드를 작성하므로, 관련 지식이 부족하다면 자료를 찾아보시길 바란다.

<br><br>

# 기본적인 방법으로 조회

## getBean() 메서드

### 빈 이름으로 조회하기

- `getBean(빈이름, 빈타입)`
    - 빈이름: 조회할 빈의 이름을 뜻한다.
    - 빈타입: 조회할 빈의 타입을 뜻한다. 다형성을 활용하기 위해 보통 인터페이스 타입으로 조회한다.

```java
import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.*;

class ApplicationContextBasicFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	@Test
	@DisplayName("빈 이름으로 조회")
	void findBeanByName() {
		
		//빈 이름으로 조회하기
		MemberService memberService = ac.getBean("memberService", MemberService.class);

		assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	}

}
```

<br>

### 빈 타입만으로 조회하기

- `getBean(빈타입)`
    - 해당 타입을 가지는 빈 객체를 반환한다.
- 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다.
    - 따라서, 빈이름을 지정해야한다.

```java
import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.*;

class ApplicationContextBasicFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	@Test
	@DisplayName("이름없이 타입만으로 조회")
	void findBeanByType() {
		
		//빈 타입만으로 조회하기
		MemberService memberService = ac.getBean(MemberService.class);

		assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	}

}
```

<br>

### 빈 구체 타입으로 조회하기

- `getBean(빈이름, 구체타입)`
    - 인터페이스 타입이 아닌 구체 타입으로 조회한다.
    - 보통 이런 방식으로 사용하지는 않는다.
    - 왜냐하면, 다형성을 사용할 수 없기 때문에 변경시 유연성이 떨어지기 때문이다.

```java
import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.*;

class ApplicationContextBasicFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	@Test
	@DisplayName("이름과 구체 타입으로 조회")
	void findBeanByName2() {
		
		//빈 구체타입으로 조회하기
		MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);

		assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	}

}
```

<br>

### 존재하지 않는 빈 조회하기

- `NoSuchBeanDefinitionException` 예외
    - 존재하지 않는 빈 이름을 조회하면 해당 예외가 발생한다.

```java
import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.*;

class ApplicationContextBasicFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	@Test
	@DisplayName("존재하지 않는 빈 조회")
	void findBeanByNameX() {
		
		//빈 조회
		Assertions.assertThrows(NoSuchBeanDefinitionException.class ,
				() -> ac.getBean("Xxxx", MemberService.class) );
	}

}
```

<br><br>

# 동일한 타입이 둘 이상일때 조회

## 전제

- `MemberRepository` 라는 인터페이스의 구현 클래스가 `MemoryMemberRepository` 클래스와 `DbMemberRepository` 클래스라고 하자.

- `MemoryMemberRepository`
    - memberRepository1 라는 이름으로 빈 등록되었다고 가정한다.
- `DbMemberRepository`
    - memberRepository2 라는 이름으로 빈 등록되었다고 가정한다.

```java
@Configuration
class SameBeanConfig {
	@Bean
	public MemberRepository memberRepository1() {
		return new MemoryMemberRepository();
	}

	@Bean
	public MemberRepository memberRepository2() {
		return new DbMemberRepository();
	}	
}
```

<br><br>

## getBean() 메서드

### 타입만으로 조회하기

- `NoUniqueBeanDefinitionException` 예외
    - 타입만으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다.

```java
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextSameBeanFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
	
	@Test
	@DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다")
	void findBeanByTypeDuplicate() {
		//예외 발생 코드
		//DiscountPolicy bean = ac.getBean(MemberRepository.class);
		
		assertThrows(NoUniqueBeanDefinitionException.class, () ->
		ac.getBean(MemberRepository.class)); //인터페이스 타입으로 조회
	}

}
```

<br>

### 빈 이름, 빈 타입으로 조회하기

```java
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextSameBeanFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
	
	@Test
	@DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
	void findBeanByName() {
		MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
		
		assertThat(memberRepository).isInstanceOf(MemberRepository.class);
	}

}
```

<br>

## getBeansOfType() 메서드

### 특정 타입을 모두 조회하기

```java
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextSameBeanFindTest {
	
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
	
	@Test
	@DisplayName("특정 타입을 모두 조회하기")
	void findAllBeanByType() {
		Map<String, MemberRepository> beansOfType = 
				ac.getBeansOfType(MemberRepository.class);

		// beansOfType 에 담긴 모든 요소 출력
		for(String key : beansOfType.keySet()) {
			System.out.println("key = " + key + " value = " + beansOfType.get(key));
		}
		
		assertThat(beansOfType.size()).isEqualTo(2);
	}

}
```

<br><br>

# 상속 관계 조회

- 부모 타입으로 조회하면, 자식 타입도 함께 조회한다.
- `Object` 타입으로 조회하면, 모든 스프링 빈을 조회하게 된다.
    - `Object` 는 모든 자바 객체의 최고 부모이기 때문이다.

<br>

## 전제

- `MemberRepository` 라는 인터페이스의 구현 클래스가 `MemoryMemberRepository` 클래스와 `DbMemberRepository` 클래스라고 하자.
- `MemoryMemberRepository`
    - memberRepository1 라는 이름으로 빈 등록되었다고 가정한다.
- `DbMemberRepository`
    - memberRepository2 라는 이름으로 빈 등록되었다고 가정한다.

```java
@Configuration
class SameBeanConfig {
	@Bean
	public MemberRepository memberRepository1() {
		return new MemoryMemberRepository();
	}

	@Bean
	public MemberRepository memberRepository2() {
		return new DbMemberRepository();
	}	
}
```

<br><br>

## 부모 타입으로 조회

### 타입으로만 조회 with `getBean()`

- 부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다.

```java
import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextExtendsFindTest {

	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

	@Test
	@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다")
	void findBeanByParentTypeDuplicate() {
		//DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
		
		assertThrows(NoUniqueBeanDefinitionException.class, () ->
		ac.getBean(DiscountPolicy.class));
	}

}
```

<br>

### 빈 이름과 타입으로 조회 with `getBean()`

- 부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름까지 지정하면 된다.

```java
import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextExtendsFindTest {

	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

	@Test
	@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 이름 지정하면 된다.")
	void findBeanByParentTypeBeanName() {
		MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
		assertThat(memberRepository).isInstanceOf(MemberRepository.class);
	}

}
```

<br>

### 부모 타입으로 모두 조회 with `getBeansOfType()`

```java
import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextExtendsFindTest {

	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

	@Test
	@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 이름 지정하면 된다.")
	void findAllBeanByParentType() {

		//MemberRepository 타입을 갖는 모든 빈 객체를 Map으로 반환받음
		Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);

		//beansOfType에 들어있는 모든 요소 출력
		for(String key : beansOfType.keySet) {
			System.out.println("key = " + key + " value = " + beansOfType.get(key));
		}

		assertThat(beansOfType.size()).isEqualTo(2);
	}

}
```

<br><br>

## 특정 하위 타입으로 조회

### 특정 하위 타입으로 조회 with `getBean()`

```java
import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ApplicationContextExtendsFindTest {

	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

	@Test
	@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 이름 지정하면 된다.")
	void findBeanBySubType() {

		//MemoryMemberRepository 타입으로 조회함
		MemberRepository memberRepository = ac.getBean("memberRepository1", MemoryMemberRepository.class);
		assertThat(memberRepository).isInstanceOf(MemberRepository.class);
	}

}
```

<br>

---

<br>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*

  [인프런 강의로 가기](https://inf.run/pcN8)