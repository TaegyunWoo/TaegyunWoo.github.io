---
category: JPA
tags: [JPA]
title: "[JPA] 객체지향 쿼리 언어 - JPQL 조인"
date:   2021-11-29 17:30:00 
lastmod : 2021-11-29 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [[JPA] 객체지향 쿼리 언어 - 소개](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_Begin)
    2. [[JPA] 객체지향 쿼리 언어 - JPQL 기초](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin)

<br/><br/>

# JPQL - 조인

## 개요

- JPQL도 SQL과 마찬가지고 조인을 지원한다.
- JPQL의 조인은 SQL의 조인과 기능이 대부분 같고, 문법만 약간 다르다.

바로 JPQL 조인에 대해 알아보자.

<br/><br/>

## 내부 조인

### 내부 조인 사용 예시

```java
String teamName = "팀A";

//INNER JOIN 적용
String query = "SELECT m FROM Member m INNER JOIN m.team t "
		+ "WHERE t.name = :teamName";

List<Member> members = em.createQuery(query, Member.class)
			.setParameter("teamName", teamName)
			.getResultList();
```

> `INNER` 는 생략 가능하다.
> 
- 생성된 JPQL 쿼리
    
    ```xml
    SELECT m FROM Member m INNER JOIN m.team t
    	WHERE t.name = '팀A'
    ```
    
    - `FROM Member m`
        - 회원을 선택하고 m이라는 별칭을 주었다.
    - `Member m JOIN m.team t`
        - 회원이 가지고 있는 **연관 필드로 팀과 조인**한다.
        - 조인한 팀에는 t라는 별칭을 주었다.
- 생성된 SQL 쿼리
    
    ```sql
    SELECT
    	M.ID AS ID,
    	M.AGE AS AGE,
    	M.TEAM_ID AS TEAM_ID,
    	M.NAME AS NAME
    FROM
    	MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
    WHERE
    	T.NAME='팀A'
    
    ```
    
<br/>

### JPQL 조인의 특징

- **JPQL 조인 시, 연관 필드를 사용하여 조인한다.**
    - JPQL: `INNER JOIN m.team t`
    - SQL: `INNER JOIN TEAM T`
- 따라서 아래와 같은 JPQL 쿼리는 잘못된 쿼리이다.
    
    ```xml
    SELECT m FROM Member m JOIN Team t
    	WHERE t.name = '팀A'
    ```
    
<br/>

### 조인한 두 개의 엔티티 동시 조회

- 아래 JPQL 쿼리를 작성하여 변수 `query` 에 저장했다고 가정하자.
    
    ```xml
    SELECT m, t FROM Member m JOIN m.team t
    ```
    
- 결과 자바 코드
    
    ```java
    List<Object[]> resultList = em.createQuery(query)
    		.getResultList();
    
    for (Object[] resultRecord : resultList) {
    	Member member = (Member) resultRecord[0];
    	Team team = (Team) resultRecord[1];
    }
    ```
    
<br/><br/>

## 외부 조인

### 외부 조인 사용 예시

- 외부 조인 JPQL 쿼리
    
    ```xml
    SELECT m FROM Member m LEFT OUTER JOIN m.team t
    ```
    
    - `OUTER` 는 생략 가능하다.
- 변환된 외부 조인 SQL 쿼리
    
    ```sql
    SELECT
    	M.ID AS ID,
    	M.AGE AS AGE,
    	M.TEAM_ID AS TEAM_ID,
    	M.NAME AS NAME
    FROM
    	MEMBER M LEFT OUTER JOIN TEAM T ON M.TEAM_ID=T.ID
    
    ```
    
<br/><br/>

## 컬렉션 조인

### 컬렉션 조인이란?

- 일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 컬렉션 조인이라고 한다.
- 다대일 조인 vs 일대다 조인
    - **다대일 조인**
        - [회원 → 팀]
        - **단일 값 연관 필드 (`m.team`)을 사용하는 조인**
    - 일대다 조인
        - [팀 → 회원]
        - **컬렉션 값 연관 필드(`t.members`)를 사용하는 조인**

<br/>

### 컬렉션 조인 사용 예시

- 컬렉션 조인을 사용하는 JPQL 쿼리
    
    ```xml
    SELECT t, m FROM Team t LEFT JOIN t.members m
    ```
    
    - `t LEFT JOIN t.members`
        - '팀'과 '팀이 보유한 회원목록'을 컬렉션 값 연관 필드로 외부 조인했다.

<br/><br/>

## 세타 조인

### 세타 조인이란?

- Full Join에서 `WHERE` 절로 조건을 제시하는 조인이다.
    - Full Join
        - 각 테이블의 레코드 수를 곱한 만큼의 레코드를 출력하는 것
        - 테이블 간에 연관관계가 전혀 없어도 상관없이 조인된다.
        - 각 테이블의 레코드 간에 가능한 모든 수의 조합을 하여 결과로 반환한다.
- 세타 조인은 내부 조인만 지원한다.

<br/>

### 세타 조인 사용 예시

- 회원 이름이 팀 이름과 똑같은 사람 수를 구하는 예시 쿼리 : JPQL
    
    ```xml
    select count(m) from Member m, Team t
    	where m.username = t.name
    ```
    
- 변환된 세타 조인 SQL 쿼리
    
    ```sql
    SELECT COUNT(M.ID)
    FROM
    	MEMBER M CROSS JOIN TEAM T
    WHERE
    	M.USERNAME=T.NAME
    ```
    
<br/><br/>

## JOIN ON 절

### JOIN ON 절이란?

- JPA 2.1부터 지원하는 기능이다.
- **ON 절을 사용하면, 조인 대상을 필터링하고 조인할 수 있다.**
    - 즉 ON 절의 조건으로 '원하는 조인 대상'을 걸러내고, 그에 대한 결과 테이블로 조인시킨다.
- 보통 ON 절은 외부 조인에서만 사용한다.
    - 왜냐하면 내부 조인에서 ON 절을 사용하면, WHERE 절을 사용하는 것과 결과가 같기 때문이다.

<br/>

### JOIN ON 절 사용 예시

- 모든 회원을 조회하면서, 회원과 연관된 팀도 조회한다. 이때 팀은 이름이 'A'인 팀만 조회한다.
    - 이에 대한 JPQL은 아래와 같다.
    
    ```xml
    SELECT m, t
    FROM
    	Member m LEFT JOIN m.team t
    ON
    	t.name = 'A'
    ```
    
    - 변환된 SQL은 아래와 같다.
    
    ```xml
    SELECT m.*, t.*
    FROM
    	MEMBER m LEFT OUTER JOIN TEAM t
    ON
    	m.TEAM_ID=t.id AND t.name='A'
    ```
    
    - **SQL 결과를 보면, 조인시점에 조인 대상을 `AND t.name='A'` 로 필터링한다.**

<br/><br/>

## 페치 조인

### 페치 조인이란?

- JPQL에서 성능 최적화를 위해 제공하는 기능이다.
    - SQL에서 이야기하는 조인의 종류는 아니다.
- 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능이 바로 페치 조인이다.
- `join fetch` 명령어로 사용할 수 있다.
- 페치 조인 문법
    
    ```xml
    페치조인 ::= [LEFT [OUTER] | INNER] JOIN FETCH 조인경로
    ```
    

페치 조인에 대해 자세히 알아보자.

<br/>

### 엔티티 페치 조인

- 페치 조인을 사용해서, 회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회하는 JPQL
    
    ```xml
    select m
    from Member m join fetch m.team
    ```
    
    - 프로젝션 대상이 `m` 밖에 없다.
    - `join fetch` 명령어를 사용했다.

<br/>

- 위와 같이 JPQL 쿼리를 작성하면, **연관된 엔티티나 컬렉션을 함께 조회**한다.
    - 위 예시에선, 회원(`m`)과 팀(`m.team`)을 함께 조회한다.
- **페치 조인은 별칭을 사용할 수 없다.**
    - 하지만 하이버네이트는 페치 조인에도 별칭을 허용한다.

<br/>

- 위 JPQL 쿼리를 SQL로 변환한 결과
    
    ```sql
    SELECT
    	M.*, T.*
    FROM MEMBER M
    INNER JOIN TEAM T ON M.TEAM_ID=T.ID
    ```
    
    - JPQL에선 프로젝션 대상이 `m` 밖에 없었지만, SQL로 변환하니 `MEMBER` 와 `TEAM` 모두 프로젝션 대상이 되었다.

- 시각화
    - 엔티티 페치 조인 시도 시
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%206.png)
        
    - 엔티티 페치 조인 결과 테이블
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%207.png)
        
    - 엔티티 페치 조인 결과 객체
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%208.png)
        
        - **회원과 팀 객체가 객체 그래프를 유지하면서 조회되었다.**

- 상세 설명
    - 엔티티 페치 조인 JPQL에서 `select m` 으로 회원 엔티티만 선택했는데, 실행된 SQL을 보면 `SELECT M.*, T.*` 로 회원과 연관된 팀도 함께 조회된 것을 확인할 수 있다.
        - 즉 엔티티 페치 조인시, 연관 엔티티까지 가져온다.

<br/>

- 엔티티 페치 조인 사용 예시
    - 위에서 작성한 페치 조인 JPQL 쿼리를 사용해보자.
    
    ```java
    String jpql = "select m from Member m join fetch m.team";
    List<Member> members = em.createQuery(jpql, Member.class)
    			.getResultList();
    
    for (Member member : members) {
    	//페치 조인으로 회원과 팀을 함께 조회했다.
    	//따라서 지연 로딩이 발생하지 않는다.
    	System.out.println(member.getUsername() + "," + member.getTeam().getName());
    }
    ```
    
    - 출력 결과는 아래와 같다.
        
        ```xml
        회원1,팀A
        회원2,팀A
        회원3,팀B
        ```
        

<br/>

- **엔티티 페치 조인과 지연로딩**
    - 만약 회원과 팀을 지연로딩으로 설정했다고 가정해보자.
        - 페치 조인을 사용했기 때문에, 회원을 조회할 때 팀도 함께 조회했다.
        - 따라서, **연관된 팀 엔티티는 프록시가 아닌 실제 엔티티**다.
        - **즉 연관됨 팀을 사용해도 지연 로딩이 일어나지 않는다.**

<br/>

### 컬렉션 페치 조인

- 이번에는 일대다 관계인 컬렉션을 페치 조인해보자.
- 아래는 컬렉션 페치 조인 JPQL 쿼리이다.
    
    ```xml
    select t
    from Team t join fetch t.members
    where t.name = '팀A'
    ```
    
    - 페치 조인을 사용했다.
    - 따라서 팀(`t`)를 조회하면서 연관된 회원 컬렉션(`t.members`)도 함께 조회한다.
- 위 JPQL을 변환하여 실행된 SQL은 아래와 같다.
    
    ```sql
    SELECT T.*, M.*
    FROM TEAM T INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
    WHERE T.NAME='팀A'
    ```
    

<br/>

- 시각화를 통해, 위 예시를 좀 더 자세히 알아보자.
    - 컬렉션 페치 조인 시도
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%209.png)
        
        - **`MEMBER` 테이블과 조인하면서 결과가 증가한다.**
        - **따라서 아래 조인 결과 테이블을 보면 같은 '팀A'가 2건 조회된다.**
    - 컬렉션 페치 조인 결과 테이블
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%2010.png)
        
        - 같은 레코드 `팀A` 가 2개 존재한다.
    - 컬렉션 페치 조인 결과 객체
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%2011.png)
        
<br/>

- 상세 설명
    - JPQL에서 `select t` 로 팀만 선택했는데, SQL을 보면 `T.*, M.*` 로 팀과 연관된 회원도 함께 조회한 것을 확인할 수 있다.
    - **`TEAM` 테이블과 `MEMBER` 테이블이 조인하면서 결과가 증가했다.**
        - **그로 인해, 조인 결과 테이블에 같은 '팀A'가 2건 조회되었다.**
        - **따라서 주소가 `0x100`으로 같은 '팀A'를 2건 가지게 된다.**
- 예시 코드
    
    ```java
    String jpql = "select t from Team t join fetch t.members where t.name = '팀A'";
    
    //아래 코드 실행 시, teams 리스트에 같은 객체(팀A)가 2개 들어간다.
    List<Team> teams = em.createQuery(jpql, Team.class).getResultList();
    
    for (Team team : teams) {
    	System.out.println(team.getName() + "," + team);
    
    	for (Member member : team) {
    		System.out.println("->" + member.getUsername() + "," + member);
    	}
    
    	System.out.println();
    }
    ```
    
    - 출력 결과는 아래와 같다.
        
        ```xml
        팀A,Team@0x100
        ->회원1,Member@0x200
        ->회원2,Member@0x300
        
        팀A,Team@0x100
        ->회원1,Member@0x200
        ->회원2,Member@0x300
        ```
        
        - 같은 '팀A'가 2건 조회된 것을 확인할 수 있다.
        - 그리고 연관된 엔티티 역시 동일하다.

<br/>

### 페치 조인과 DISTINCT

- `DISTINCT` 란?
    - SQL에서의 `DISTINCT`
        - 중복된 결과를 제거하는 명령어
    - JPQL에서의 `DISTINCT`
        - SQL에 `DISTINCT` 를 추가한다.
        - **또한 애플리케이션에서 한 번 더 중복을 제거한다.**

<br/>

- 바로 위에서 살펴본 컬렉션 페치 조인은 '팀A'가 중복으로 조회된다. **`DISTINCT` 를 사용해서 이러한 문제를 해결**할 수 있다.
    - 예시 JPQL 쿼리
        
        ```xml
        select distinct t
        from Team t join fetch t.members
        where t.name = '팀A'
        ```
        
        - SQL에 `DISTINCT` 가 추가된다.
        - 하지만 결과 테이블의 레코드 데이터가 다르므로, 'SQL의 `DISTINCT`'로서의 효과는 없다.
            
            ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%2012.png)
            
        - **다음으로 애플리케이션에서 `DISTINCT` 명령어를 보고 중복된 데이터를 걸러낸다.**
            
            ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%2013.png)
            
        - 이제 다시 결과를 출력해보자.
            
            ```xml
            팀A,Team@0x100
            ->회원1,Member@0x200
            ->회원2,Member@0x300
            ```
            
            - '팀A' 객체의 중복이 사라졌다.

<br/>

### 페치 조인과 일반 조인의 차이

- 일반 조인 예시
    - JPQL
        
        ```xml
        select t
        from Team t join t.members m
        where t.name = '팀A'
        ```
        
    - SQL
        
        ```sql
        SELECT T.*
        FROM TEAM T JOIN MEMBER M ON T.ID=M.TEAM_ID
        WHERE T.NAME='팀A'
        ```
        

- 일반 조인 시
    - **JPQL은 결과를 반환할 때, 연관관계까지 고려하지 않는다.**
    - **단지 SELECT 절에 지정한 엔티티 (프로젝션)만 조회할 뿐이다.**
    - 따라서 **"회원 컬렉션을 지연 로딩으로 설정"하면 아래 그림과 같이, 프록시나 아직 초기화하지 않은 컬렉션 래퍼를 반환**한다.
        
        ![Untitled](/assets/img/2021-11-29-JPA_ObjectQuery_JPQL_Join/Untitled%2014.png)
        
    - 또는 **"회원 컬렉션을 즉시 로딩으로 설정"하면, 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한번 더 실행한다.**

<br/>

### 페치 조인의 특징과 한계

- 페치 조인의 특징
    - **페치 조인을 사용하면, SQL 한번으로 연관된 엔티티들을 함께 조회할 수 있다.**
        - 따라서 SQL 호출 횟수를 줄여 성능을 최적화할 수 있다.
    - **페치 조인은 글로벌 로딩 전략보다 우선한다.**
        - 즉 글로벌 로딩 전략을 지연 로딩으로 설정해도, JPQL에서 페치 조인을 사용하면 페치 조인을 적요해서 함께 조회한다.
    - **연관된 엔티티를 쿼리 시점에 조회하므로 지연 로딩이 발생하지 않는다.**
        - **따라서 준영속 상태에서도 객체 그래프를 탐색할 수 있다.**
        
        > 이미 연관된 엔티티 간의 객체 그래프가 생성되어 있기 때문에
        > 

<br/>

- 효과적인 설정방법
    - 글로벌 로딩 전략 == 지연 로딩
    - 최적화가 필요한 경우 == 페치 조인 적용

<br/>

- 페치 조인의 한계
    - **페치 조인 대상에는 별칭을 줄 수 없다.**
        - 하이버네이트는 페치 조인에 별칭을 지원하긴 하지만, 특별히 조심해서 사용해야한다.
        
        > 자세한 것은 추후 포스팅으로 다룰 예정이다.
        > 
    - **둘 이상의 컬렉션을 페치할 수 없다.**
    - **컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.**
        - 왜냐하면 페치 조인과 함께 페이징 API를 사용하면, 메모리에서 페이징 처리를 하기 때문에 Overflow가 발생할 수 있다.

<br/>

### 참고

- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다.
- 반면에 여러 테이블을 조인해서, 엔티티가 가진 모양이 아닌 다른 결과를 내야 한다면 DTO로 반환하는 것이 더 효과적일 수 있다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>