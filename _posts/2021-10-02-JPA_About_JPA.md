---
category: JPA
tags: [JPA]
title: "[JPA] JPA 개요"
date:   2021-10-02 20:06:00 
lastmod : 2021-10-02 20:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# JPA 소개

- JPA를 소개하기 전, JPA 없이 JDBC를 직접 사용하면 발생하는 문제에 대해 알아보자.
- 즉, JPA의 필요성에 대해 먼저 알아보자.

<br/><br/>

## SQL을 직접 다룰 때의 문제점

### JDBC를 직접 사용하는 애플리케이션 예시

- Member 클래스
    
    ```java
    public class Member {
      private String memberId;
      private String name;
      //... 기타 코드
    }
    ```

- MemberDAO 클래스
    
    ```java
    public class MemberDAO {
      public Member find(String memberId) { ... }
      public Member save(Member member) { ... }
      public Member update(Member member) { ... }
      public Member delete(Member member) { ... }
    }
    ```

    - `find(String memberId)` 메서드를 완성한다고 가정해보자. 그 과정은 아래와 같다.
        - `find(String memberId)` 메서드을 작성 절차
            1. 회원 조회용 SQL 작성한다.
            2. JDBC API를 사용하여 SQL을 실행한다.
            3. 조회 결과를 Member 객체로 매핑한다.
    - `save(Member member)` 메서드를 완성한다고 가정해보자. 그 과정은 아래와 같다.
        - `save(Member member)` 메서드을 작성 절차
            1. 회원 등록용 SQL 작성한다.
            2. 회원 객체의 값(property)을 꺼내서 등록 SQL에 전달한다.
            3. JDBC API를 사용하여 SQL을 실행한다.
    
    > `update(Member member)` 와 `delete(Member member)` 는 생략한다.

<br/>

### 이때 발생하는 문제

- 객체 자체를 DB에 바로 저장할 수 없기 때문에, 일일히 객체의 속성값(property)를 꺼내서 SQL에 대입한 뒤, SQL을 실행해야한다.
    - **이 과정에서 너무 많은 SQL과 JDBC API를 코드로 작성해야 한다.**

<br/>

- 만약 "회원의 연락처를 함께 저장해야한다"는 요구사항이 추가된다면 어떻게 될까?
    - 추가된 요구사항에 의해 변경되는 사항
        - 연락처를 저장하기 위해 `save(Member member)` 메서드의 SQL을 수정해야한다.
        - 연락처를 출력하기 위해 `find(String memberId)` 메서드의 SQL을 수정해야한다.
        - 연락처를 수정하기 위해 `update(Member member)` 메서드의 SQL을 수정해야한다.
    - **너무 많은 코드를 수정해야한다.**

<br/>

- 만약 "회원은 무조건 어떤 한 팀에 소속되어야한다"는 요구사항이 추가된다면 어떻게 될까?
    - 추가된 요구사항에 의해 변경되는 사항
        - 팀을 표현하기 위해, Team 객체 및 테이블이 추가된다. Member 테이블과 Team 테이블이 서로 외래키로 연관된다.
        - **소속 팀을 회원과 함께 조회**하기 위해 `findWithTeam(Member member)` 메서드가 추가되고, **'Member 테이블' 과 'Team 테이블'을 Join 시켜주는 SQL을 작성**해야한다.
        - 회원 출력시, 소속 팀을 함께 출력하려면 `find(Member member)`메서드를 사용하던 모든 부분을 `findWithTeam(Member member)`메서드로 수정해야한다.
            
            > 왜냐하면, `find(Member member)` 메서드는 Team 테이블을 함께 조회하지 않기 때문이다.  
            (즉, Team 테이블을 Join하는 쿼리가 필요하다. 그것을 바로 `findWithTeam(Member member)` 메서드가 가지고 있다.)
            > 
    - **따라서 Member 객체가 연관된 Team 객체를 사용할 수 있을지, 없을지는 전적으로 SQL에 달려있다.**
        - SQL이 Join을 통해 Team 테이블을 같이 조회해주어야 하므로.
    - **데이터 접근 계층(DAO)을 사용해서 SQL을 숨겨도 어쩔 수 없이, DAO를 열어서 어떤 SQL이 실행되는지 확인해야한다.**

<br/>

- 발생 문제 정리
    - **진정한 의미의 계층 분할이 어렵다.**
    - **엔티티를 신뢰할 수 없다.**
        
        > 왜냐하면, 특정 프로퍼티를 사용할 수 있을지를 SQL이 결정하므로.
        > 
    - **SQL에 의존적인 개발을 피하기 어렵다.**

<br/><br/>

## JPA를 통한 문제해결

- JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다.
- JPA를 통해, 위 문제점들을 해결할 수 있다.
- 코드에 집중하지 말고, 기능적인 부분에 초점을 맞추어 읽자.

<br/>

### 저장 기능

```java
jpa.persist(member); //회원 저장 코드
```

- JPA가 객체와 매핑정보를 보고 적절한 INSERT SQL을 생성해서 데이터베이스에 전달한다.
- 매핑정보는 어떤 객체를 어떤 테이블에 관리할지 정의한 정보이다.

<br/>

### 조회 기능

```java
String memberId = "helloId";
Member member = jpa.find(Member.class, memberId);
```

- JPA는 객체와 매핑정보를 보고 적절한 SELECT SQL을 생성해서 DB에 전달한다.
- 그리고 그 결과로 Member 객체를 생성해서 반환한다.

<br/>

### 수정 기능

```java
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경"); //트랜잭션 커밋시 수정된다.
```

- JPA는 별도의 수정 메서드를 제공하지 않는다.
- 대신 객체를 조회해서 값을 변경만 하면 트랜잭션을 커밋할 때 DB에 적절한 UPDATE SQL이 전달된다.

<br/>

### 연관된 객체 조회

```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```

- JPA는 연관된 객체(Team)를 사용하는 시점에 적절한 SELECT SQL을 실행한다.
- 따라서 JPA를 사용하면 연관된 객체를 마음껏 조회할 수 있다.
- 즉, Member 조회시 Team 테이블을 `JOIN` 하였는지를 고려하지 않아도 Team 객체에 접근할 수 있다.

<br/><br/>

## 패러다임의 불일치

### 패러다임의 불일치란?

- 관계형 데이터베이스
    - 데이터 중심으로 구조화가 되어있다.
    - 추상화, 상속, 다형성과 같은 개념이 없다.
- 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다.
- 이것을 객체와 관계형 데이터베이스의 패러다임 불일치 문제라고 한다.
- **따라서, 객체 구조를 테이블 구조에 저장하는 데는 한계가 있다.**

<br/>

### 패러다임 불일치: 상속문제

- 객체는 상속이라는 기능을 가지고 있다.
- 하지만, 테이블은 상속이라는 기능이 없다.
- **상속관계를 테이블에 표현하기 (JPA 없이)**
    - `Item` 을 상속받는 `Movie` , `Book` 객체가 존재한다고 하자.
        
        ![Untitled](/assets/img/2021-10-02-JPA_About_JPA/Untitled.png)
        
    - 이것을 테이블로 표현하는 최선의 방법은 다음과 같다.
        
        ![Untitled](/assets/img/2021-10-02-JPA_About_JPA/Untitled%201.png)
        
        - `dtype` 칼럼을 사용해서 어떤 자식 테이블과 관계가 있는지 정의한다.
            - 예를 들어, `dtype` 값이 Movie면 영화 테이블과 관계가 있다.
    - Movie 객체를 저장하려면 이 객체를 분해해서 다음 두 SQL을 만들어야 한다. 왜냐하면, name과 price 칼럼의 값을 director, actor와 함께 저장해야하기 때문이다.
        - INSERT INTO ITEM ...
        - INSERT INTO MOVIE ...
    - **즉, `dtype`을 통해 직접 상속을 표현하는 것은 상당히 귀찮고 복잡하다.**

<br/>

### **JPA를 통해 상속 문제 해결하기**

- JPA는 상속과 관련된 패러다임의 불일치 문제를 대신 해결해준다.

```java
jpa.persist(movie);
```

- 위 코드 한줄만 입력하면, JPA는 자동으로 다음 두 SQL을 실행해서 ITEM과 MOVIE 테이블에 저장한다.
    - INSERT INTO ITEM ...
    - INSERT INTO MOVIE ...
- 추가로, Movie 객체를 조회해보자.

```java
jpa.find(Movie.class, id);
```

- 위 코드 한줄만 입력하면, JPA가 ITEM과 MOVIE 테이블을 조인해서 데이터를 조회하고 결과를 반환한다.
    - SELECT I.*, M.* FROM ITEM I JOIN MOVIE M ON I.ID = M.ID

<br/>

### 패러다임 불일치: 연관관계 문제

- 객체는 참조(주소값)를 사용해서 다른 객체와 연관관계를 가지고 **참조에 접근해서 연관된 객체를 조회**한다.
- 테이블은 외래키를 사용해서 다른 테이블과 연관관계를 가지고 **조인을 사용해서 연관된 테이블을 조회**한다.

```java
public class Member {
	String id;
	Team team;
	String username;

	//...
}
```

```java
public class Team {
	Long id;
	String name;
}
```

- 위와 같이, `Member` 객체에 `Team` 객체가 프로퍼티로 존재한다.
- 이것을 어떻게 테이블로 표현해야할까?

<br/>

### **JPA를 통해 연관관계 문제 해결하기**

- JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다.

```java
member.setTeam(team); //Member와 Team 간의 연관관계 설정 (참조설정)
jpa.persist(member); //Member와 연관관계를 함께 저장
```

- 위 코드를 통해, 연관관계를 함께 저장할 수 있다.
- JPA는 team의 참조를 외래키로 변환해서 적절한 INSERT SQL을 DB에 전달한다.
- 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.

```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```

<br/>

### 패러다임 불일치: 객체 그래프 탐색 문제

- **객체에서 회원이 소속된 팀을 조회할 때, 다음처럼 참조를 사용해서 연관된 팀을 찾는 것**을 객체 그래프 탐색이라고 한다.
- 예시
    
    `member.getOrder().getOrderItem().getItem(). ...`
    
- 객체는 마음껏 객체 그래프를 탐색할 수 있어야 하는데, 이것을 관계형 DB도 지원할까? 정답은 아니다.
    - SELECT M.*, O.* FROM MEMBER M **JOIN** Order O ON M.ID = O.ID
    - 위 SQL로 member 객체를 조회했다면, 그와 연관된 order 객체는 가져올 수 있지만, **order 객체와 연관된 또 다른 객체는 접근할 수 없다.**
    - 또한, **member 객체와 연관된 또다른 객체 team이 존재할 때, 해당 SQL로 접근할 수 없다.**
    - 위 SQL은 오직 order 객체에만 접근할 수 있다.
- **즉, SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.**
- **따라서 어디까지 객체 그래프 탐색이 가능한지 알아보려면, 결국 데이터 접근 계층인 DAO를 열어서 SQL을 직접 확인해야 한다.**

<br/>

### **JPA를 통해 객체 그래프 탐색 문제 해결하기**

- JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. 이것은 지연로딩 기능이 가능케 한다.
    - **지연 로딩이란?**
        - 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미루는 기능
        - 지연 로딩을 통해 직접 사용해야하는 코드는 없다. (JPA가 지연로딩을 투명하게 처리한다.)
- 따라서, JPA를 사용하면 연관된 객체를 신회하고 마음껏 조회할 수 있다.

```java
//처음 조회 시점에 SELECT MEMBER SQL을 실행한다.
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); //Order 객체를 사용하는 이 시점에 SELECT ORDER SQL을 실행한다.
```

- 이러한 지연로딩 대신, 즉시 연관 객체를 조회하도록 설정할 수도 있다.

<br/>

### 패러다임 불일치: 비교 문제

- 데이터베이스는 **기본 키의 값으로 각 로우(row)를 구분**한다.
- 객체는 **동일성 비교**와 **동등성 비교**하는 두 가지 비교 방법이 있다.
    - **동일성 비교**
        - 객체A == 객체B
        - 참조값(주소값)을 비교한다.
    - **동등성 비교**
        - 객체A.equals(객체B)
        - 객체의 내용을 비교한다.
- 따라서, 테이블의 로우를 구분하는 방법과 객체를 구분하는 방법에는 차이가 있다.

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
member1 == member2; //false: 서로 다른 객체이다.
```

- `member1` 과 `member2` 은 서로 같은 memberId로 조회했음에도 다른 객체로 판단된다.
    - 왜냐하면, `member1`과 `member2` 은 같은 데이터베이스 로우에서 조회했지만, 객체 측면에서 볼 때 둘은 다른 인스턴스이기 때문이다.

<br/>

### **JPA를 통해 비교 문제 해결하기**

- JPA는 같은 트랜잭션일 때, 같은 객체가 조회되는 것을 보장한다.

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);
member1 == member2; //true: 서로 같은 객체이다.
```

<br/>

### 정리

- 객체 모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 서로 다르다.
- 이 패러다임의 차이를 극복하려면 개발자가 너무 많은 시간과 코드를 소비해야한다.
- 뿐만 아니라, 객체지향 애플리케이션답게 정교한 객체 모델링을 할수록 패러다임의 불일치 문제가 더 커진다.
- JPA는 패러다임의 불일치 문제를 해결해주고 정교한 객체 모델링을 유지하게 도와준다.

<br/><br/>

## JPA란 무엇인가

- 지금까지 아래 내용을 살펴보았다.
    - SQL을 직접 다룰때 발생하는 문제
    - 패러다임의 불일치
    - JPA가 각 문제를 해결하는 방법 (기능적 내용)
- 이제부턴 JPA가 무엇인지 구체적으로 알아보자.

<br/>

### ORM이란

- ORM: Object-Relational Mapping
- 객체와 관계형 데이터베이스를 매핑한다는 뜻이다.
- ORM 프레임워크는 객체와 데이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다.
- ORM 프레임워크 구현체 종류
    - Hibernate (대표적)
    - EclipseLink
    - DataNucleus

<br/>

### JPA란

- JPA는 자바 진영의 ORM 기술 표준이다.
- JPA는 다음과 같이 작동한다.
    
    ![Untitled](/assets/img/2021-10-02-JPA_About_JPA/Untitled%202.png)
    

- JPA는 하이버네이트를 기반으로 만들어진 "자바 ORM 기술 표준" 이다.
    - 즉 JPA는 인터페이스 역할을 하고, 'Hibernate'나 'EclipseLink'나 'DataNucleus'가 JPA 구현체이다.
- 따라서, JPA를 사용하려면 JPA를 구현한 ORM 프레임워크 ('Hibernate', 'EclipseLink', 'DataNucleus')를 선택해야한다.
- JPA 표준은 일반적이고 공통적인 기능의 모음이다.
- **따라서, 표준을 먼저 이해하고 JPA구현체가 제공하는 고유의 기능을 알아가면 된다.**

<br/>

### JPA를 사용하는 이유

- **생산성**
    - JPA를 사용하면 자바 컬렉션에 객체를 저장하듯이 JPA에게 저장할 객체를 전달하면 된다.
    - 따라서, 상당히 편리하게 개발할 수 있다.
- **유지보수**
    - JPA를 사용하면 복잡한 과정(SQL작성, JDBC 관련 코드 등)을 JPA가 알아서 처리해준다.
    - 따라서, 유지보수해야 하는 코드의 수가 줄어든다.
- **패러다임의 불일치 해결**
    - JPA가 패러다임의 불일치 문제를 해결해준다.
- **성능**
    - JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.
- **데이터 접근 추상화와 벤더 독립성**
    - JPA는 애플리케이션과 데이버테이스 사이에 추상화된 데이터 접근 계층을 제공해서 애플리케이션이 특정 DB 기술에 종속되지 않도록 한다.
    - 예시
        - 로컬 개발 환경 = H2
        - 상용 환경 = MySQL
    
    > 벤더: MySQL, Oracle, MSSQL 등과 같은 DB 종류
    > 
- **표준**
    - 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>