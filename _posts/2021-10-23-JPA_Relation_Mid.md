---
category: JPA
tags: [JPA]
title: "[JPA] 연관관계 매핑 - 중급"
date:   2021-10-23 20:36:00 
lastmod : 2021-10-23 20:36:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 포스팅
    - [[JPA] 연관관계 매핑 - 기초](https://taegyunwoo.github.io/jpa/JPA_Relation_Basic)

<br/><br/>

# 이전 내용 복습

본격적인 내용에 들어가기 전, 먼저 이전에 포스팅한 내용을 정리해보자.

엔티티의 연관관계를 매핑할 때는 다음 3가지를 고려해야 한다.

- 다중성
- 단방향, 양방향
- 연관관계의 주인

이전 내용을 하나씩 복습해보자.

<br/>

## 다중성

### 다중성의 종류

- 다대일
- 일대다
- 일대일
- 다대다

> 다대다 관계는 실무에서 거의 사용하지 않는다.
> 

<br/>

## 단방향, 양방향

### 테이블 vs 객체

- 테이블
    - 외래키 하나로 조인을 사용해서 양방향으로 쿼리가 가능하므로, 사실상 방향이라는 개념이 없다.
- 객체
    - 참조용 필드를 가지고 있는 객체만 연관된 객체를 조회할 수 있다.
    - 단방향
        - 한 쪽만 참조하는 것
    - 양방향
        - 양 쪽이 서로 참조하는 것

<br/>

## 연관관계의 주인

### 배경

- 테이블의 연관관계를 관리하는 포인트는 외래키 하나이다.
- 반면에 엔티티를 양방향으로 매핑하면 A→B, B→A 2곳에서 서로를 참조한다.
- **따라서, 객체의 연관관계를 관리하는 포인트는 2곳이다.**
    - 그렇기 때문에, 2곳 중 한 곳에서만 연관관계를 관리할 수 있도록 설정해야한다. ⇒ 연관관계의 주인

### 연관관계의 주인이란?

- JPA는 두 객체 연관관계 중 하나를 정해서 DB 외래키를 관리한다.
- 이것을 **연관관계의 주인**이라고 한다.
- '외래키를 가진 테이블과 매핑되는 엔티티'가 외래키를 관리하는 것이 효율적이다.
    - **그러므로, '외래키를 가진 테이블과 매핑되는 엔티티'가 연관관계의 주인이 된다.**
        
        > 정확히 말하자면 해당 엔티티의 외래키 필드가 연관관계의 주인이다.
        > 
- 연관관계의 주인은 `mappedBy` 속성을 사용하지 않는다.

이제부터 본격적으로 다양한 연관관계 매핑에 대해 설명하겠다.

<br/><br/><br/>

# 다양한 연관관계 매핑

## 다대일

### 개요

- **DB 테이블의 일(1), 다(N) 관계에서 외래키는 항상 다쪽에 있다.**
- **따라서, 객체 양방향 관계에서 연관관계의 주인은 항상 다쪽이다.**

<br/>

### 다대일 단방향 [N:1]

예시 코드를 통해, 다대일 단방향 연관관계에 대해 알아보자.

- 다대일 단방향 연관관계 시각화
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%205.png)

<br/>

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	@ManyToOne //다대일
    	@JoinColumn(name = "TEAM_ID") //FK 칼럼과 매핑
    	private Team team;
    
    	//getter, setter 생략
    }
    ```

- 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private Long id;
    
    	private String name;
    
    	//getter, setter 생략
    }
    ```

<br/>

- 연관 관계
    - 회원은 `Member.team` 으로 팀 엔티티를 참조할 수 있다.
    - 팀에서는 회원을 참조하는 필드가 없다.
    - 따라서, 회원과 팀은 다대일 단방향 연관관계이다.

<br/>

- `@JoinColumn(name = "TEAM_ID")`
    - 해당 애너테이션을 사용해서, `Member.team` 필드를 `TEAM_ID` 외래키와 매핑했다.
    - 따라서, `Member.team` 필드로 회원 테이블의 `TEAM_ID` 외래키를 관리한다.
        
        > 단방향 관계이기 때문에, 따로 연관관계의 주인을 명시할 필요는 없다.
        > 

<br/>

### 다대일 양방향 [N:1, 1:N]

예시 코드를 통해, 다대일 양방향 연관관계에 대해 알아보자.

- 다대일 양방향 연관관계 시각화
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%206.png)
    
    - 실선: 연관관계의 주인
    - 점선: 연관관계의 주인 X

<br/>

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	@ManyToOne //다대일
    	@JoinColumn(name = "TEAM_ID") //FK 칼럼과 매핑
    	private Team team;
    
    	//편의 메서드
    	public void setTeam(Team team) {
    		this.team = team;
    
    		if (!team.getMembers().contains(this)) { //무한루프에 빠지지 않도록 체크
    			team.getMembers().addMember(this);
    		}
    	}
    
    	//나머지 getter, setter 생략
    }
    ```
    
<br/>

- 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private Long id;
    
    	private String name;
    
    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();
    
    	//편의 메서드
    	public void addMember(Member member) {
    		this.members.add(member);
    
    		if (member.getTeam() != this) { //무한루프에 빠지지 않도록 체크
    			member.setTeam(this);
    		}
    	}
    
    	//나머지 getter, setter 생략
    }
    ```
    
<br/>

- **양방향은 외래키가 있는 쪽이 연관관계의 주인이다.**
    - 다시 한번 강조하지만, 일대다·다대일 관계에서는 항상 다(N)에 외래키가 존재한다.
    - 따라서, `Member.team` 이 연관관계의 주인이다.
    - JPA는 외래키를 관리할 때, 연관관계의 주인만 사용한다.

<br/>

- **양방향 연관관계는 항상 서로를 참조해야 한다.**
    - 어느 한 쪽만 참조하면 양방향 연관관계가 성립하지 않는다.
    - 항상 서로 참조하게 하려면 **연관관계 편의메서드를 작성하는 것**이 좋다.
        - 위 예시 코드에서는 회원의 `setTeam()` , 팀의 `addMember()` 메서드가 편의 메서드 역할을 한다.
    - 편의 메서드는 한 곳에만 작성하거나, 양쪽 다 작성할 수 있다.
        - **양쪽에 다 작성하면 무한루프에 빠지므로 주의해야 한다.**

<br/><br/>

## 일대다

### 일대다 단방향 [1:N]

- 하나의 팀은 여러 회원을 참조할 수 있다.
    - 이런 관계를 **일대다 관계**라고 한다.
- 그리고 팀은 회원들을 참조하지만, 반대로 회원에서 팀을 참조할 수는 없다.
    - 이런 관계를 **단방향 관계**라고 한다.

<br/>

- 일대다 단방향 연관관계 시각화
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%207.png)
    
    - **MEMBER 테이블의 외래키를 Team 엔티티가 관리한다.**
        - 즉, 반대쪽 테이블에 있는 외래 키를 관리한다.
    - MEMBER 테이블의 외래키를 Team 엔티티가 관리하는 이유
        - 일대다 관계에서 외래키는 항상 다쪽 테이블에 있다.
        - 하지만, 다 쪽인 **Member 엔티티에는 외래키를 매핑할 수 있는 참조 필드가 없다**.
        - 대신 반대쪽인 **Team 엔티티에만 참조 필드인 `members` 가 있다.**
        - 따라서 **반대편 테이블의 외래키를 관리하는 특이한 모습이 나타난다.**

<br/>

- 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private Long id;
    
    	private String name;
    
    	@OneToMany
    	@JoinColumn(name = "TEAM_ID") // 반대쪽 테이블의 FK를 매핑한다.
    	private List<Member> members = new ArrayList<Member>();
    
    	//getter, setter 생략
    }
    ```
    

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	//getter, setter 생략
    }
    ```

<br/>

- 상세 설명
    - 일대다 단방향 관계를 매핑할 때는 `@JoinColumn` 을 명시해야 한다.
    - 그렇지 않으면 JPA는 **연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑**한다.

<br/>

- 일대다 단방향 매핑의 단점
    - '매핑한 객체가 관리하는 외래키'가 다른 테이블에 있다.
    - 다른 테이블에 외래키가 있으면 연관관계 처리를 위한 **UPDATE SQL을 추가로 실행**해야 한다.

<br/>

- 일대다 단방향 매핑 연관관계 추가 예시
    - 예시 코드
        
        ```java
        public void testSave(EntityManager em) {
        
        	Member member1 = new Member("member1");
        	Member member2 = new Member("member2");
        	Team team1 = new Team("team1");
        
        	//연관관계 추가
        	team1.getMembers().add(member1);
        	team1.getMembers().add(member2);
        
        	//저장
        	em.persist(member1); // "INSERT: member1"
        	em.persist(member2); // "INSERT: member2"
        	em.persist(team1); // "INSERT: team1", "UPDATE: member1.fk", "UPDATE: member2.fk"
        	//즉, 위 코드를 커밋시 team1을 저장하고 난 뒤, member1과 2의 fk를 업데이트한다.
        
        	transaction.commit(); // 트랜잭션 커밋
        }
        ```
        
    
    - 커밋 결과 SQL
        
        ```sql
        //Member의 외래키로 어떤 team 엔티티가 사용됐는지 아직 모른다.
        INSERT INTO MEMBER (MEMBER_ID, USERNAME) VALUES (NULL, ?)
        INSERT INTO MEMBER (MEMBER_ID, USERNAME) VALUES (NULL, ?)
        
        //이제서야 어떤 team 엔티티인지 알수있다.
        INSERT INTO TEAM (TEAM_ID, NAME) VALUES (NULL, ?)
        
        //다시 team 엔티티를 연관관계로 추가한다.
        UPDATE MEMBER SET TEAM_ID=? WHERE MEMBER_ID=?
        UPDATE MEMBER SET TEAM_ID=? WHERE MEMBER_ID=?
        ```
        
        - member 엔티티가 저장될 때, 아직 team 엔티티를 모른다.
        - 그리고 연관 관계에 대한 정보는 team 엔티티의 `members` 필드가 관리한다.
        - **따라서 member 엔티티를 저장할 때는 MEMBER 테이블의 TEAM_ID 외래 키에 아무 값도 저장되지 않는다.**
        - **대신 team 엔티티를 저장할 때 `Team.members` 의 참조 값을 확인해서 회원 테이블에 있는 TEAM_ID 외래키를 업데이트한다.**

<br/>

- **따라서 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자.**
    - 위 코드처럼 작성하여 일대다 단방향 매핑을 구현하면, 관리가 까다롭고 성능상의 문제도 발생한다.
    - 따라서, 관리해야 하는 외래 키가 본인 테이블에 있는 '다대일 양방향 매핑'을 사용하자.

<br/>

### 일대다 양방향 [1:N, N:1]

- 일대다 양방향 매핑은 존재하지 않는다. 대신 다대일 양방향 매핑을 사용해야 한다.
    - 사실 '일대다 양방향'과 '다대일 양방향'은 같은 말이다.
    - 하지만 일(1)이 연관관계의 주인이 되도록 해보자.
- 양방향 매핑에서 `@OneToMany` 는 연관관계의 주인이 될 수 없다.
    - 왜냐하면, 항상 다(N)쪽에 외래 키가 있기 때문이다.
    - **따라서, `@OneToMany` 에는 `mappedBy` 속성이 없다.**
- 그래도 일대다 양방향 매핑이 완전히 불가능한 것은 아니다.
    - 일대다 단방향 매핑 반대편에 같은 외래키를 사용하는 다대일 단방향 매핑을 **읽기 전용으로 하나 추가**하면 된다.

<br/>

- 일대다 양방향 연관관계 시각화
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%208.png)
    

<br/>

- 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private Long id;
    
    	private String name;
    
    	@OneToMany
    	@JoinColumn(name = "TEAM_ID") // 반대쪽 테이블의 FK를 매핑한다.
    	private List<Member> members = new ArrayList<Member>();
    
    	//getter, setter 생략
    }
    ```
    

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	//읽기 전용으로 추가된 필드
    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    	private Team team;
    
    	//getter, setter 생략
    }
    ```
    
<br/>

- 상세 설명
    - 일대다 단방향 매핑 반대편에 **다대일 단방향 매핑을 추가**했다.
    - 이때 **일대다 단방향 매핑과 같은 TEAM_ID 외래키 컬럼을 매핑**한다.
    - 둘 다 같은 키를 관리하므로 문제가 발생할 수 있다.
        - 따라서 반대편인 다대일 쪽은 `insertable = false` , `updatable = false` 속성을 통해 읽기만 가능하게끔 한다.
    - 해당 방법은 일대다 양방향 매핑이라기보단, **일대다 단방향 매핑 반대편에 다대일 단방향 매핑을 읽기 전용으로 추가하여 일대다 양방향처럼 보이도록 하는 방법**이다.
    - **따라서, 일대다 단방향 매핑이 가지는 단점을 그대로 가진다.**

<br/>

- **될 수 있으면 다대일 양방향 매핑을 사용하자.**

<br/><br/>

## 일대일

### 일대일 관계란?

- 일대일 관계는 양쪽이 서로 하나의 관계만 가진다.
    - ex) 회원은 하나의 사물함만 사용하고, 사물함도 하나의 회원에 의해서만 사용된다.
        - 이때, '주 테이블 = 회원', '대상 테이블 = 사물함'이다.
- 일대일 관계의 특징
    - 일대일 관계는 그 반대도 일대일 관계다.
    - 일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느 곳이나 외래키를 가질 수 있다.
    - 테이블은 주 테이블이든 대상 테이블이든 외래키 하나만 있으면 양쪽으로 조회할 수 있다.
    - **주 테이블이나 대상 테이블 중, 누가 외래키를 가질지 선택해야 한다.**

<br/>

- **주 테이블에 외래키가 있는 경우**
    - 주 객체가 대상 객체를 참조하는 것처럼, 주 테이블에 외래키를 두고 대상 테이블을 참조한다.
    - 외래키를 객체 참조와 비슷하게 사용할 수 있다.
    - 주 테이블이 외래키를 가지고 있으므로, 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.
- **대상 테이블에 외래키가 있는 경우**
    - 테이블 관계를 일대일에서 일대다로 변경할 때, 테이블 구조를 그대로 유지할 수 있다.

<br/>

### 주 테이블에 외래키가 있는 경우

주 테이블에 외래키가 있으면 JPA에서 좀 더 편리하게 매핑할 수 있다.

<br/>

- **단방향**
    - MEMBER가 주 테이블이고, LOCKER는 대상 테이블이라고 가정하자.
    - 연관관계 시각화
        
        ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%209.png)
        
        - UNIQUE KEY : 유일한 값을 보장하는 제한조건, 중복X, null 허용
    
    - 회원 엔티티
        
        ```java
        @Entity
        public class Member {
        	@Id @GeneratedValue
        	@Column(name = "MEMBER_ID")
        	private Long id;
        
        	private String username;
        
        	@OneToOne
        	@JoinColumn(name = "LOCKER_ID")
        	private Locker locker;
        
        	//getter, setter 생략
        }
        ```
        
    - 사물함 엔티티
        
        ```java
        @Entity
        public class Locker {
        	@Id @GeneratedValue
        	@Column(name = "LOCKER_ID")
        	private Long id;
        
        	private String name;
        
        	//getter, setter 생략
        }
        ```
        
    - 상세 설명
        - `@OneToOne` 애너테이션을 통해, 일대일 관계로 객체 매핑을 하였다.
        - DB에는 LOCKER_ID 외래키에 유니크 제약 조건을 추가했다.
            - 일대일 관계이므로 유니크 제약 조건이 필요하다.
            - 만약 유니크 제약 조건이 없다면, 여러 회원이 하나의 사물함을 사용할 수 있게 된다.
        - 이 관계는 대체로 '다대일 단방향 연관관계'와 비슷하다.

<br/>

- **양방향**
    - 이번에도 역시 MEMBER가 주 테이블이고, LOCKER는 대상 테이블이라고 가정하자.
    - 연관관계 시각화
        
        ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2010.png)
        
    
    - 회원 엔티티
        
        ```java
        @Entity
        public class Member {
        	@Id @GeneratedValue
        	@Column(name = "MEMBER_ID")
        	private Long id;
        
        	private String username;
        
        	@OneToOne
        	@JoinColumn(name = "LOCKER_ID")
        	private Locker locker;
        
        	//getter, setter 생략
        }
        ```
        
    - 사물함 엔티티
        
        ```java
        @Entity
        public class Locker {
        	@Id @GeneratedValue
        	@Column(name = "LOCKER_ID")
        	private Long id;
        
        	private String name;
        
        	@OneToOne(mappedBy = "locker")
        	private Member member;
        
        	//getter, setter 생략
        }
        ```
        
    - 상세 설명
        - LOCKER_ID 외래키를 MEMBER 테이블이 가지고 있으므로, Member 엔티티의 `Member.locker` 를 연관관계의 주인으로 설정한다.
            - 양방향 관계이므로 연관관계의 주인을 설정해야 한다.
            - `Locker.member` 에 `mappedBy` 속성을 사용한다.
    
<br/>

### 대상 테이블에 외래키가 있는 경우

이번에는 대상 테이블에 외래키가 있는 일대일 관계에 대해 알아보자.

<br/>

- **단방향**
    - 일대일 관계 중 **대상 테이블에 외래키가 있는 단방향 관계는 JPA에서 지원하지 않는다.**
    - 따라서 이 경우 아래와 같은 방법으로 처리해야 한다.
        - 단방향 관계를 Locker 엔티티에서 Member 엔티티 방향으로 수정한다.
            
            ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2011.png)
            
        - 양방향 관계로 만들고 Locker를 연관관계의 주인으로 설정한다.
            
            ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2012.png)
            
    - 연관관계 시각화
        
        ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2013.png)
        
        > JPA 2.0부터 "일대다 단방향 관계에서 대상 테이블에 외래키가 있는 매핑"을 허용했다.  
        하지만, "일대일 단방향 관계에서 대상 테이블에 외래키가 있는 매핑"은 허용하지 않는다.
        > 

<br/>

- **양방향**
    - 연관관계 시각화
        
        ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2014.png)
        
    
    - 회원 엔티티
        
        ```java
        @Entity
        public class Member {
        	@Id @GeneratedValue
        	@Column(name = "MEMBER_ID")
        	private Long id;
        
        	private String username;
        
        	@OneToOne(mappedBy = "member")
        	private Locker locker;
        
        	//getter, setter 생략
        }
        ```
        
    - 사물함 엔티티
        
        ```java
        @Entity
        public class Locker {
        	@Id @GeneratedValue
        	@Column(name = "LOCKER_ID")
        	private Long id;
        
        	private String name;
        
        	@OneToOne
        	@JoinColumn(name = "MEMBER_ID")
        	private Member member;
        
        	//getter, setter 생략
        }
        ```
        
    
    - 상세설명
        - 일대일 매핑에서 대상 테이블에 외래키를 두고 싶으면, 이렇게 양방향으로 매핑하면 된다.
        - `mappedBy` 속성을 Member 엔티티의 `Member.locker` 필드에 적용하여, Locker 엔티티의 `Locker.member` 필드를 연관관계의 주인으로 설정하였다.
    
<br/><br/>

## 다대다

### 다대다 관계란

- 관계형 DB는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- 그래서 보통 다대다 관계를 **일대다, 다대일 관계로 풀어내는 연결 테이블을 사용**한다.
- 예시
    - 회원들이 상품을 주문한다. 그리고 상품들이 회원들에 의해 주문된다.
- 따라서 아래 그림처럼 중간에 연결 테이블을 추가해야 한다.
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2015.png)
    
<br/>

- `@ManyToMany` 애너테이션 사용시
    - 위 테이블 상태(연견 테이블 존재)에서, 해당 애너테이션을 사용하면 아래 그림처럼 다대다 관계를 편리하게 매핑할 수 있다.
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2016.png)
    
<br/>

### 다대다 단방향

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	@ManyToMany
    	@JoinTable(name = "MEMBER_PRODUCT",
    			joinColumns = @JoinColumn(name = "MEMBER_ID"),
    			inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    	private List<Product> products = new ArrayList<Product>();
    
    	//getter, setter 생략
    }
    ```
    
- 상품 엔티티
    
    ```java
    @Entity
    public class Product {
    	@Id @GeneratedValue
    	@Column(name = "PRODUCT_ID")
    	private Long id;
    
    	private String name;
    
    	//getter, setter 생략
    }
    ```
    

<br/>

- `@JoinTable` 애너테이션
    - 해당 애너테이션을 통해, 연결테이블을 편리하게 처리한다.
    - `@JoinTable.name` 속성
        - 연결 테이블을 지정한다.
        - 위 코드에서는 MEMBER_PRODUCT 연결 테이블을 지정했다.
    - `@JoinTable.joinColumns` 속성
        - '현재 방향인 회원'과 매핑할 조인 컬럼 정보를 지정한다.
        - 위 코드에서는 MEMBER_ID로 지정했다.
    - `@JoinTable.inverseJoinColumns` 속성
        - '반대 방향인 상품'과 매핑할 조인 컬럼 정보를 지정한다.
        - 위 코드에서는 PRODUCT_ID로 지정했다.

<br/>

- 상세설명
    - 회원 엔티티와 상품 엔티티를 `@ManyToMany` 와 `@JoinTable` 로 편리하게 매핑했다.
    - 이와 동시에, 연결 테이블까지 바로 매핑되었다.
    - **따라서, 회원과 상품을 연결하는 '회원_상품(Member_Product) 엔티티'(중간 연결 엔티티) 없이 매핑을 완료할 수 있다.**

<br/>

- **다대다 관계 연산: 저장**
    - 다음으로 다대다 관계를 저장하는 예제를 보자.
        
        > 참고로, 단방향 관계에 한정되는 내용은 아니다. 다대다 양방향 관계에서도 적용되는 내용이다.
        > 
    
    ```java
    public void save() {
    	//일반 상품 저장
    	Product productA = new Product();
    	productA.setId("productA");
    	productA.setName("상품A");
    	em.persist(productA);
    
    	//회원저장
    	Member member1 = new Member();
    	member1.setId("member1");
    	member1.setName("회원1");
    	member1.getProducts().add(productA) //연관관계 설정
    	em.persist(member1);
    }
    ```
    
    - 회원1과 상품A의 연관관계를 설정했으므로 **회원1을 저장할 때 연결 테이블에도 값이 저장**된다.
    - 따라서, 위 코드를 실행하면 다음과 같은 SQL이 실행된다.
    
    ```sql
    INSERT INTO PRODUCT ... //상품저장
    INSERT INTO MEMBER ... //회원저장
    INSERT INTO MEMBER_PRODUCT ... //연결테이블에도 저장
    ```
    
<br/>

- **다대다 관계 연산: 조회 및 탐색**
    - 다음으로, 엔티티를 조회하고 그래프 탐색을 해보자.
        
        > 참고로, 단방향 관계에 한정되는 내용은 아니다. 다대다 양방향 관계에서도 적용되는 내용이다.
        > 
    
    ```java
    public void find() {
    	Member member = em.find(Member.class, "member1");
    	List<Product> products = member.getProducts(); //객체 그래프 탐색
    	
    	//정보 출력
    	for (Product product : products) {
    		System.out.println("product name: " + product.getName());
    	}
    }
    ```
    
    - 결과: 저장해두었던 상품1이 조회된다.
    - `member.getProducts()` 를 호출할 때, 아래와 같은 SQL이 실행된다.
    
    ```sql
    SELECT * FROM MEMBER_PRODUCT MP
    	INNER JOIN PRODUCT P ON MP.PRODUCT_ID=P.PRODUCT_ID
    	WHERE MP.MEMBER_ID=?
    ```
    
    - **즉, `member.getProducts()` 를 호출하면 관련된 Product 엔티티를 조회하여 제공해주어야 한다.** 따라서, 위 SQL이 실행된다.
    - **위 SQL은 '연결 테이블인 MEMBER_PRODUCT'와 '상품 테이블'을 조인해서 연관된 상품을 조회한다.**

<br/>

### 다대다 양방향

- 양방향 매핑을 하려면 늘 그래왔듯이 `mappedBy` 속성을 이용하면 된다.
    - 양쪽다 다(N)이므로, 원하는 곳에 적용하면 된다.

<br/>

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	@ManyToMany
    	@JoinTable(name = "MEMBER_PRODUCT",
    			joinColumns = @JoinColumn(name = "MEMBER_ID"),
    			inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    	private List<Product> products = new ArrayList<Product>();
    
    	//편의 메서드
    	public void addProduct(Product product) {
    		// ...
    		products.add(product);
    		product.getMembers().add(this);
    	}
    
    	//getter, setter 생략
    }
    ```
    
- 상품 엔티티
    
    ```java
    @Entity
    public class Product {
    	@Id @GeneratedValue
    	@Column(name = "PRODUCT_ID")
    	private Long id;
    
    	private String name;
    
    	//역방향 추가
    	@ManyToMany(mappedBy = "products")
    	private List<Member> members = new ArrayList<Member>();
    
    	//getter, setter 생략
    }
    ```
    
<br/>

### 다대다 매핑의 한계와 극복: 배경

- 위에서 `@ManyToMany` 애너테이션을 통해, 다대다 매핑을 손쉽게 했다.
- 하지만 이러한 방식을 실무에서 사용하기엔 한계가 있다.

<br/>

- 한계
    - 보통 연결 테이블에 `MEMBER_ID` 와 `PRODUCT_ID` 같은 칼럼만 존재하지는 않다.
    - 대부분 그외의 일반 칼럼 (ex. 주문일자, 주문 수량 등)이 연결 테이블에 포함된다.
    - 즉, 이것을 ERD로 표현하면 아래와 같다.
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2017.png)
    
    - **이렇게 칼럼이 추가되면 더는 `@ManyToMany` 를 사용할 수 없다!**
        - 왜냐하면, 추가된 칼럼들을 매핑할 수 없기 때문이다.
        - 그리고 추가된 칼럼들은 `@JoinTable` 로 처리가 불가능한 것들이다.
    - 따라서, 결국에는 **연결 테이블을 매핑하는 연결 엔티티를 만들고, 이곳에 추가한 컬럼들을 매핑해야 한다.**
    - 또한, **엔티티 간의 관계도 테이블 관계처럼 일대다, 다대일로 풀어내야 한다.**
    - 이것을 그림으로 표현하면 아래와 같다.
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2018.png)
    
    > 참고로, 'Product→MemberProduct' 방향은 비즈니스 로직상 필요없다고 판단되어, 단방향으로 설정하였다.
    > 

<br/>

### 다대다 매핑의 한계와 극복: 연결엔티티 사용

위와 같은 한계가 존재하기 때문에, 연결 엔티티를 사용하여 매핑해야 한다.

<br/>

- DB 테이블 상태가 아래와 같다고 하자. (위에서 살펴본 것과 동일하다.)
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2017.png)
    
<br/>

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	//연결엔티티와 연관관계 설정
    	@OneToMany(mappedBy = "member")
    	private List<MemberProduct> memberProducts = new ArrayList<MemberProduct>();
    
    	//getter, setter 생략
    }
    ```
    
    - '회원엔티티→연결엔티티' 방향으로 접근이 가능하도록 설정했다.
    - '연결엔티티→회원엔티티' 방향으로도 접근이 가능하게 할 것임으로, **양방향 관계**이다.
    - 따라서, FK가 존재하는 테이블인 MEMBER_PRODUCT와 매핑되는 MemberProduct 엔티티를 연관관계의 주인으로 설정했다.
    - 연결엔티티인 `MemberProduct` 엔티티에 대해선 아래에서 자세히 알아보자.

<br/>

- 상품 엔티티
    
    ```java
    @Entity
    public class Product {
    	@Id @GeneratedValue
    	@Column(name = "PRODUCT_ID")
    	private Long id;
    
    	private String name;
    
    	//getter, setter 생략
    }
    ```
    
    - '상품엔티티→연결엔티티' 방향으로 접근할 수 없다.
    - '연결엔티티→상품엔티티' 방향으로는 접근할 수 있도록 할 것이므로, **단방향 관계**이다.
        
        > 비즈니스 로직상 필요없다고 판단되어, 단방향으로 설정하였다.
        > 

<br/>

- 회원상품 엔티티 (연결엔티티)
    
    ```java
    @Entity
    @IdClass(MemberProductId.class) //복합기본키 매핑을 위한 식별자 클래스 설정
    public class MemberProduct {
    
    	@Id
    	@ManyToOne
    	@JoinColumn(name = "MEMBER_ID")
    	private Member member; //MemberProductId.member와 연결
    
    	@Id
    	@ManyToOne
    	@JoinColumn(name = "PRODUCT_ID")
    	private Product product; //MemberProductId.product와 연결
    
    	private int orderAmount;
    
    	//getter, setter 생략
    }
    ```
    
<br/>

- 회원상품 식별자 클래스
    
    ```java
    public class MemberProductId implements Serializable {
    
    	//아래 변수들은 각 엔티티의 기본키를 담는다. (String 형임에 주의하자.)
    	private String member; //MemberProduct.member와 연결
    	private String product; //MemberProduct.product와 연결
    
    	@Override
    	public boolean equals(Object o) { ... }
    
    	@Override
    	public int hashCode() { ... }
    	
    }
    ```
    
    - JPA에서 복합 기본키를 사용하기 위해, 만들어진 식별자 클래스
    
    > 자세한 것은 아래에서 설명한다.
    > 

<br/>

- 코드 상세 설명
    - 회원상품 엔티티 (연결 엔티티)
        - `@Id` 와 `@JoinColumn` 을 동시에 사용해서 기본키+외래키를 한번에 매핑한다.
        - `@IdClass` 를 사용해서, **복합 기본키를 매핑**한다.

<br/>

- **복합 기본키**
    - 복합 기본키란, 기본키가 여러 칼럼인 것을 뜻한다.
        - 단순히 기본키로 여러 개의 칼럼이 사용되는 것을 뜻한다.
    - 회원상품 엔티티는 **"기본키가 MEMBER_ID와 PRODUCT_ID로 이루어진 복합 기본키"** 를 갖는다.
    - JPA에서의 복합 기본키 사용법
        1. 별도의 식별자 클래스 만들기
        2. 엔티티에 `@IdClass` 를 사용해서 식별자 클래스를 지정하기

<br/>

- **식별자 클래스의 특징**
    - 식별자 클래스를 통해서, 여러 기본키를 하나의 복합 기본키로 묶는다.
    - 복합키는 별도의 식별자 클래스로 만들어야한다.
    - `Serializable` 을 구현해야 한다.
    - `equals` 와 `hashCode` 메서드를 구현해야 한다.
    - 기본 생성자가 있어야 한다.
    - public 클래스여야 한다.
    - `@IdClass` 대신, `@EmbeddedId` 를 사용하는 방법도 있다.

<br/>

- **식별 관계**
    - 회원상품은 회원과 상품의 기본키를 받아서 자신의 기본키로 사용한다.
    - 이렇게 **부모 테이블의 기본키를 받아서, '자신의 기본키 + 외래키'로 사용하는 것을 DB용어로 "식별관계"** 라고 한다.

<br/>

- 예시코드 정리
    - **회원상품은 회원의 기본키를 받아서, 자신의 기본키로 사용함과 동시에 회원과의 관계를 위한 외래키로 사용한다.**
    - **회원상품은 상품의 기본키를 받아서, 자신의 기본키로 사용함과 동시에 회원과의 관계를 위한 외래키로 사용한다.**
    - **`MemberProductId` 식별자 클래스로 두 기본키를 묶어서 복합 기본키로 사용한다.**

<br/>

- 연결엔티티를 사용한 관계: 저장 예시 코드
    - 위처럼 구성한 관계를 어떻게 저장하는지 살펴보자. 예시코드는 아래와 같다.
    
    ```java
    public void save() {
    	//일반 상품 저장
    	Product productA = new Product();
    	productA.setId("productA");
    	productA.setName("상품A");
    	em.persist(productA);
    
    	//회원 저장
    	Member member1 = new Member();
    	member1.setId("member1");
    	member1.setName("회원1");
    	em.persist(member1);
    
    	//회원상품 저장
    	MemberProduct memberProduct = new MemberProduct();
    	memberProduct.setMember(member1); //주문 회원 - 연관관계 설정
    	memberProduct.setProduct(productA); //주문 상품 - 연관관계 설정
    	memberProduct.setOrderAmount(2); //주문 개수 - 일반 칼럼 값 설정
    	em.persist(memberProduct);
    }
    ```
    
    - 회원상품 엔티티를 만들면서, 연관된 회원 엔티티와 상품 엔티티를 설정했다.
    - 회원상품 엔티티는 DB에 저장될 때, 연관된 회원의 기본키와 상품의 기본키를 가져와서 자신의 기본키 값으로 사용한다.

<br/>

- 연결엔티티를 사용한 관계: 조회 예시 코드
    - 위처럼 저장했을 때, 어떻게 조회하는지 살펴보자. 예시코드는 아래와 같다.
    
    ```java
    public void find() {
    	//복합 기본키(식별자 클래스 객체) 생성
    	MemberProductId memberProductId = new MemberProductId();
    	memberProductId.setMember("member1"); //member 엔티티의 기본키 값 설정
    	memberProductId.setProduct("productA"); //product 엔티티의 기본키 값 설정
    
    	//식별자 클래스 객체를 사용하여 조회
    	//조회할때 사용될 기본키로 식별자 클래스 객체를 넘겨준다!
    	MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);
    
    	//객체 그래프 탐색
    	Member member = memberProduct.getMember();
    	Product product = memberProduct.getProduct();
    }
    ```
    
    - **`em.find()` 부분을 보면 `MemberProductId` 식별자 클래스의 인스턴스를 통해 조회한다.**

<br/>

- 이러한 과정은 **단순히 칼럼 하나만 기본키로 사용하는 것에 비해 너무 복잡하다!**
- 좀 더 쉽고 편리한 방법에 대해 계속해서 알아보자.

<br/>

### 다대다: 새로운 기본키 사용

위에서 **복합 기본키 사용을 위해, 식별자 클래스**를 생성하여 매핑하였다. 하지만 너무 복잡하다는 단점이 있다. 따라서, 다른 방법을 알아보자.

- 복합 기본키를 연결테이블의 기본키로 사용하는 방법 대신, 추천하는 기본키 생성 전략은 **DB에서 자동으로 생성해주는 대리키를 Long 값으로 사용하는 것**이다.
    - 이것은 이전 방법보다 간편하고 거의 영구히 사용할 수 있으며, 비즈니스에 의존하지 않는다.
    - 무엇보다 복합 기본키를 만들지 않아도 된다.

이제부터 예시코드를 통해 알아보자.

> 이제 연결엔티티의 이름을 '주문'으로 변경하자. 앞으로 진행하는 내용을 수행하면 '회원상품'이라는 이름보단 '주문'이라는 이름이 더 잘 어울릴 것이다.
> 

<br/>

- 테이블 관계 시각화
    
    ![Untitled](/assets/img/2021-10-23-JPA_Relation_Mid/Untitled%2019.png)
    
    - `ORDER_ID` 칼럼이 새로 추가되어, 대리키로서의 역할을 수행한다. (AUTO INCREMENT 등을 통해, DB가 값을 자동으로 생성해준다.)
    - `MEMBER_ID` , `PRODUCT_ID` 칼럼은 단순 FK로만 변경되었다.

<br/>

- 주문 엔티티
    
    ```java
    @Entity
    public class Order {
    
    	//새 기본키
    	@Id @GeneratedValue
    	@Column(name = "ORDER_ID")
    	private Long id;
    
    	@ManyToOne
    	@JoinColumn(name = "MEMBER_ID")
    	private Member member; //연관관계의 주인이다.
    
    	@ManyToOne
    	@JoinColumn(name = "PRODUCT_ID")
    	private Product product; //연관관계의 주인이다.
    
    	private int orderAmount;
    
    	//getter, setter 생략
    }
    ```
    
    - 대리키를 사용함으로써 이전 방식보다 훨씬 매핑이 단순하고 이해하기 쉽다.
        - 이전 방식
            - 연결엔티티의 PK·FK = 회원, 상품
            - 따라서, 식별자 클래스를 통해 복합키를 사용한다.
        - 현재 방식
            - 연결엔티티의 PK = DB가 자동생성해준 값 (대리키)
            - 연결엔티티의 FK = 회원, 상품
            - 복합키를 사용하지 않기 때문에, 식별자 클래스가 필요없다.

<br/>

- 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    
    	private String username;
    
    	//연결엔티티와 연관관계 설정
    	@OneToMany(mappedBy = "member")
    	private List<Order> orders = new ArrayList<Order>(); //이름변경
    
    	//getter, setter 생략
    }
    ```
    
    - 연결 엔티티의 이름이 '회원상품'에서 '주문'으로 변경되었으므로, orders 필드 이름을 변경한다.

<br/>

- 상품 엔티티
    
    ```java
    @Entity
    public class Product {
    	@Id @GeneratedValue
    	@Column(name = "PRODUCT_ID")
    	private Long id;
    
    	private String name;
    
    	//getter, setter 생략
    }
    ```
    
    - 단방향이기 때문에, 변경사항 없다.

<br/>

- 저장 예시
    
    ```java
    public void save() {
    	//일반 상품 저장
    	Product productA = new Product();
    	productA.setId("productA");
    	productA.setName("상품A");
    	em.persist(productA);
    
    	//회원 저장
    	Member member1 = new Member();
    	member1.setId("member1");
    	member1.setName("회원1");
    	em.persist(member1);
    
    	//주문 저장
    	Order order = new Order();
    	order.setMember(member1); //주문 회원 - 연관관계 설정
    	order.setProduct(productA); //주문 상품 - 연관관계 설정
    	order.setOrderAmount(2); //주문 개수 - 일반 칼럼 값 설정
    	em.persist(order);
    }
    ```
    
    - 저장의 경우, 기존 코드와 크게 달라진 것은 없다.

<br/>

- 조회 예시
    
    ```java
    public void find() {
    
    	Long orderId = 1L;
    	MemberProduct memberProduct = em.find(MemberProduct.class, orderId); //단순화됨
    
    	//객체 그래프 탐색
    	Member member = memberProduct.getMember();
    	Product product = memberProduct.getProduct();
    }
    ```
    
    - 식별자 클래스를 사용하지 않아서 코드가 한결 단순해졌다.

<br/>

- 다대다 연관관계 정리
    - 다대다 관계를 일대다 다대일 관계로 풀어내기 위해, 연결 테이블을 만들 때 식별자를 어떻게 구성할지 선택해야 한다.
    - 식별자 구성 종류
        - **식별 관계** : 받아온 식별자를 기본키+외래키로 사용한다.
        - **비식별 관계** : 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가한다.
    - **비식별 관계로 다대다 연관관계를 구성하는 것이 단순하고 편리하다.**

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>