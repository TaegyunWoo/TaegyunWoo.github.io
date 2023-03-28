---
category: Spring-Core
tags: [Spring-Core]
title: "[스프링 - 핵심원리] 스프링 컨테이너의 구조와, 다양한 설정 형식 지원의 원리"
date:   2021-07-24 21:30:00 
lastmod : 2021-07-24 21:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 스프링 컨테이너의 구조

## 스프링 컨테이너의 특징

- 스프링 컨테이너는 다양한 형식의 설정 정보 (예. AppConfig) 를 받아들일 수 있게 유연하게 설계되어있다.

<br><br>

## 스프링 컨테이너의 구조

![스프링 컨테이너 구조](/assets/img/2021-07-24-SPRING_Container_Structure/Untitled%208.png)

- `AnnotationConfigApplicationContext`
    - 자바코드를 이용한 설정 정보
- `GenerixXmlApplicationContext`
    - XML을 이용한 설정 정보

<br><br>

## 애노테이션 기반 자바 코드 설정 사용

- `new AnnotationConfigApplicationContext` (`AppConfig.class`)
- `AnnotationConfigApplicationContext` 클래스를 사용하면서 자바 코드로된 설정 정보를 넘기면 된다.

<br><br>

## XML 설정 사용

- 최근에는 스프링 부트를 많이 사용하면서 **XML 기반의 설정은 잘 사용하지 않는다.**
- 하지만 레거시 프로젝트 수행, 컴파일 없이 빈 설정 정보 변경 등의 이유로 알아두는 것이 권장된다.

<br>

### XML 기반의 스프링 빈 설정 정보 파일

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http:// www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="memberService" class="hello.core.member.MemberServiceImpl">
		<constructor-arg name="memberRepository" ref="memberRepository" />
	</bean>

	<bean id="memberRepository"
		class="hello.core.member.MemoryMemberRepository" />
	
	<bean id="orderService" class="hello.core.order.OrderServiceImpl">
		<constructor-arg name="memberRepository" ref="memberRepository" /> <constructor-arg name="discountPolicy" ref="discountPolicy" />
	</bean>

	<bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy" />

</beans>
```

<br>

### XML 설정 테스트 코드

```java
import hello.core.member.MemberService;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

import static org.assertj.core.api.Assertions.*;

public class XmlAppContext {

	@Test
	void xmlAppContext() {
		ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
		MemberService memberService = ac.getBean("memberService", MemberService.class);
		assertThat(memberService).isInstanceOf(MemberService.class);
	}

}
```

<br><br>

# 스프링 빈 설정 메타 정보

## BeanDefinition 이란?

- 스프링이 다양한 설정 형식을 지원할 수 있게 하는 빈 설정 메타정보이다.
- 즉, 스프링은 역할과 구현을 개념적으로 나누어 다양한 설정 형식을 지원할 수 있다.
- 스프링 컨테이너는 설정 형식이 '자바코드'인지, 'XML'인지 몰라도 된다.
오직 `BeanDefinition` 만 알면 된다.
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성(등록)한다.

![BeanDefinition](/assets/img/2021-07-24-SPRING_Container_Structure/Untitled%209.png)

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*