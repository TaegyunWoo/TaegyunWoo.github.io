---
category: JPA
tags: [JPA]
title: "[JPA] 객체지향 쿼리 언어 - 소개"
date:   2021-11-28 16:30:00 
lastmod : 2021-11-28 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

앞으로 총 6개의 포스팅으로, 객체지향 쿼리 언어에 대해 다룰 예정이다.  

본 포스팅은 그에 대한 첫번째 글이다. 이 글을 통해, 앞으로 떠날 대장정에 대해 대략적인 흐름을 잡아보자.

<br/><br/>

# 객체지향 쿼리 소개

## 개요

### 다룰 내용

- 앞으로 다룰 내용
    - JPQL
    - QueryDSL
    - 네이티브 SQL
    - 객체지향 쿼리 심화

본 포스팅을 통해 객체지향 쿼리 언어에 대한 여행을 떠나기 전, 전체적인 흐름을 잡는데 도움을 줄 수 있도록 위에 서술한 내용에 대해 대략적으로 알아본다.

<br/>

### 본 포스팅에서 다룰 내용

JPQL, QueryDSL, 네이티브 SQL, 객체지향 쿼리 심화에 대해 간략하게 설명하겠다.

<br/>

### 들어가며...

- 이전 포스팅에선 지금까지 아래 방법을 통해, 엔티티·연관엔티티를 찾았다.
    - 식별자로 조회: `EntityManager.find()`
    - 객체 그래프 탐색: `a.getB().getC()`
- **하지만 이 기능만으로 애플리케이션을 개발하긴 어렵다.**
    - 예) 나이가 30살 이상인 회원을 모두 검색하기
    - 위 경우, 조건(`WHERE`) 검색이 필요하다.
    - 하지만 기존 방법으로는 불가능하다.
- **따라서 JPQL과 같은 객체지향 쿼리를 사용해야 한다.**

<br/>

### JPA가 제공하는 다양한 검색 방법들

- **JPQL**
- **Criteria 쿼리**
    - JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
    - 하지만 너무 복잡하여 QueryDSL을 사용하는 것을 권장한다.
    
    > 따라서 우리 블로그에서는 자세히 다루지 않는다.
    > 
- **네이티브 SQL**
    - JPA에서 JPQL 대신 직접 SQL을 사용할 수 있다.

<br/>

### JPA가 공식 지원하지는 않지만 유용한 검색 방법들

- **QueryDSL**
    - Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음
    - 비표준 오픈소스 프레임워크이다.
- **JDBC 직접사용, MyBatis와 같은 SQL 매퍼 프레임워크 사용**
    - 필요하면 JDBC를 직접 사용할 수 있다.

<br/><br/>

## JPQL 맛보기

### JPQL이란?

- ORM의 목적
    - DB 테이블이 아닌, 엔티티 객체를 대상으로 개발하기
- JPQL이란?
    - 위와 같은 ORM의 목적처럼, 엔티티 객체를 대상으로 데이터를 검색하기 위한 객체지향 쿼리이다.
    - 보다 복잡한 쿼리를 위해 JPA에서 제공하는 기능이다.

- JPQL의 특징
    - 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리
    - SQL을 추상화해서 특정 DB SQL에 의존하지 않는다.

- SQL vs JPQL
    - SQL
        - DB 테이블을 대상으로 하는 데이터 중심의 쿼리
    - JPQL
        - 엔티티 객체를 대상으로 하는 객체지향 쿼리
        - JPA가 JPQL을 분석하여 적절한 SQL을 만들어 DB를 조회한다.

- **다양한 검색 방법들 중, JPQL이 가장 중요하다.**
    - Criteria나 QueryDSL은 JPQL을 편하게 작성하도록 도와주는 빌더 클래스일 뿐이다.
    - 따라서 JPQL을 이해해야 나머지도 이해할 수 있다.

<br/>

### JPQL 소개

- JPQL은 엔티티 객체를 조회하는 객체지향 쿼리이다.
- JPQL은 SQL을 추상화해서 특정 DB에 의존하지 않는다.
    - 데이터베이스 방언(Dialect)만 변경하면 JPQL을 수정하지 않아도 자연스럽게 DB를 변경할 수 있다.
- JPQL은 SQL보다 간결하다.
    - 엔티티 직접 조회
    - 묵시적 조인
    - 다형성
    - 위 기능들을 지원하여, 간결하게 쿼리를 작성할 수 있다.

<br/>

### JPQL 사용 예시

- 회원 엔티티
    
    ```java
    @Entity(name = "Member") // 엔티티이름 설정
    public class Member {
    
    	@Column(name = "name")
    	private String username;
    
    	//...
    
    }
    ```
    
- JPQL 사용
    
    ```java
    //쿼리 생성
    String jpql = "select m from Member m where m.username = 'kim'";
    List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
    ```
    

<br/>

- 상세 설명
    - 회원이름이 `kim` 인 **엔티티를 조회**한다.
    - JPQL에서 `Member`는 **엔티티 이름**이다.
    - `m.username` 은 테이블 칼럼명이 아니라, **엔티티 객체의 필드명**이다.
    - `em.createQuery()` 메서드
        - 실행할 JPQL
        - 반환할 엔티티의 클래스 타입인 `Member.class`
        - 위 두가지를 해당 메서드에 넘겨주고 `getResultList()` 메서드를 실행하면, JPA는 JPQL을 SQL로 변환해서 DB를 조회한다.

<br/>

- 실행한 JPQL
    
    ```xml
    select m
    from Member m
    where m.username = 'kim'
    ```
    
- 실제로 실행된 SQL
    
    ```sql
    select
    	member.id as id,
    	member.age as age,
    	member.team_id as team,
    	member.name as name
    from
    	Member member
    where
    	member.name='kim'
    ```
    
    > 실제 하이버네이트 구현체가 생성한 SQL은 별칭이 복잡하다.  
    따라서 별칭부분만 알아보기 쉽게 수정한 것이다.
    > 

<br/><br/>

## Criteria 맛보기

### Criteria란?

> 여기서 다루는 내용을 마지막으로, Criteria에 대해선 앞으로 설명하지 않는다.
> 
- Criteria는 JPQL을 생성하는 빌더 클래스이다.
- Criteria의 장점
    - 문자가 아닌 프로그래밍 코드로 JPQL을 작성할 수 있다.
        - 예시) `query.select(m).where(...)`
        - JPQL에선 문자열을 직접 다룬다. 따라서 오타가 있어도 **컴파일은 성공하지만, 런타임 시점에 오류가 발생**한다.
        - 따라서 문자가 아닌 코드로 JPQL를 작성한다면, 컴파일 시점에 오류를 발견할 수 있다.
    - IDE를 사용하면 코드 자동완성을 지원한다.
    - 동적 쿼리를 작성하기 편하다.
- Criteria의 단점
    - **모든 장점을 상쇄할 정도로 복잡하고 장황하다.**
    - **따라서 사용하기 불편하고, 코드가 한눈에 들어오지도 않는다.**

<br/>

### Criteria 사용 예시

위 JPQL에서 작성한 것을 그대로 Criteria로 옮겨보자.

- JPQL 쿼리
    
    ```xml
    select m from Member as m where m.username = 'kim';
    ```
    
- Criteria 코드
    
    ```java
    //Criteria 사용 준비
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> query = cb.createQuery(Member.class);
    
    //루트 클래스 (조회를 시작할 클래스)
     Root<Member> m = query.from(Member.class);
    
    //쿼리 생성
    CriteriaQuery<Member> cq =
    		query.select(m).where(cb.equal(m.get("username"), "kim"));
    List<Member> resultList = em.createQuery(cq).getResultList();
    ```
    
    - `m.get("username")` 에서 필드명을 `"username"` 과 같은 문자열로 작성했다.
        - 메타모델을 사용하면, 문자가 아닌 코드로 작성할 수 있다.
    - 메타모델?
        - 자바가 제공하는 애너테이션 프로세서 기능을 사용하면, 애너테이션을 분석해서 클래스를 생성할 수 있다.
        - JPA는 이 기능을 사용해서 `Member` 엔티티 클래스로부터 `Member_` 라는 Criteria 전용 클래스를 생성한다.
        - 이것을 메타모델이라고 한다.

> 다시 말하지만, Criteria에 대해선 여기까지만 다룬다.  
Criteria를 좀 더 알아보고 싶다면, 다른 자료를 찾아보는 것을 추천한다.
> 

<br/><br/>

## QueryDSL 맛보기

### QueryDSL이란?

- Criteria처럼 JPQL 빌더 역할을 한다.
- QueryDSL의 장점
    - 코드 기반이다.
    - 단순하고 사용하기 쉽다.
    - 작성한 코드도 JPQL과 비슷해서 한눈에 들어온다.

> 따라서 Criteria 대신 QueryDSL을 사용하는 것을 권장한다.
> 

<br/>

### QueryDSL 사용 예시

이번에도 위 JPQL에서 작성한 것을 그대로 QueryDSL로 옮겨보자.

- JPQL 쿼리
    
    ```xml
    select m from Member as m where m.username = 'kim';
    ```
    
- QueryDSL 코드
    
    ```java
    //준비
    JPAQuery query = new JPAQuery(em);
    QMember member = QMember.member;
    
    //쿼리, 결과조회
    List<Member> members =
    		query.from(member)
    		.where(member.username.eq("kim"))
    		.list(member);
    ```
    

<br/>

- 상세설명
    - QueryDSL이 Criteria 보다 훨씬 간결하고, 보기 좋지 않은가?
    - QueryDSL 역시 애너테이션 프로세서를 사용해서, 쿼리 전용 클래스를 만들어야 한다.
        - Criteria 처럼 애너테이션 프로세서를 사용해야 한다.
        - `QMember` 가 `Member` 엔티티 클래스를 기반으로 생성한 QueryDSL 쿼리 전용 클래스이다.

<br/><br/>

## 네이티브 SQL 맛보기

### 네이티브 SQL이란?

- JPA에서 지원하는 기능으로, SQL을 직접 사용할 수 있다.
- 네이티브 SQL의 필요성
    - JPQL을 사용해도 가끔 특정 DB에 의존하는 기능을 사용해야 할 때가 있다.
        - 예시) 오라클의 `CONNECT BY` 기능 등
    - 이러한 기능들은 표준화가 되어있지 않으므로, JPQL에서 사용할 수 없다.
    - 이때 네이티브 SQL이 필요하다.
- 네이티브 SQL의 단점
    - 당연하게도, 특정 DB에 의존하는 SQL을 작성해야 한다는 것이 단점이다.
    - 따라서 DB 변경시, 네이티브 SQL도 수정해야 한다.

<br/>

### 네이티브 SQL 사용 예시

이번에도 위 JPQL에서 작성한 것을 그대로 네이티브 SQL로 옮겨보자.

- JPQL 쿼리
    
    ```xml
    select m from Member as m where m.username = 'kim';
    ```
    
- 네이티브 SQL 코드
    
    ```java
    String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER "
    				+ "WHERE NAME = 'kim'";
    List<Member> resultList = 
    		em.createNativeQuery(sql, Member.class).getResultList();
    ```
    
    - `em.createNativeQuery()` 를 사용하면 된다.
    - 나머지는 같다.

<br/><br/>

## JDBC 직접 사용, 마이바티스 같은 SQL 매퍼 프레임워크 사용

이럴 일은 드물겠지만, JDBC 커넥션에 직접 접근하고 싶으면 JPA는 JDBC 커넥션을 획득하는 API를 제공하지 않으므로, **JPA 구현체가 제공하는 방법을 사용**해야 한다.

### 하이버네이트에서 직접 JDBC Connection 획득하기

```java
// 1.
Session session = entityManager.unwrap(Session.class);

// 2.
session.doWork(new Work() {

	@Override
	public void execute(Connection connection) throws SQLException {
		//work...
	}

});

```

1. 먼저 JPA `EntityManager` 에서 하이버네이트 `Session` 을 구한다.
2. 그리고 `Session` 의 `doWork()` 메서드를 호출하면 된다.

<br/>

### 직접 JDBC or 마이바티스 사용 시, 발생하는 문제

- JDBC나 마이바티스를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시(Flush)해야 한다.
    
    > [플러시(Flush) 관련 포스팅](https://taegyunwoo.github.io/jpa/JPA_Persistence1#16)
    > 
- 왜냐하면 JDBC나 마이바티스는 모두 JPA를 위회해서 DB에 접근하기 때문이다.
    - **JPA를 우회하는 SQL에 대해서는 JPA가 전혀 인식하지 못한다.**
    - 따라서 영속성 컨텍스트와 DB를 불일치 상태로 만들어 데이터 무결성을 훼손할 수 있다.
- **따라서 JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시해서 DB와 영속성 컨텍스트를 동기화하면 된다.**

> 잘 이해가 되지 않아도 걱정할 필요없다. 추후 포스팅에서 자세히 다룬다.  
이런게 있구나 정도로만 알아두자.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>