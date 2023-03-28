---
category: JPA
tags: [JPA]
title: "[JPA] 영속성 관리 - 2"
date:   2021-10-09 21:06:00 
lastmod : 2021-10-09 21:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    - [[JPA] 영속성 관리 - 1](https://taegyunwoo.github.io/jpa/JPA_Persistence1)

본 게시글은 직전 게시글인 '[JPA] 영속성 관리 - 1' 게시글에 이어지는 내용을 설명합니다. 따라서, 해당 게시글을 아직 읽지 않으신 분은 해당 게시글을 먼저 읽어주시기 바랍니다.

<br/><br/><br/>

# 영속성 관리

## 플러시

### 플러시란?

- **플러시(Flush)는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.**

<br/>

### 플러시 vs 커밋

플러시에 대해 설명하기 전에, 커밋과의 차이점에 대해 간단히 알아보자.

- **Flush**
    - 영속성 컨텍스트의 변경 내용을 DB에 반영한다.
    - **아직 반영된 사항이 DB에서 확정된 것은 아니다!**
    - **즉, 어떤 것을 어떻게 변경할 것인지 DB에 알려주는 기능을 한다.**
    - 따라서, 트랜잭션 중 Flush를 하는 것은 자연스러운 것이며, Flush해도 트랜잭션이 마무리되지 않는다.
        - 그저 DB에 변경 내용을 알려주기만 하는 것이기 때문에, 트랜잭션은 아직 끝나지 않는다.

<br/>

- **Commit**
    - DB에 반영된 변경사항이 확정된다.
    - **즉, Flush를 통해 반영된 변경사항을 DB가 실제로 수행하도록 한다.**
    - 트랜잭션을 Commit 하는 것은 트랜잭션을 DB에 적용하고, 트랜잭션을 끝낸다는 의미이다.

<br/>

### 플러시 작동

1. **변경 감지(Dirty Checking)이 동작**해서, 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교하요 수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어 '쓰기 지연 SQL 저장소'에 등록한다.
2. '쓰기 지연 SQL 저장소'의 쿼리를 DB에 전송한다. *(등록, 수정, 삭제 쿼리)*

<br/>

### 영속성 컨텍스트를 플러시하는 방법

영속성 컨텍스트를 플러시하는 방법으로 아래 3가지가 존재한다.

- `em.flush()` 를 직접 호출한다.
- 트랜잭션 커밋 시 플러시가 자동 호출된다.
- JPQL 쿼리 실행 시 플러시가 자동 호출된다.

지금부터 하나씩 자세히 알아보자

<br/>

### 플러시 직접 호출

- 엔티티 매니저의 `flush()` 메서드를 직접 호출해서 영속성 컨텍스트를 강제로 플러시한다.
- 하지만, 아래 경우를 제외하고 거의 사용되지 않는다.
    - 테스트
    - 타 프레임워크 + JPA 사용시

<br/>

### 트랜잭션 커밋 시 플러시 자동 호출

- SQL 쿼리를 DB에 전달하지 않고, 커밋(commit)을 하면 어떠한 동작도 이루어지지 않는다.
    - 변경 사항을 DB에 알려주지도 않고 변경 확정만 하는 꼴이다.
- **따라서, 트랜잭션을 커밋하기 전에, 꼭 플러시를 호출해서 영속성 컨텍스트의 변경 내용을 DB에 반영해야 한다.**
- **JPA는 이런 문제를 예방하기 위해, 트랜잭션을 커밋할 때 플러시를 자동으로 호출한다.**

<br/>

### JPQL 쿼리 실행 시 플러시 자동 호출

- JPQL이나 Criteria(추후 포스팅 예정) 같은 객체지향 쿼리를 호출할 때도 플러시가 실행된다.
- 왜 그럴까? 아래 예시코드를 보자.
    
    ```java
    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);
    
    //중간에 JPQL 실행
    query = em.createQuery("select m from Member m", Member.class);
    List<Member> members = query.getResultList();
    ```
    
    - 엔티티 `memberA` , `memberB` , `memberC` 를 영속 상태로 만들었다.
    - 해당 엔티티들은 영속성 컨텍스트에는 존재하지만, DB에는 아직 반영되지 않았다.
    - **이 상태에서 JPQL을 실행하면, JPQL이 SQL로 변환되어 DB에서 엔티티를 조회한다.**
    - 그런데 `memberA`, `memberB` , `memberC` 는 아직 DB에 반영되지 않아 존재하지 않으므로, 쿼리 결과로 조회되지 않는다!
    - **따라서 쿼리를 실행하기 직전에 영속성 컨텍스트를 플러시해서 변경 내용을 DB에 반영해야 한다.**
    - JPA는 이런 문제를 예방하기 위해, JPQL을 실행할 때도 플러시를 자동 호출한다.

<br/>

### 플러시 모드 옵션

- 엔티티 매니저에 플러시 모드를 직접 지정하려면 `javax.persistence.FlushModeType` 을 사용하면 된다.
    - `FlushModeType.AUTO`
        - 커밋이나 쿼리를 실행할 때 플러시한다.
        - 기본값이다.
    - `FlushModeType.COMMIT`
        - 커밋할 때만 플러시한다.
- 플러시 모드 변경 예시
    
    ```java
    EntityManager em = emf.createEntityManager();
    em.setFlushMode(FlushModeType.COMMIT); //플러시 모드 직접 설정
    ```
    
- **하지만 대부분 AUTO 기본 설정을 그대로 사용한다.**

<br/><br/>

## 준영속

### 알아볼 내용

- 과거에 엔티티의 "비영속 → 영속 → 삭제 상태 변화"에 대해 자세히 알아보았었다.
- 이제, "영속 → 준영속 상태 변화"에 대해 알아보자.

<br/>

### 준영속이란?

- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 것을 준영속 상태라고 한다.
- 따라서, 준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

<br/>

### 준영속 상태로 만드는 방법

영속 상태의 엔티티를 준영속 상태로 만드는 방법에는 아래처럼 3가지의 방법이 존재한다.

- **`em.detach(entity)`**
    - 특정 엔티티만 준영속 상태로 전환한다.
- **`em.clear()`**
    - 영속성 컨텍스트를 완전히 초기화한다.
- **`em.close()`**
    - 영속성 컨텍스트를 종료한다.

지금부터 하나씩 자세히 알아보자.

<br/>

### 준영속 상태로 전환하기: `detach()`

- `em.detach()` 메서드는 특정 엔티티를 준영속 상태로 만든다.
- 메서드 형식
    - `detach(영속상태_엔티티)`
- 예시 코드
    
    ```java
    public void testDetached() {
    	...
    	//회원 엔티티 생성
    	//비영속 상태
    	Member member = new Member();
    	member.setId("memberA");
    	member.setUsername("회원A");
    
    	//회원 엔티티 영속 상태
    	em.persist(member);
    
    	//회원 엔티티를 영속성 컨텍스트에서 분리한다.
    	//준영속 상태
    	em.detach(member);
    
    	transaction.commit(); //[트랜잭션-커밋]
    }
    ```

<br/>

- 회원 엔티티(`member`)를 영속화한 다음 `em.detach(member)`를 호출한다.
    - 이것은 **영속성 컨텍스트에게 더는 해당 엔티티를 관리하지 말라는 것**이다.
- 해당 메서드를 호출하는 순간 1차 캐시부터 '쓰기 지연 SQL 저장소'까지 해당 엔티티를 관리하기 위한 모든 정보가 제거된다.

<br/>

- 영속성 컨텍스트 변화
    - 아래 그림이 위 예시코드에서 영속성 컨텍스트가 어떻게 변화하는지 나타낸 것이다.
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2014.png)
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2015.png)
    
<br/>

- **정리**
    - 영속상태: 영속성 컨텍스트로부터 관리되는 상태이다.
    - 준영속상태: 영속성 컨텍스트로부터 분리된 상태이다.

<br/>

### 영속성 컨텍스트 초기화: `clear()`

- `em.clear()` 은 영속성 컨텍스트를 초기화해서, 해당 영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만든다.

<br/>

- 예시 코드
    
    ```java
    //엔티티 조회
    //영속 상태
    Member member = em.find(Member.class, "memberA");
    
    em.clear(); //영속성 컨텍스트 초기화
    
    //준영속 상태에서 필드 변경 시도
    member.setUsername("changeName");
    ```
    
    - 준영속 상태인 `member` 엔티티에 대해 필드 변경을 시도하였으므로, Dirty Checking이 작동하지도 않고, DB에 수정내용이 반영되지도 않는다.

<br/>

- 영속성 컨텍스트 변화
    - 아래 그림이 위 예시코드에서 영속성 컨텍스트가 어떻게 변화하는지 나타낸 것이다.
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2016.png)
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2017.png)
    

- 영속성 컨텍스트에 있는 모든 것이 초기화되어 버렸다. ⇒ 이것은 영속성 컨텍스트를 제거하고 새로 만든 것과 같다.
- **따라서, `memberA`, `memberB` 는 영속성 컨텍스트가 관리하지 않으므로, 준영속 상태이다.**

<br/>

### 영속성 컨텍스트 종료: `close()`

- 영속성 컨텍스트를 종료하면, 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다.

<br/>

- 예시 코드
    
    ```java
    public void closeEntityManager() {
    
    	EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
    	EntityManager em = emf.createEntityManager();
    	EntityTransaction transaction = em.getTransaction();
    
    	transaction.begin(); //[트랜잭션 - 시작]
    	
    	Member memberA = em.find(Member.class, "memberA");
    	Member memberB = em.find(Member.class, "memberB");
    
    	transaction.commit(); //[트랜잭션 - 커밋]
    	em.close(); //영속성 컨텍스트 닫기 (종료)
    }
    ```
    
<br/>

- 영속성 컨텍스트 변화
    - 아래 그림이 위 예시코드에서 영속성 컨텍스트가 어떻게 변화하는지 나타낸 것이다.
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2018.png)
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2019.png)
    
    - 영속성 컨텍스트가 종료되어 더는 `memberA`, `memberB`가 관리되지 않는다.

> 영속 상태의 엔티티는 주로 영속성 컨텍스트가 종료되면서 준영속 상태가 된다.  
개발자가 직접 준영속 상태로 만드는 일은 드물다.
> 

<br/>

### 준영속 상태의 특징

- **거의 비영속 상태에 가깝다**
    - 영속성 컨텍스트가 제공하는 어떠한 기능도 동작하지 않는다.

- **식별자 값을 가지고 있다**
    - 비영속 상태의 엔티티는 식별자 값이 없을 수도 있다.
        - 예를 들어, `new 엔티티클래스()` 코드로 엔티티 생성시 `@Id` 가 적용된 필드의 값이 없는 상태이다.
    - **하지만, 준영속 상태는 이미 한번 영속 상태였으므로 반드시 식별자 값을 가지고 있다.**

- **지연 로딩을 할 수 없다**
    - 지연 로딩은 실제 객체 대신 프록시 객체를 로딩해두고 해당 객체를 실제 사용할 때, 영속성 컨텍스트를 통해 데이터를 불러오는 방법이다.
    - 하지만, 준영속 상태는 영속성 컨텍스트가 관리하지 않으므로 지연 로딩 시 문제가 발생한다.
    - 지연 로딩에 대한 자세한 내용은 추후에 포스팅할 예정이다.

<br/><br/>

## 병합

### 병합: `merge()`

- 준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다.
- `merge()` 메서드는 준영속 상태의 엔티티를 받아서 그 정보로 **새로운 영속 상태의 엔티티를 반환**한다.
- 메서드 형식
    - `merge(준영속_엔티티)`

<br/>

- 예시 코드
    
    ```java
    public class ExamMergeMain {
    
    	EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
    
    	public static void main(String[] args) {
    		Member member = createMember("memberA", "회원1");
    		
    		member.setUsername("회원명 변경"); //준영속 상태에서 변경 시도
    
    		mergeMember(member);
    	}
    
    	static Member createMember(String id, String username) {
    		EntityManager em1 = emf.createEntityManager();
    		EntityTransaction tx1 = em1.getTransaction();
    	
    		tx1.begin(); //[트랜잭션 - 시작]
    		
    		Member member = new Member();
    		member.setId(id);
    		member.setUsername(username);
    
    		em1.persist(member);
    		tx1.commit(); //[트랜잭션 - 커밋]
    
    		em1.close(); //영속성 컨텍스트 종료
    		//member는 준영속 상태의 엔티티가 된다. (영속성 컨텍스트가 종료되었으므로)
    		
    		return member;
    	}
    
    	static Member mergeMember(Member member) {
    		EntityManager em2 = emf.createEntityManager();
    		EntityTransaction tx2 = em2.getTransaction();
    	
    		tx2.begin(); //[트랜잭션 - 시작]
    		
    		//전달받은 준영속 상태의 엔티티의 내용을 복사한 새 엔티티를 생성
    		//새 엔티티를 새 영속성 컨텍스트(em2의)에 등록
    		Member mergeMember = em2.merge(member);
    		
    		tx2.commit(); //[트랜잭션 - 커밋]
    
    		em2.close(); //영속성 컨텍스트 종료
    		//member는 준영속 상태의 엔티티가 된다. (영속성 컨텍스트가 종료되었으므로)
    	}
    
    }
    ```
    
    1. `member` 엔티티는 영속 상태였다가 영속성 컨텍스트1이 종료되면서 준영속 상태가 되었다. 따라서 `createMember()` 메서드는 준영속 상태의 `member` 엔티티를 반환한다.
    2. 준영속 상태인 `member` 엔티티를 관리하는 영속성 컨텍스트가 더는 존재하지 않으므로 수정 사항을 DB에 반영할 수 없다.
    3. 준영속 상태의 엔티티를 수정하려면 준영속 상태를 다시 영속 상태로 변경해야 한다. 이때 병합(`merge()`)를 사용한다.
        - **그러면 `member` 엔티티의 정보를 복사한 새로운 영속 상태의 엔티티가 반환된다.**
        - **즉, `member` 엔티티가 준영속 상태에서 영속 상태로 변경되는 것은 아니다.**

<br/>

- `merge()` 의 동작 방식
    
    ![Untitled](/assets/img/2021-10-09-JPA_Persistence2/Untitled%2020.png)
    
    1. `merge()` 를 실행한다.
    2. 파라미터로 넘어온 **준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회**한다.
        
        2-2. 만약 1차 캐시에 엔티티가 없으면 DB에서 엔티티를 조회하고 1차 캐시에 저장한다.
        
    3. **조회한 영속 엔티티에 `member` 엔티티의 값을 채워 넣는다.**
        - 이때, 회원명이 변경된다!
    4. `mergeMember` 를 반환한다.

- 병합이 끝나고 `tx2.commit()` 을 호출해서 트랜잭션을 커밋했다. 이때 변경 감지 기능이 동작해서 변경 내용을 DB에 반영한다.
- `merge()` 는 파라미터로 넘어온 준영속 엔티티를 사용해서, 새롭게 병합된 영속 상태의 엔티티를 반환한다.
    - **파라미터로 넘어온 엔티티(기존의 준영속 상태 엔티티)는 병합 후에도 준영속 상태로 남아있다!!**

<br/>

### 비영속 병합

- 병합은 비영속 엔티티도 영속 상태로 만들 수 있다.

- 예시 코드
    
    ```java
    Member member = new Member();
    Member newMember = em.merge(member); //비영속 병합
    tx.commit();
    ```

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>