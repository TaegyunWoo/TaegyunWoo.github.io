---
category: Spring
tags: [스프링, 핵심원리, OCP, DIP, SRP]
title: "[스프링 - 핵심원리] OCP와 DIP 그리고 SRP"
date:   2021-07-23 11:00:00 
lastmod : 2021-07-23 11:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 들어가며..

스프링의 핵심적인 개념은 바로 **OCP(개방-폐쇄 원칙)** 와 **DIP(의존 역전 원칙)** 그리고 **SRP(단일 책임 원칙)** 이다. 이들은 모두 객체지향 설계 원칙인 SOLID의 일부이다. 해당 원칙들을 지키기 위해 만들어진 것이 바로 스프링 프레임워크이다. 따라서, 스프링을 본격적으로 공부하기 전에 해당 개념들을 정리하고자 한다.

간단한 회원 도메인을 통해 이러한 개념들을 익힐 수 있도록 한다.

<br><br>

# 회원 도메인 설계

## 도메인 요구사항

- 회원을 가입하고 조회할 수 있다.
- 회원은 일반과 VIP 두 가지 등급이 있다.
- **회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)**

## 클래스 다이어그램

![기본 클래스 다이어그램](/assets/img/2021-07-23-SPRING_OCP_DIP/Untitled.png)

<br>

- MemberRepository 인터페이스의 구현체가 총 2개인데, 이들 중 어느것이라도 **SOLID원칙을 지키며 언제든 교체 가능**하도록 해야한다는 것이 핵심이다.

<br>

## 도메인 개발

### 회원 등급

```java
package hello.core.member;
public enum Grade {
	BASIC,
	VIP 
}
```

<br>

### 회원 엔티티

```java
package hello.core.member;

public class Member {
	private Long id;
	private String name;
	private Grade grade;
	public Member(Long id, String name, Grade grade) {
		this.id = id;
		this.name = name;
		this.grade = grade;
	}
	//getter, setter 생략
}
```

<br>

### 회원 저장소 인터페이스

```java
package hello.core.member;

public interface MemberRepository {
	void save(Member member);
	Member findById(Long memberId);
}
```

<br>

### 메모리 회원 저장소 구현체

```java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class MemoryMemberRepository implements MemberRepository { 
	
	private static Map<Long, Member> store = new HashMap<>(); @Override

	public void save(Member member) {
		store.put(member.getId(), member);
	}

	@Override
	public Member findById(Long memberId) {
		return store.get(memberId);
	}
}
```

<br>

### 회원 서비스 인터페이스

```java
package hello.core.member;

public interface MemberService {
	void join(Member member);
	Member findMember(Long memberId);
}
```

<br>

### 회원 서비스 구현체

```java
package hello.core.member;

public class MemberServiceImpl implements MemberService {
	private final MemberRepository memberRepository = new MemoryMemberRepository();

	public void join(Member member) {
		memberRepository.save(member);
	}
	
	public Member findMember(Long memberId) {
		return memberRepository.findById(memberId);
	}
}
```

<br>

## 도메인 설계의 문제점

- 만약 MemberRepository 인터페이스의 구현체를 바꿔야한다면 OCP를 위배한다.
    - `MemberServiceImpl`클래스의 `private final MemberRepository memberRepository = new MemoryMemberRepository();`를 `private final MemberRepository memberRepository = new DbMemberRepository();`로 **수정해야하기 때문**이다.
    - 즉, `MemberServiceImpl`클래스의 변경이 불가피하므로 OCP를 위배한다.

- 위 설계는 DIP와 SRP 원칙을 위배한다.
    - `MemberRepository`인터페이스와 이에 대한 구체화 클래스 `MemoryMemberRepository` 모두 종속되기 때문이다.
    (`private final MemberRepository memberRepository = new MemoryMemberRepository();`)
    - 즉, 인터페이스와 구체화 클래스가 모두 한 클래스 내에서 호출되기에 DIP와 SRP를 위배한다.
    - DIP와 SRP를 만족하려면 추상(인터페이스)에만 의존해야한다.

<br>

## 해결방법

### 관심사 분리

기존의 `MemberServiceImpl`클래스는 "객체 실행", "객체 생성&연결" 두가지의 역할을 모두 가지고 있었다 ( `private final MemberRepository memberRepository = new MemoryMemberRepository();` ). 이러한 역할을 다음과 같이 나누어 DIP와 OCP를 지킬 수 있다.

- 객체를 실행하는 것에만 집중하는 클래스
- 객체를 생성하고 연결하는 것에만 집중하는 클래스

위와 같이 클래스의 역할을 나누면 아래 클래스다이어그램처럼 변화한다.

<br>

![분할된 클래스 다이어그램](/assets/img/2021-07-23-SPRING_OCP_DIP/Untitled%201.png)

<br>

- MemberServiceImpl 클래스
  - **객체 실행** 에 집중하는 클래스
- AppConfig 클래스
  - **객체 생성 및 연결** 에 집중하는 클래스

이것을 코드화한다면 다음과 같다.

- 기존의 MemberServiceImpl 클래스

    ```java
    //기존의 MemberServiceImpl 클래스

    public class MemberServiceImpl implements MemberService {
    	private final MemberRepository memberRepository = new MemoryMemberRepository();

    	public void join(Member member) {
    		memberRepository.save(member);
    	}
    	
    	public Member findMember(Long memberId) {
    		return memberRepository.findById(memberId);
    	}
    }
    ```

    <br>

- 객체 실행에 집중하는 MemberServiceImpl 클래스

    ```java
    //실행에 집중하는 MemberServiceImpl 클래스

    public class MemberServiceImpl implements MemberService {
    	
    	//이제 객체를 생성하고 연결하는 역할이 사라졌다.
    	private final MemberRepository memberRepository;
    	
    	//생성자를 통해 MemberRepository 인터페이스의 구현 객체를 외부로부터 주입받는다.
    	//이것을 생성자 주입이라고 한다.
    	public MemberServiceImpl(MemberRepository memberRepository) {
    		this.memberRepository = memberRepository;
    	}

    	public void join(Member member) {
    		//기존 코드와 동일
    		memberRepository.save(member);
    	}

    	public Member findMember(Long memberId) {
    		//기존 코드와 동일
    		return memberRepository.findById(memberId);
    	}

    }
    ```

- 객체 생성 및 연결에 집중하는 AppConfig 클래스

    ```java
    //객체 생성 및 연결에 집중하는 AppConfig 클래스

    public class AppConfig {

    	//MemberServiceImpl 객체에 MemberRepository 인터페이스의 구현 객체를 주입해준다.
    	//이것을 의존성 주입이라고 한다. (DI: Dependency Injection)
    	public MemberService memberService() {
    		return new MemberServiceImpl(new MemoryMemberRepository());
    	}

    }
    ```

이와 같이 설계를 변경하면, `MemberServiceImpl` 클래스는 `MemoryMemberRepository` 구체 클래스를 의존하지 않게 된다. 단지, `MemberRepository` 인터페이스만 의존하게 된다. 따라서, `MemberServiceImpl` 클래스는 **실행** 에만 집중하면 된다. 어떤 구체 클래스의 객체를 주입할 것인지는 `AppConfig` 클래스가 담당하게 된다. 즉, **DIP를 만족** 한다.

또한, `MemberRepository` 의 구현 객체를 `MemoryMemberRepository` 가 아닌 `DbMemberRepository` 로 바꾼다고 했을 때 `MemberServiceImpl` 클래스를 수정할 필요가 없다. 단지, `AppConfig` 클래스의 `memberService` 메서드를 수정하면 된다. 이를 통해 Dependency Injection (의존성 주입) 을 수행한다. 따라서, **OCP를 만족** 한다.

# 정리

- **역할과 구현의 분리**
    - 역할: 인터페이스
    - 구현: 구체 클래스
    - 단적인 예로, 인터페이스 타입으로 구체화 객체를 담는 것을 말할 수 있다.
    - 예) `private final MemberRepository memberRepository = new MemoryMemberRepository();`

- **관심사 분리**
    - 한 클래스가 '객체생성', '객체연결', '객체실행' 을 모두 실행하게 된다면, OCP와 DIP 그리고 SRP를 위배하게 된다.
    - 따라서, 관심사 분리가 필요하다.

- **생성자 주입을 통한 OCP, DIP, SRP 만족**

<br>

---

<br>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*

  [인프런 강의로 가기](https://inf.run/pcN8)