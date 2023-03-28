---
category: JPA
tags: [JPA]
title: "[JPA] 연관관계 매핑 - 기초"
date:   2021-10-18 20:06:00 
lastmod : 2021-10-18 20:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 연관관계 매핑 기초

## 개요

### 객체와 테이블의 연관관계 방식

- 객체의 연관관계 방식
    - 참조(주소)를 사용해서 관계를 맺는다.
- 테이블의 연관관계 방식
    - 외래키를 사용해서 관계를 맺는다.

<br/>

### 목표

- **객체의 참조와 테이블의 외래 키를 매핑하는 것**

<br/>

### 핵심 키워드

- **방향**
    - 단방향, 양방향
    - 단방향 예시
        - 회원과 팀이 존재하고, 서로 관계가 있을 때
        - "회원 → 팀" 또는
        - "팀 → 회원" 둘 중 한 쪽만 참조하는 것
    - 양방향 예시
        - "회원 → 팀"과 "팀 → 회원" 둘 다 참조하는 것

<br/>

- **다중성**
    - 다대일, 일대다, 일대일, 다대다
    - 다대일 관계 예시
        - 여러 회원이 한 팀에 속한다.
        - 그러므로, 회원과 팀은 "다대일" 관계이다.
    - 일대다 관계 예시
        - 한 팀에 여러 회원이 속한다.
        - 그러므로, 팀과 회원은 "일대다" 관계이다.

<br/>

- **연관관계의 주인**
    - 객체를 양방향 연관관계로 만들면 연관관계의 주인을 정해야 한다.

<br/><br/>

## 단방향 연관관계

### 단방향 연관관계란?

단방향 연관관계를 이해하기 위해, 먼저 예시 배경을 명시하고 진행하겠다. 아래는 예시 배경에 대한 서술 내용이다.

- 예시 배경
    - 회원과 팀이 있다.
    - 회원은 하나의 팀에만 소속될 수 있다.
    - 회원과 팀은 다대일 관계이다.

<br/>

- 다대일 연관관계, 단방향
    
    ![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled.png)
    
    - 객체 연관관계
        - 회원 객체는 Member.team 필드로 팀 객체와 연관관계를 맺는다.
        - 회원 객체와 팀 객체는 **단방향 관계**이다.
            - 회원은 Member.team 필드를 통해서 팀을 알 수 있지만, 팀은 회원을 알 수 없다.
    - 테이블 연관관계
        - 회원 테이블은 TEAM_ID 외래키로 팀 테이블과 연관관계를 맺는다.
        - 회원 테이블과 팀 테이블은 **양방향 관계**이다.
        - 회원 테이블의 TEAM_ID 외래키를 통해서 회원과 팀을 조인할 수 있고, 팀과 회원도 조인할 수 있다.
            - `MEMBER JOIN TEAM` , `TEAM JOIN MEMBER` 모두 가능하다.

<br/>

- 객체 연관관계와 테이블 연관관계의 차이점
    - 참조를 통한 연관관계는 언제나 단방향이다.
        - 만약 객체간에 연관관계를 양방향으로 만들고 싶다면, 반대쪽에도 필드를 추가해서 참조를 보관해야 한다.
        - **하지만 정확히 말하자면, 이것은 양방향 관계가 아니라 서로 다른 단방향 관계가 2개 있는 것이다.**
    - 반면에 테이블은 외래키 하나로 양방향으로 조인할 수 있다.

<br/>

- 정리
    - 객체는 참조(주소)로 연관관계를 맺는다.
    - 테이블은 외래키로 연관관계를 맺는다.
    - 객체는 **참조(`a.getB().getC()`)** 를 통해 연관된 데이터를 조회한다.
    - 테이블은 **조인(`JOIN`)** 을 통해 연관된 데이터를 조회한다.
    - 참조를 사용하는 객체의 연관관계는 **단방향**이다.
    - 외래키를 사용하는 테이블의 연관관계는 **양방향**이다.
    - 객체를 양방향으로 참조하려면, 단방향 연관관계를 2개 만들어야 한다.

<br/>

### 순수한 객체 연관관계

순수하게 객체만 사용한 연관관계를 살펴보자.

- 회원 클래스
    
    ```java
    public class Member {
    	private String id;
    	private String username;
    
    	private Team team; // 팀의 참조를 보관한다.
    
    	public Member() {}
    	public Member(id, username) {
    		this.id = id;
    		this.username = username;
    	}
    
    	public void setTeam(Team team) {
    		this.team = team;
    	}
    
    	//getter, setter 생략
    }
    ```
    

- 팀 클래스
    
    ```java
    public class Team {
    	private String id;
    	private String name;
    
    	public Team() {}
    	public Team(id, name) {
    		this.id = id;
    		this.name = name;
    	}
    	//getter, setter 생략
    }
    ```
    

- 동작코드
    - 아래 코드를 통해서, 회원1과 회원2를 팀1에 소속시키자.
    
    ```java
    public static void main(String[] args) {
    	Member member1 = new Member("member1", "회원1");
    	Member member2 = new Member("member2", "회원2");
    	Team team1 = new Team("team1", "팀1");
    
    	//팀에 소속시키기
    	member1.setTeam(team1);
    	member2.setTeam(team1);
    
    	//객체 그래프 탐색
    	Team findTeam = member1.getTeam();
    }
    ```
    
    - 다대일 인스턴스 관계
        
        ![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%201.png)
        
    
    - 객체 그래프 탐색
        - `Team findTeam = member1.getTeam();`
        - 위 코드처럼, 객체는 참조를 사용해서 연관관계를 탐색할 수 있다.
        - 이것을 **객체 그래프 탐색** 이라고 한다.
    
<br/>

### 테이블 연관관계

이번에는 DB 테이블의 회원과 팀의 관계를 살펴보자. 아래는 회원 테이블과 팀 테이블의 DDL이다.

- 테이블 DDL
    
    ```sql
    //MEMBER 테이블 추가
    CREATE TABLE MEMBER (
    	MEMBER_ID VARCHAR(255) NOT NULL,
    	TEAM_ID VARCHAR(255),
    	USERNAME VARCHAR(255),
    	PRIMARY KEY (MEMBER_ID)
    )
    
    //TEAM 테이블 추가
    CREATE TABLE TEAM (
    	TEAM_ID VARCHAR(255) NOT NULL,
    	NAME VARCHAR(255),
    	PRIMARY KEY (TEAM_ID)
    )
    
    //MEMBER 테이블에 외래키 추가
    ALTER TABLE MEMBER ADD CONSTRAINT FK_MEMBER_TEAM
    	FOREIGN KEY (TEAM_ID)
    	REFERENCES TEAM
    ```
    

- 동작 SQL
    - 아래 SQL을 통해서, 회원1과 회원2를 팀1에 소속시키자.
    
    ```sql
    INSERT INTO TEAM(TEAM_ID, NAME) VALUES ('team1', '팀1');
    INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, NAME) VALUES ('member1', 'team1', '회원1');
    INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, NAME) VALUES ('member2', 'team1', '회원2');
    
    //회원1의 소속팀 조회
    SELECT * FROM MEMBER M
    	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
    	WHERE M.MEMBER_ID = 'member1';
    ```
    
<br/>

- 조인
    - DB는 위 코드처럼 외래 키를 사용해서 연관관계를 탐색한다.
    - 이것을 **조인**이라고 한다.

<br/>

### 객체 관계 매핑

이제 JPA를 사용해서 "객체만 사용한 연관관계"와 "테이블만 사용한 연관관계"를 매핑해보자.

- 매핑한 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id
    	@Column(name = "MEMBER_ID")
    	private String id;
    	private String username;
    
    	//연관관계 매핑
    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;
    
    	public Member() {}
    	public Member(id, username) {
    		this.id = id;
    		this.username = username;
    	}
    
    	//연관관계 설정
    	public void setTeam(Team team) {
    		this.team = team;
    	}
    
    	//getter, setter 생략
    }
    ```
    

- 매핑한 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id
    	@Column(name = "TEAM_ID")
    	private String id;
    	private String name;
    
    	public Team() {}
    	public Team(id, name) {
    		this.id = id;
    		this.name = name;
    	}
    	//getter, setter 생략
    }
    ```
    

- 연관관계
    - **객체 연관관계**
        - 회원 객체의 **Member.team 필드**를 사용하여 연관관계를 맺음.
    - **테이블 연관관계**
        - 회원 테이블의 **MEMBER.TEAM_ID 외래키 컬럼**을 사용하여 연관관계를 맺음.

- 매핑 코드 분석
    
    ```java
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ```
    
    - `@ManyToOne`
        - 다대일 관계라는 매핑 정보이다.
        - Member와 Team의 관계는 다대일이다.
    - `@JoinColumn(name = "TEAM_ID")`
        - 외래 키를 매핑할 때 사용한다.
        - `name` 속성에는 매핑할 **외래 키 이름을 지정**한다.
        - 회원과 팀 테이블은 TEAM_ID 외래키로 연관관계를 맺으므로, `"TEAM_ID"` 로 지정되었다.

<br/><br/>

## 애너테이션 설명

### `@JoinColumn`

- 목적
    - `@JoinColumn` 애너테이션은 외래키를 매핑할 때 사용된다.

<br/>

- 주요 속성

|속성|기능|기본값|
|----|----|------|
|name|매핑할 외래키 이름|'필드명' + '_' + '참조하는 테이블의 기본키 컬럼명'|
|referencedColumnName|외래키가 참조하는 대상 테이블의 컬럼명|참조하는 테이블의 기본키 컬럼명|
|foreignKey(DDL)|- 외래키 제약조건을 직접 지정할 수 있다.<br/>- 이 속성은 테이블을 생성할 때만 사용한다.||
|unique|`@Column` 의 속성과 같다.||
|nullable|`@Column` 의 속성과 같다.||
|insertable|`@Column` 의 속성과 같다.||
|updatable|`@Column` 의 속성과 같다.||
|columnDefinition|`@Column` 의 속성과 같다.||
|table|`@Column` 의 속성과 같다.||

<br/>

- `@JoinColumn` 생략시
    - 생략 예시
        
        ```java
        @ManyToOne
        private Team team;
        ```
        
    - 해당 애너테이션 생략시, 외래 키를 찾을 때 기본 전략을 사용한다.
        - '필드명' + '_' + '참조하는 테이블의 기본키 컬럼명'

<br/>

### `@ManyToOne`

- 목적
    - `@ManyToOne` 애너테이션은 다대일 관계에서 사용한다.

<br/>

- 주요 속성

|속성|기능|기본값|
|----|----|------|
|optional|false로 설정하면, 연관된 엔티티가 항상 있어야 한다.|true|
|fetch|자세한 내용은 추후에 다룬다.|- @ManyToOne=FetchType.EAGER<br/>- @OneToMany=FetchType.LAZY|
|cascade|자세한 내용은 추후에 다룬다.||
|targetEntity|- 연관된 엔티티의 타입 정보를 설정한다.<br/>- 이 기능은 거의 사용하지 않는다.<br/>- 왜냐하면, 제네릭을 통해 타입을 알 수 있기 때문이다.||

<br/>

- `targetEntity` 속성 사용 예시
    
    ```java
    //--- targetEntity가 필요없는 코드 ---
    @OneToMany
    private List<Member> members; //제네릭 타입 O
    
    //--- targetEntity가 필요한 코드 ---
    @OneToMany(targetEntity = Member.class)
    private List members; //제네릭 타입 X
    ```
    
<br/><br/>

## 연관관계 사용

연관관계를 등록, 수정, 삭제, 조회하는 예제를 통해 연관관계를 어떻게 사용하는지, 예시코드를 통해 알아보자.

<br/>

### 저장

- 아래 코드는 회원과 팀을 저장하는 코드이다.

```java
public void testSave() {
	//팀1 저장
	Team team1 = new Team("team1", "팀1");
	em.persist(team1);

	//회원1 저장
	Member member1 = new Member("member1", "회원1");
	member1.setTeam(team1); // 연관관계 설정 (member1 -> team1)
	em.persist(member1);

	//회원2 저장
	Member member2 = new Member("member2", "회원2");
	member2.setTeam(team1); // 연관관계 설정 (member1 -> team1)
	em.persist(member2);
}
```

- **JPA에서 엔티티를 저장할 때, 연관된 모든 엔티티는 영속 상태여야 한다.**
    - 따라서 위 예시에서 `member1`과 `member2`가 `team1`을 가르킬 때, `team1`이 영속성 컨텍스트에 포함된 상태이다.

- 주요 소스 코드 분석
    
    ```java
    member1.setTeam(team1); // 연관관계 설정 (member1 -> team1)
    em.persist(member1);
    ```
    
    - 회원 엔티티는 팀 엔티티를 찾조하고 저장했다.
    - JPA는 **참조한 팀의 기본 식별자(Team.id)를 외래키로 사용해서 적절한 등록 쿼리를 생성**한다.
    - 결과 SQL
        
        ```sql
        INSERT INTO TEAM (TEAM_ID, NAME) VALUES ('team1', '팀1') //팀1 저장
        INSERT INTO MEMBER (MEMBER_ID, NAME, TEAM_ID) VALUES ('member1', '회원1', 'team1') //회원1 저장
        INSERT INTO MEMBER (MEMBER_ID, NAME, TEAM_ID) VALUES ('member2', '회원2', 'team1') //회원2 저장
        ```
        
<br/>

### 조회

이제 연관관계가 있는 엔티티를 조회하는 방법에 대해 알아보자. 연관관계가 있는 엔티티를 조회하는 방법은 크게 2가지가 존재한다. 그것은 아래와 같다.

- 연관관계가 있는 엔티티를 조회하는 방법
    - 객체 그래프 탐색 (객체 연관관계를 사용한 조회)
    - 객체지향 쿼리 사용 (JPQL)

지금부터 하나씩 설명하도록 하겠다.

<br/>

- 객체 그래프 탐색
    - 먼저 위에서 저장한대로, 회원1과 회원2가 팀1에 소속해 있다고 가정하자.
    - `member.getTeam()` 을 사용해서 `member`와 연관된 `team` 엔티티를 조회할 수 있다.
    - 예시 코드
        
        ```java
        Member member = em.find(Member.class, "member1");
        Team team = member.getTeam(); //객체 그래프 탐색
        ```
        
    - 이처럼 객체를 통해, 연관된 엔티티를 조회하는 것을 객체 그래프 탐색이라고 한다.
    - 객체 그래프 탐색에 대한 자세한 내용은 추후에 설명하겠다.

<br/>

- 객체지향 쿼리 사용
    - JPQL에서도 조인을 지원하므로, 조인을 사용하여 연관관계를 사용할 수 있다.
    - 예시 코드
        
        ```java
        private static void queryLogicJoin(EntityManager em) {
        	//jpql
        	String jpql = "select m from Member m join m.team t where" +
        				"t.name=:teamName";
        
        	List<Member> resultList = em.createQuery(jpql, Member.class)
        					.setParameter("teamName", "팀1")
        					.getResultList();
        }
        ```
        
    - JPQL의 `from Member m join m.team t` 부분을 보면 회원이 팀과 관계를 가지고 있는 필드(m.team)를 통해서, Member와 Team을 조인했다.
    - JPQL을 포함한 객체 쿼리에 대한 상세한 내용은 추후에 다룬다.

<br/>

### 수정

이번에는 연관관계를 어떻게 수정하는지 알아보자. **팀1 소속이던 회원을 새로운 팀2에 소속하도록 수정해보자.**

- 예시 코드
    
    ```java
    private static void updateRelation(EntityManager em) {
    	
    	//새로운 팀2
    	Team team2 = new Team("team2", "팀2");
    
    	//회원1에 새로운 팀2를 설정한다.
    	Member member = em.find(Member.class, "member1");
    	member.setTeam(team2);
    }
    ```
    
    - 이전 게시글에서 설명했듯이, JPA는 member엔티티의 **모든 필드**를 수정한다.

<br/>

- 단순히 불러온 엔티티의 값만 변경해두면 트랜잭션을 커밋할 때 플러시가 일어나면서, 변경감지 기능 (Dirty Checking)이 작동한다. 이러한 기능이 연관관계를 수정할 때도 일어난다.

<br/>

### 제거

마지막으로 연관관계를 제거하는 방법에 대해 설명하겠다. **회원1을 팀에 소속하지 않도록 변경해보자.**

- 예시 코드
    
    ```java
    private static void deleteRelation(EntityManager em) {
    	
    	Member member = em.find(Member.class, "member1");
    	member.setTeam(null); //연관관계 제거
    }
    ```
    

- 위 코드처럼 null을 사용하여 연관관계를 제거할 수 있다.

<br/>

### 연관된 엔티티 삭제

이제 연관된 엔티티를 삭제하는 방법에 대해 알아보자. (ex. team1 엔티티 삭제) 연관된 엔티티를 삭제하는 절차는 아래와 같다.

- 연관된 엔티티 삭제 절차
    1. 연관관계 제거
    2. 엔티티 삭제

- 예시 코드
    
    ```java
    member1.setTeam(null); //회원1 연관관계 제거
    member2.setTeam(null); //회원2 연관관계 제거
    em.remove(team); //팀 삭제
    ```
    
<br/><br/>

## 양방향 연관관계

- 이번에는 반대 방향인 팀에서 회원으로 접근하는 관계를 추가하자.
- "회원→팀", "팀→회원"이 가능하도록 매핑해보자.

<br/>

### 양방향 객체 연관관계

- 먼저 객체 연관관계를 살펴보자.
- 아래 그림처럼, 회원과 팀은 다대일 관계이다.
- 반대로, 팀과 회원은 일대다 관계이다.
- 일대다 관계는 **여러 건과 연관관계를 맺을 수 있으므로, 컬렉션을 사용**해야 한다.

![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%202.png)

<br/>

### 양방향 테이블 연관관계

- DB 테이블은 외래 키 하나로 양방향 조회가 가능하다.
- 즉, 처음부터 양방향 관계이다.
- 따라서, DB에 추가해야하는 내용은 전혀 없다.

![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%203.png)

<br/>

### 양방향 연관관계 매핑

이제 양방향 관계를 매핑해보자.

- 매핑한 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id
    	@Column(name = "MEMBER_ID")
    	private String id;
    	private String username;
    
    	//연관관계 매핑
    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;
    
    	public Member() {}
    	public Member(id, username) {
    		this.id = id;
    		this.username = username;
    	}
    
    	//연관관계 설정
    	public void setTeam(Team team) {
    		this.team = team;
    	}
    
    	//getter, setter 생략
    }
    ```
    
    - 회원 엔티티 부분은 변경할 사항이 없다.

- 매핑한 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id
    	@Column(name = "TEAM_ID")
    	private String id;
    	private String name;
    
    	//일대다 관계 설정
    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();
    
    	public Team() {}
    	public Team(id, name) {
    		this.id = id;
    		this.name = name;
    	}
    	//getter, setter 생략
    }
    ```
    

- 코드 설명
    - 팀과 회원은 일대다 연관관계이다.
    - 따라서, 팀 엔티티에 컬렉션(`List<Member>`)을 추가했다.
    - 그리고 일대다 관계 매핑을 위해 `@OneToMany` 매핑 정보를 사용했다.
    - `mappedBy` 속성
        - 이 속성은 양방향 매핑일 때 사용한다.
        - **반대쪽 매핑의 필드 이름**을 값으로 주면 된다.
        - 따라서, 반대쪽(Member 엔티티)의 필드 team의 이름을 값으로 주었다.
        
        > 자세한 내용은 아래에서 계속 설명한다.
        > 
    - 이것으로 양방향 매핑을 완료했다. 이제부터 팀 엔티티에서 회원 컬렉션으로 객체 그래프를 탐색할 수 있다.

<br/>

### 일대다 컬렉션 조회

위에서 설정한 일대다 컬렉션을 조회하는 방법에 대해 알아보자.

- 예시 코드
    
    ```java
    public void biDirection() {
    	Team team = em.find(Team.class, "team1");
    	
    	//컬렉션 조회
    	List<Member> members = team.getMembers(); //객체 그래프 조회 (팀->회원)
    }
    ```
    
<br/><br/>

## 연관관계의 주인

### 연관관계의 주인이 필요한 배경

위에서 설명한 `@OneToMany` 는 직관적으로 이해할 수 있다. 그렇다면 `mappedBy` 라는 속성은 왜 필요할까?

- 객체에는 양방향 연관관계라는 것이 없다.
- 따라서, **서로 다른 단방향 연관관계 2개를 Application 로직으로 잘 묶어서 양방향인 것처럼 보이게 해야한다.**

- 객체 연관관계 vs 테이블 연관관계
    - **객체 연관관계**
        - 회원 → 팀 연관관계 1개 (단방향)
        - 팀 → 회원 연관관계 1개 (단방향)
    - **테이블 연관관계**
        - 회원 ↔ 팀 연관관계 1개 (양방향)

- 연관관계의 주인 필요성
    - 이러한 객체와 테이블간의 차이 때문에, **JPA는 연관관계의 주인으로 연관관계를 관리**한다.

<br/>

### 연관관계의 주인이란?

- 연관관계의 주인이란, **두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리하는 것**이다.
- 엔티티 자체가 아닌, **엔티티의 연관관계 필드(외래키 필드)가 연관관계의 주인으로 선택**된다.
- **즉, 연관관계의 주인을 정한다는 것은 외래키 관리자를 선택하는 것이다.**

<br/>

### 양방향 매핑의 규칙: 연관관계의 주인

- **연관관계의 주인만이 'DB 연관관계'와 매핑되고, '외래키를 관리(등록, 수정, 삭제)'할 수 있다.**
- 반면에 주인이 아닌 쪽은 읽기만 할 수 있다.

<br/>

### 연관관계의 주인 설정

- 위에서 설명한 `@OneToMany` 애너테이션의 속성 `mappedBy` 를 통해 연관관계의 주인을 설정할 수 있다.
- **연관관계의 주인은 `mappedBy` 속성을 사용하지 않는다.**
- **주인이 아니면, `mappedBy` 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야한다.**
- **연관관계의 주인을 정한다는 것은 외래 키 관리자를 선택하는 것이다.**
    - 즉, 회원·팀 예시에서는 `TEAM_ID` 외래키를 관리할 관리자를 선택해야한다.

<br/>

- 그렇다면 어떤 것을 연관관계의 주인으로 선택해야할까?
    - 먼저 객체 연관관계와 테이블 연관관계를 다시 보자.
        
        ![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%202.png)
        
        ![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%203.png)
        
    
    - **회원 엔티티에 있는 `Member.team` 필드를 주인으로 선택시**
        - 자기 테이블에 있는 외래키(`TEAM_ID`)를 관리하면 된다.
    
    - **팀 엔티티에 있는 `Team.members` 필드를 주인으로 선택시**
        - 물리적으로 전혀 다른 테이블의 외래 키를 관리해야 한다.
        - 왜냐하면 Team 엔티티는 TEAM 테이블에 매핑되어 있는데, 관리해야할 외래키는 MEMBER 테이블에 있기 때문이다.
        - 따라서, **연관관계의 주인은 `Member.team` 이다.**

<br/>

- 정리
    - MEMBER 테이블에 TEAM_ID라는 외래키가 존재하므로, 연관관계의 주인은 `Member.team` 필드이다.
        - 다시말해, 외래키가 존재하는 테이블과 매핑된 엔티티의 연관 필드가 연관 관계의 주인이 되는 것이 자연스럽다.
        - 왜냐하면, 반대쪽 엔티티가 가지고 있지 않은 외래키를 관리하는 것이 부자연스럽기 때문이다.
    - 연관관계의 주인만이 'DB 연관관계'와 매핑되고, '외래키를 관리(등록, 수정, 삭제)'할 수 있다.
    - 연관관계의 주인이 아닌 반대편은 읽기만 가능하고 외래키를 변경하지는 못한다.

<br/>

### 연관관계의 주인은 외래키가 있는 곳

- **연관관계의 주인은 테이블에 외래키가 있는 곳으로 정해야 한다.**

- 주인 설정하기
    - 주인이 아닌 필드: `mappedBy` 속성의 값으로 '주인인 필드의 이름'을 넘겨준다
    - 주인인 필드: 추가적으로 할 것은 없다.

<br/>

- 주인 설정 예시
    
    ```java
    @Entity
    public class Team {
    	//...
    
    	//일대다 관계 설정, 연관관계의 주인 필드명 설정
    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();
    
    	//...
    }
    ```
    
<br/>

### `mappedBy` 속성이 `@OneToMany`에만 존재하는 이유

- DB 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래키를 갖는다.
- **즉 `@ManyToOne` 이 항상 연관관계의 주인이 되므로, `@OneToMany` 에만 `mappedBy` 속성이 존재한다.**

<br/><br/>

## 양방향 연관관계 연산

### 저장

양방향 연관관계를 사용해서 팀1, 회원1, 회원2를 저장해보자.

```java
public void testSave() {
	
	//팀1 저장
	Team team1 = new Team("team1", "팀1");
	em.persist(team1);

	//회원1 저장
	Member member1 = new Member("member1", "회원1");
	member1.setTeam(team1); //연관관계 설정 (member1 -> team1)
	em.persist(member1);

	//회원2 저장
	Member member2 = new Member("member2", "회원2");
	member2.setTeam(team1); //연관관계 설정 (member2 -> team1)
	em.persist(member2);
}
```

- 이 코드는 단방향 연관관계에서 살펴본 코드와 완전히 같다.

<br/>

- DB 조회 결과
    - `SELECT * FROM MEMBER;`
    - **TEAM_ID 외래키에 팀의 기본키 값이 저장되어 있다.**

|MEMBER_ID|USERNAME|TEAM_ID|
|---------|--------|-------|
|member1|회원1|team1|
|member2|회원2|team1|

<br/>

### 양방향 연관관계의 주의점

- 양방향 연관관계는 연관관계의 주인이 외래키를 관리한다.
- 따라서, 아래 코드는 DB에 반영되지 않는다.
    
    ```java
    team1.getMembers().add(member1); //무시됨
    team1.getMembers().add(member2); //무시됨
    ```
    
- 아래 코드는 DB에 반영된다.
    
    ```java
    member1.setTeam(team1); //연관관계 설정됨
    member2.setTeam(team1); //연관관계 설정됨
    ```
    
- 양방향 연관관계를 설정하고 가장 흔히 하는 **실수가 바로, "연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하는 것"**이다.
    - 이런 경우, 연관관계 설정이 DB에 반영되지 않는다.

<br/>

### 순수한 객체까지 고려한 양방향 연관관계

- 위에서 연관관계의 주인에만 값을 설정해도, 연관관계 설정이 DB에 반영된다고 이야기했다.
- 그렇다면 연관관계의 주인인 필드에만 값을 설정해도 될까? 정답은 아니오이다.
- **사실 객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.**
    - 만약, 양쪽 방향에 값을 설정해주지 않는다면, 순수한 객체 관점에서는 문제가 일어날 수 있다.
    - 예를 들어, JPA를 사용하지 않는 테스트 코드를 작성할 경우에 문제가 발생할 수 있다.
    
    > ORM은 객체와 관계형 DB 모두 신경을 써야한다.
    > 

<br/>

- 문제 테스트코드 예시
    
    ```java
    @DisplayName("순수한객체 양방향")
    @Test
    public void test() {
    	
    	//팀1
    	Team team1 = new Team("team1", "팀1");
    	Member member1 = new Member("member1", "회원1");
    	Member member2 = new Member("member2", "회원2");
    
    	member1.setTeam(team1); //연관관계 설정
    	member2.setTeam(team1); //연관관계 설정
    
    	List<Member> members = team1.getMembers();
    	
    	assertEquals(members.size(), 2);
    }
    ```
    
    - 결과 = Fail (expected: 2, actual: 0)
    - 이 결과는 당연한 것이다. 왜냐하면, `Member.team` 에만 연관관계를 설정하고 반대 방향은 연관관계를 설정하지 않았기 때문이다.
    - 따라서, `members.size()` 의 값은 0이 나온다.

<br/>

- 이러한 상황을 해결하기 위해, 양쪽 모두 관계를 설정해서 다시 테스트해보자.
    
    ```java
    @DisplayName("순수한객체 양방향")
    @Test
    public void test() {
    	
    	//팀1
    	Team team1 = new Team("team1", "팀1");
    	Member member1 = new Member("member1", "회원1");
    	Member member2 = new Member("member2", "회원2");
    
    	member1.setTeam(team1); //연관관계 설정 (member1 -> team1)
    	team1.getMembers().add(member1); //연관관계 설정 (team1 -> member1)
    
    	member2.setTeam(team1); //연관관계 설정 (member2 -> team1)
    	team1.getMembers().add(member2); //연관관계 설정 (team1 -> member2)
    
    	List<Member> members = team1.getMembers();
    	
    	assertEquals(members.size(), 2);
    }
    ```
    
    - 결과 = Success (expected: 2, actual: 2)
    - 객체까지 고려하면 이렇게 양쪽 다 관계를 맺어야 한다. 단순히 DB의 경우만 생각해서는 안된다.

<br/>

- 이제 JPA를 사용해서 양쪽 관계를 모두 맺어보자.
    
    ```java
    public void testORM() {
    	
    	//팀1 저장
    	Team team1 = new Team("team1", "팀1");
    	em.persist(team1);
    
    	//회원1 양방향 관계 설정
    	Member member1 = new Member("member1", "회원1");
    	member1.setTeam(team1); //연관관계 설정 (member1 -> team1)
    	team1.getMembers().add(member1); //연관관계 설정 (team1 -> member1)
    	em.persist(member1);
    
    	//회원2 양방향 관계 설정
    	Member member2 = new Member("member2", "회원2");
    	member2.setTeam(team1); //연관관계 설정 (member2 -> team1)
    	team1.getMembers().add(member2); //연관관계 설정 (team1 -> member2)
    	em.persist(member2);
    }
    ```
    
    - 이제 순수한 객체 상태에서도 동작하고, 테이블의 외래키도 정상 입력된다.
        
        > 물론 외래키의 경우, `Member.team` 필드를 사용한다.
        > 

<br/>

- 정리
    - `member1.setTeam(team1);`
        - 연관관계의 주인
        - DB 저장시에 사용되고, 객체 관점에서도 사용된다.
    - `team1.getMembers().add(member1);`
        - 객체 관점에서만 사용된다.
        - 객체 상태를 위해, 작성된 코드이다.

<br/>

- 결론
    - **객체의 양방향 연관관계는 양쪽 모두 관계를 맺어주자.**
        - 객체와 테이블 모두 고려해야하므로

<br/><br/>

## 연관관계 편의 메서드

### 연관관계 편의 메서드란?

- 양방향 연관관계는 결국 양쪽 다 신경써야 한다.
- 하지만, 이것을 매번 반복하기엔 상당히 귀찮다.
- 따라서, 양쪽 다 신경쓰기 위한 로직을 메서드 하나로 처리할 수 있는 것을 **연관관계 편의 메서드**라 한다.

<br/>

### 연관관계 편의 메서드 예시

- Member 엔티티 수정
    
    ```java
    public class Member {
    	private Team team;
    
    	//setter 수정
    	public void setTeam(Team team) {
    		this.team = team; //기존 코드
    		team.getMembers().add(this); //Team쪽에도 연관관계 설정
    	}
    }
    ```
    

- 실행 코드
    
    ```java
    member1.setTeam(team1); //양쪽 모두 연관관계가 설정된다.
    member2.setTeam(team1); //양쪽 모두 연관관계가 설정된다.
    
    //== 기존 코드 삭제 ==
    //team1.getMembers().add(member1);
    //team1.getMembers().add(member2);
    //====================
    ```
    
<br/>

### 편의 메서드 작성시 주의사항

- 사실 우리가 방금 살펴본 `setTeam()` 메서드에는 버그가 있다.
- 버그 시나리오
    1. 회원1의 팀을 팀A로 설정한다.
    2. 그후, 회원1의 팀을 다시 팀B로 설정한다.
    3. 이때 팀A의 `members` 필드에 회원1이 그대로 남는다.
    - 아래 코드로 어떻게 버그가 발생하는지 알아보자.
    
    ```java
    member1.setTeam(teamA); //연관관계: (member1->teamA), (teamA->member1)
    member1.setTeam(teamB); //연관관계: (member1->teamB), (teamB->member1)
    Member findMember = teamA.getMembers(); //member1이 여전히 조회된다. (teamA->member1)
    ```
    

- 이 상황을 시각화한 것이 아래 그림이다.
    
    ![Untitled](/assets/img/2021-10-18-JPA_Relation_Basic/Untitled%204.png)
    
    - teamA의 members 필드가 여전히 member1을 가지고 있다. (참조하고 있다.)

<br/>

- 버그의 원인
    - teamB로 변경할 때, 'teamA→member1' 관계를 제거하지 않았다.
    - 즉 연관관계를 변경할 때는, 기존 팀이 있으면 **기존 팀과 회원의 연관관계를 삭제하는 코드를 추가**해야 한다.

- 버그 해결
    - 아래 코드는 기존 관계를 제거하는 코드가 포함된 `setTeam()` 편의 메서드이다.
    
    ```java
    public void setTeam(Team team) {
    	if (this.team != null) { //만약 기존팀이 존재한다면
    		this.team.getMembers().remove(this); //현재 인스턴스 객체를 지운다.
    	}
    
    	team.getMembers().add(this); //현재 인스턴스 객체를 새 팀에 추가한다.
    	this.team = team;
    }
    ```
    
<br/>

### 참고사항

- "teamA → member1" 관계가 제거되지 않아도 DB 외래키를 변경하는데는 문제없다.
- 왜냐하면, 외래키 변경은 연관관계의 주인(`Member.team`)에서 주관하기 때문이다.
- 즉, `Team.members` 가 연관관계의 주인이 아니기 때문에, DB 외래키는 변경이 정상적으로 된다.
- **하지만, 꼭 관계가 정리되도록 하자.**

<br/><br/>

## 정리

내용을 정리하면 다음과 같다.

- **단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료되었다.**
    - 연관관계의 주인을 통해, 연관 엔티티를 DB에 적용할 수 있으므로
- **단방향을 양방향으로 만들면, 반대방향으로 객체 그래프 탐색 기능이 추가된다.**
- **양방향 연관관계를 매핑하려면, 객체에서 양쪽 방향을 모두 관리해야 한다.**

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>