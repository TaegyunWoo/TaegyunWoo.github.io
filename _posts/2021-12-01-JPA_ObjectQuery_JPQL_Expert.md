---
category: JPA
tags: [JPA]
title: "[JPA] 객체지향 쿼리 언어 - JPQL 심화"
date:   2021-12-01 14:30:00 
lastmod : 2021-12-01 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [[JPA] 객체지향 쿼리 언어 - 소개](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_Begin)
    2. [[JPA] 객체지향 쿼리 언어 - JPQL 기초](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin)
    3. [[JPA] 객체지향 쿼리 언어 - JPQL 조인](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Join)

<br/><br/>

# JPQL 심화

## 경로 표현식

### 경로 표현식이란?

- 쉽게 말하자면, .(점)을 찍어 객체 그래프를 탐색하는 것을 말한다.
    - 아래 JPQL 쿼리를 보자.
        
        ```xml
        select m.username
        from Member m
        join m.team t
        join m.orders o
        where t.name = '팀A'
        ```
        
        - `m.username` , `m.team` , `m.orders` , `t.name`
        - 위에 서술한 것들이 바로 경로 표현식을 사용한 것들이다.

<br/>

### 경로 표현식 용어 정리

경로 표현식을 이해하기 위해, 필요한 용어들을 설명하겠다.

- **상태 필드**
    - 단순히 값을 저장하기 위한 필드
    - 필드 or 프로퍼티
- **연관 필드**
    - 객체 사이의 연관관계를 맺기 위해 사용하는 필드
    - 임베디드 타입 포함
    - 필드 or 프로퍼티
    - 연관 필드의 종류
        - **단일 값 연관 필드**
            - `@ManyToOne`
            - `@OneToOne`
            - 대상이 엔티티이다.
        - **컬렉션 값 연관 필드**
            - `@OneToMany`
            - `@ManyToMany`
            - 대상이 컬렉션이다.

<br/>

- 예제 코드
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	@Column(name = "name")
    	private String name; //상태 필드
    	private Integer age; //상태 필드
    
    	@ManyToOne(..)
    	private Team team; //연관 필드 (단일값 연관 필드)
    
    	@OneToMany(..)
    	private List<Order> orders; //연관 필드 (컬렉션 값 연관 필드)
    
    }
    ```
    
    - 상태 필드: `m.username` , `m.age`
    - 단일 값 연관 필드: `m.team`
    - 컬렉션 값 연관 필드: `m.orders`

<br/>

### 경로 표현식과 특징

경로 표현식의 경로 탐색 범위는 아래와 같다.

- **상태 필드 경로**
    - 경로 탐색의 끝
    - 더는 탐색할 수 없다.
- **단일 값 연관 경로**
    - 묵시적으로 내부 조인이 일어난다.
        
        > 묵시적 내부 조인에 대해선 본 포스팅에서 자세히 다룬다.
        > 
    - 계속 탐색할 수 있다.
- **컬렉션 값 연관 경로**
    - 묵시적으로 내부 조인이 일어난다.
    - 더는 탐색할 수 없다.
    - 단 FROM 절에서 조인을 통해 별칭을 얻으면, 별칭으로 탐색할 수 있다.

<br/>

예시를 통해 각각 알아보자.

- **상태 필드 경로 탐색**
    - JPQL 쿼리
        
        ```xml
        select m.username, m.age from Member m
        ```
        
        - 상태 필드 경로 탐색: `m.username` , `m.age`
    - 결과 SQL
        
        ```sql
        SELECT m.name, m.age FROM MEMBER m
        ```
        

<br/>

- **단일 값 연관 경로 탐색 - 1**
    - JPQL 쿼리
        
        ```xml
        select o.member from Order o
        ```
        
        - 단일 값 연관 경로 탐색: `m.member`
    - 결과 SQL
        
        ```sql
        SELECT m.* FROM Orders o
        INNER JOIN MEMBER m ON o.MEMBER_ID=m.ID
        ```
        
        - **단일 값 연관 필드로 경로 탐색을 하면, SQL에서 내부 조인이 일어난다. 이것을 '묵시적 조인'이라고 한다.**
        - **묵시적 조인은 모두 내부 조인이다.**

<br/>

- **단일 값 연관 경로 탐색 - 2**
    - JPQL 쿼리
        
        ```xml
        select o.member.team from Order o
        where o.product.name = 'productA'
        and o.address.city = 'Incheon'
        ```
        
        - 단일 값 연관 경로 탐색: `o.member.team` , `o.product.name` , `o.address.city`
    - 결과 SQL
        
        ```sql
        SELECT t.* FROM Orders o
        INNER JOIN MEMBER m ON o.MEMBER_ID=m.ID
        INNER JOIN TEAM t ON o.TEAM_ID=t.ID
        INNER JOIN PRODUCT p ON o.PRODUCT_ID=P.ID
        WHERE p.NAME='productA' AND o.CITY='Incheon'
        ```
        
        - `o.address.city` 처럼 임베디드 타입에 접근하는 것 역시 단일 값 연관 경로 탐색이다.
        - **하지만 `city` 가 하나의 테이블(`Order`)에 존재하므로 조인할 필요가 없다.**

<br/>

- **컬렉션 값 연관 경로 탐색**
    - JPQL 쿼리
        
        ```xml
        select t.members from Team t //성공
        select t.members.username from Team t //불가능
        ```
        
        - 컬렉션 값 연관 경로 탐색: `t.members`
        - `select t.members.username from Team t`
            - members의 각 member의 name이 리스트로 반환되지 않을까?
            - **불가능하다!**
    - 상세 설명
        - `t.members` 처럼 컬렉션까지는 경로 탐색이 가능하다.
        - `**t.members.username` 처럼 컬렉션에서 경로 탐색을 시작하는 것은 허락하지 않는다.**
        - **만약 컬렉션에서 경로 탐색을 하고 싶으면, 아래 코드처럼 조인을 사용해서 새로운 별칭을 획득해야 한다.**
            
            ```xml
            //별칭 m을 획득하여 경로를 탐색할 수 있다.
            //이 경우, m.username은 상태 필드 경로 탐색이다.
            select m.username from Team t join t.members m
            ```
            

> 참고: `size` 라는 것을 통해, 컬렉션의 크기를 구할 수 있다.  
예시: `select t.members.size from Team t`
> 

<br/>

### 경로 탐색을 사용한 묵시적 조인 시 주의사항

- **묵시적 조인은 항상 내부 조인이다.**
- 컬렉션은 경로 탐색의 끝이다.
    - 컬렉션에서 경로 탐색을 하려면 명시적으로 조인해서 별칭을 얻어야 한다.
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만, 묵시적 조인으로 인해 SQL의 FROM절에 영향을 준다.

<br/>

- **묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.**
- **따라서 성능이 중요하면 분석하기 쉽도록 명시적 조인을 사용하자.**

<br/><br/>

## 서브 쿼리

### JPQL 서브쿼리 제약

JPQL도 SQL처럼 서브 쿼리를 지원한다. 단 몇 가지 제약이 있다.

- 서브쿼리를 WHERE, HAVING 절에서만 사용할 수 있다.
- 서브쿼리를 SELECT, FROM 절에서는 사용할 수 없다.

<br/>

### JPQL 서브쿼리 사용 예시

- 나이가 평균보다 많은 회원 찾기
    
    ```xml
    select m from Member m
    where m.age > (select AVG(m2.age) from Member m2)
    ```
    
- 한 건이라도 주문한 고객 찾기
    
    ```xml
    select m from Member m
    where 0 < (select COUNT(o) from Order o where o.member=m)
    ```
    
    > 예시 도메인 모델은 [이전 게시글을 참고](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin#3)하자.
    > 

<br/>

### 서브 쿼리 함수

- `EXISTS` 함수
    - 문법
        - `[NOT] EXISTS (서브쿼리)`
    - 설명
        - 서브쿼리에 결과가 존재하면 참이다.
        - NOT은 반대이다.
    - 예시 : 팀A 소속인 회원 조회
        
        ```xml
        select m from Member m
        where EXISTS (select t from m.team where t.name='팀A')
        ```
        

<br/>

- `{ALL | ANY | SOME}` 함수
    - 문법
        - `{ALL | ANY | SOME} (서브쿼리)`
    - 설명
        - 비교 연산자와 같이 사용한다.
            - ALL : 조건을 모든 서브쿼리 결과 레코드가 만족하면 참이다.
            - ANY or SOME : 둘다 같은 의미다. 조건을 하나의 결과 레코드가 만족하면 참이다.
    - 예시 : 전체 상품 각각의 재고보다 주문량이 많은 주문들
        
        ```xml
        select o from Order o
        where o.orderAmount > ALL (select p.stockAmount from Product p)
        ```
        
    - 예시 : 어떤 팀이든 팀에 소속된 회원
        
        ```xml
        select m from Member m
        where m.team = ANY (select t from Team t)
        ```
        

<br/>

- `IN` 함수
    - 문법
        - `[NOT] IN (서브쿼리)`
    - 설명
        - 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참이다.
        - 참고로 `IN`은 서브쿼리가 아닌 곳에서도 사용한다.
    - 예시 : 20세 이상을 보유한 팀
        
        ```xml
        select t from Team t
        where t IN (select t2 from Team t2 join t2.members m2 where m2.age >= 20)
        ```
        
<br/>

## 조건식

### 타입 표현

JPQL에서 사용하는 타입은 아래와 같다. (대소문자는 구분하지 않는다.)

|종류|설명|예제|
|----|----|----|
|문자|- 작은 따옴표 사이에 표현 <br/> - 작은 따옴표를 표현하고 싶으면 작은 따옴표 연속 두 개('') 사용|'HELLO' <br/> 'It''s'|
|숫자|- L (Long 타입 지정) <br/> - D (Double 타입 지정) <br/> - F (Float 타입 지정)|10L <br/> 10D  <br/>10F|
|날짜|- DATE {d 'yyyy-mm-dd'} <br/> - TIME {t 'hh-mm-ss'} <br/> - DATETIME {ts 'yyyy-mm-dd hh:mm:ss.f'}|{d '2021-11-29'} <br/> {t '10-11-11'} <br/> {ts '2021-11-29 10-11-11.123'} <br/> m.createDate = {d '2012-03-24'}|
|Boolean|TRUE <br/> FALSE||
|Enum|패키지명을 포함한 전체 이름을 사용해야 한다.|jpabook.MemberType.Admin|
|엔티티타입|엔티티의 타입을 표현한다. <br/> 주로 상속과 관련해서 사용한다.|TYPE(m)=Member|

<br/>

### 연산자 우선 순위

연산자 우선 순위는 아래와 같다.

1. 경로 탐색 연산
    - . (점)
2. 수학 연산
    - +
    - -(단항 연산자)
    - *
    - /
3. 비교 연산
    - =
    - \>
    - \>=
    - <
    - <=
    - <>(다름)
    - [NOT] BETWEEN
    - [NOT] LIKE
    - [NOT] IN
    - IS [NOT] NULL
    - IS [NOT] EMPTY
    - [NOT] MEMBER [OF]
    - [NOT] EXISTS
4. 논리 연산
    - NOT
    - AND
    - OR

<br/>

### 논리 연산과 비교식

- 논리 연산
    - AND : 둘다 만족하면 참
    - OR : 둘 중 하나만 만족해도 참
    - NOT : 조건식의 결과 반대
- 비교식
    - =
    - \>
    - \>=
    - <
    - <=
    - <>

<br/>

### Between, IN, Like, NULL 비교

- **Between 식**
    - 문법
        - `X [NOT] BETWEEN A AND B`
    - 설명
        - X는 A~B 사이의 값이면 참이다.
        - A와 B를 포함한다.
    - 예시 : 나이가 10~20인 회원 찾기
        
        ```xml
        select m from Member m
        where m.age BETWEEN 10 AND 20
        ```
        

<br/>

- **IN 식**
    - 문법
        - `X [NOT] IN (예제)`
    - 설명
        - X와 같은 값이 예제에 하나라도 있으면 참이다.
        - 예제에는 서브쿼리를 사용할수도 있다.
    - 예시 : 이름이 회원1이거나 회원2인 회원 찾기
        
        ```xml
        select m from Member m
        where m.username IN ('회원1', '회원2')
        ```
        

<br/>

- **LIKE 식**
    - 문법
        - `문자표현식 [NOT] LIKE 패턴값 [ESCAPE 이스케이프문자]`
    - 설명
        - 문자표현식과 패턴값을 비교한다.
        - `%` : 아무 값들이입력되어도 된다. (값이 없어도 됨)
        - `_` : 한 글자는 아무 값이 입력되어도 된다. (값이 반드시 있어야 함)
    - 예시
        
        ```xml
        //중간에 원이라는 단어가 들어간 회원
        select m from Member m
        where m.username LIKE '%원%'
        
        //처음에 회원이라는 단어가 들어간 회원
        select m from Member m
        where m.username LIKE '회원%'
        
        //마지막에 회원이라는 단어가 들어간 회원
        select m from Member m
        where m.username LIKE '%회원'
        
        //회원A, 회원1
        select m from Member m
        where m.username LIKE '회원_'
        
        //회원3
        select m from Member m
        where m.username LIKE '__3'
        
        //회원%
        select m from Member m
        where m.username LIKE '원\%' ESCAPE '\'
        ```
        

<br/>

- **NULL 비교식**
    - 문법
        - `{단일값 경로 | 입력 파라미터} IS [NOT] NULL`
    - 설명
        - NULL인지 비교한다.
        - NULL은 `=` 으로 비교하면 안되고, 반드 `IS NULL` 을 사용해야 한다.
    - 예시
        
        ```xml
        select m from Member m
        where m.username IS NULL
        ```
        

<br/>

- **컬렉션 식**
    - 컬렉션 식이란?
        - 컬렉션에만 사용하는 특별한 기능이다.
        - 컬렉션은 컬렉션 식 이외에 다른 식은 사용할 수 없다.
    - **빈 컬렉션 비교식: 문법**
        - `{컬렉션 값 연관 경로} IS [NOT] EMPTY`
    - **빈 컬렉션 비교식: 설명**
        - 컬렉션에 값이 비었으면 참이다.
    - **빈 컬렉션 비교식: 예시**
        
        ```xml
        //JPQL: 주문이 하나라도 있는 회원 조회
        select m from Member m
        where m.orders IS EMPTY
        
        //실행된 SQL
        SELECT m.* from MEMBER m
        WHERE EXISTS (
        	SELECT o.ID FROM ORDERS o WHERE m.ID=o.MEMBER_ID
        )
        
        //오류가 발생하는 JPQL
        select m from Member m
        where m.orders IS NULL
        ```
        
    
    - **컬렉션의 멤버식: 문법**
        - `{엔티티or값} [NOT] MEMBER [OF] {컬렉션 값 연관 경로}`
    - **컬렉션의 멤버식: 설명**
        - 엔티티나 값이 컬렉션에 포함되어 있으면 참이다.
    - **컬렉션의 멤버식: 예시**
        
        ```xml
        //JPQL: 주문이 하나라도 있는 회원 조회
        select t from Team t
        where :memberParam MEMBER OF t.members
        ```
        
<br/>

### 스칼라 식

스칼라는 숫자, 문자, 날짜, case, 엔티티 타입 같은 가장 기본적인 타입들을 말한다.

- 수학 식
    - +, - : 단항 연산자
    - *, /, +, - : 사칙연산

<br/>

- 문자 함수

|함수|설명|예제|
|----|----|----|
|CONCAT(문자1, 문자2, ...)|문자를 합한다.|CONCAT('A', 'B') = AB|
|SUBSTRING(문자, 위치, [길이])|- 위치부터 시작해 길이만큼 문자를 구한다. <br/> - 길이 값이 없으면 나머지 전체 길이를 뜻한다.| SUBSTRING('ABCDEF', 2, 3) = BCD|
|TRIM([[LEADING | TRAILING | BOTH] [트림문자] FROM] 문자)|- LEADING: 왼쪽만 트림문자 제거<br/> - TRAILING: 오른쪽만 트림문자 제거 <br/> - BOTH: 양쪽 다 트림문자 제거 <br/> - 기본값은 BOTH <br/> - 트림문자의 기본값은 공백(SPACE)이다.|TRIM('ABC')='ABC'|
|LOWER(문자)|소문자로 변경|LOWER('ABC')='abc'|
|UPPER(문자)|대문자로 변경|UPPER('abc')='ABC'|
|LENGTH(문자)|문자 길이|LENGTH('ABC')=3|
|LOCATE(찾을 문자, 원본 문자, [검색시작위치])|- 검색위치부터 문자를 검색한다. <br/> - 1부터 시작, 못찾으면 0 반환|LOCATE('DE', 'ABCDEFG')=4|

<br/>

- 수학 함수

|함수|설명|예제|
|----|----|----|
|ABS(수학식)|절대값을 구한다.|ABS(-10)=10|
|SQRT(수학식)|제곱근을 구한다.|SQRT(4)=2.0|
|MOD(수학식, 나눌수)|나머지를 구한다.|MOD(4, 3)=1|
|SIZE(컬렉션 값 연관 경로식)|컬렉션의 크기를 구한다.|SIZE(t.members)|
|INDEX(별칭)|- LIST 타입 컬렉션의 위치값을 구한다. <br/> - 단 컬렉션이 @OrderColumn을 사용하는 LIST 타입일 때만 사용할 수 있다.|t.members m where INDEX(m) > 3|

<br/>

- 날짜 함수
    - `CURRENT_DATE` : 현재 날짜
    - `CURRENT_TIME` : 현재 시간
    - `CURRENT_TIMESTAMP` : 현재 날짜 시간
    - 예시 : 종료 이벤트 조회
        
        ```xml
        select e from Event e where e.endDate < CURRENT_DATE
        ```
        
    - 하이버네이트는 날짜 타입에서 년, 월, 일, 시간, 분, 초 값을 구하는 기능을 지원한다.
        - `YEAR` , `MONTH` , `DAY` , `HOUR` , `MINUTE` , `SECOND`
        - 예시
            
            ```xml
            select year(CURRENT_TIMESTAMP), match(CURRENT_TIMESTAMP),
            	day(CURRENT_TIMESTAMP)
            from Member
            ```
            
<br/>

### CASE 식

- CASE 식이란?
    - 특정 조건에 따라 분기할 때 사용한다.
- CASE 식의 종류
    - **기본 CASE**
    - **심플 CASE**
    - **COALESCE**
    - **NULLIF**

하나씩 순서대로 알아보자.

<br/>

- **기본 CASE**
    - 문법
        
        ```xml
        CASE
        	{WHEN {조건식} THEN {스칼라식}}+
        	ELSE {스칼라식}
        END
        ```
        
    - 예시
        
        ```xml
        select case
        	when m.age <= 10 then '학생요금'
        	when m.age >= 60 then '경로요금'
        	else '일반요금'
        end from Member m
        ```

<br/>

- **심플 CASE**
    - 문법
        
        ```xml
        CASE {조건대상}
        	{WHEN {스칼라식1} THEN {스칼라식2}}+
        	ELSE [스칼라식]
        END
        ```
        
    - 설명
        - 자바의 switch 문과 유사하다.
    - 예시
        
        ```xml
        select case t.name
        	when '팀A' then '인센티브110%'
        	when '팀B' then '인센티브120%'
        	else '인센티브105%'
        end from Team t
        ```
        
<br/>

- **COALESCE**
    - 문법
        
        ```xml
        COALESCE({스칼라식} {, {스칼라식}}+)
        ```
        
    - 설명
        - 스칼라식을 차례대로 조회해서 NULL이 아니면 반환한다.
    - 예시 : m.username 이 null이면 '이름없는 회원'을 반환하라
        
        ```xml
        select coalesce(m.username, '이름없는 회원') from Member m
        ```
        
<br/>

- **NULLIF**
    - 문법
        
        ```xml
        NULLIF({스칼라식}, {스칼라식})
        ```
        
    - 설명
        - 두 값이 같으면 NULL을 반환한다.
        - 두 값이 다르면 첫 번째 값을 반환한다.
    - 예시 : 사용자 이름이 '관리자'면 NULL을 반환하고 나머지는 본인의 이름을 반환하라
        
        ```xml
        select nullif(m.username, '관리자') from Member m
        ```
        
<br/><br/>

## 다형성 쿼리

### 다형성 쿼리란?

- JPQL로 부모 엔티티를 조회했을 때, 그 자식 엔티티도 함께 조회한다는 것이다. 아래 예시를 보자.
- 예시
    - `Item` 의 자식으로 `Book` , `Album` , `Movie` 가 존재한다고 하자.
    
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    	//...
    }
    
    @Entity
    @DiscriminatorValue("B")
    public class Book extends Item {
    	//...
    	private String author;
    }
    
    //Album, Movie 생략
    ```
    
    - 이때 아래와 같이 조회하면 `Item`의 자식도 함께 조회된다.
        
        ```java
        List resultList = em.createQuery("select i from Item i").getResultList();
        ```
        
    - 결과 SQL: 단일테이블 전략
        
        ```sql
        SELECT * FROM ITEM
        ```
        
        - 단일 테이블이기 때문에 ITEM에서만 조회해도 자식까지 조회된다.
    - 결과 SQL: 조인 전략
        
        ```sql
        SELECT i.*, b.*, a.*, m.*
        FROM ITEM i
        LEFT OUTER JOIN BOOK b ON i.ITEM_ID = b.ITEM_ID
        LEFT OUTER JOIN ALBUM a ON i.ITEM_ID = a.ITEM_ID
        LEFT OUTER JOIN MOVIE m ON i.ITEM_ID = m.ITEM_ID
        ```
        
        - 외부 조인을 통해, 자식까지 모두 조회한다.

<br/>

### TYPE

- TYPE이란, 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 주로 사용하는 것이다.
- 예시 : Item 중에 Book, Movie를 조회하라.
    - JPQL 쿼리
        
        ```xml
        select i from Item i
        where i.DTYPE in ('B', 'M')
        ```
        
    - 결과 SQL
        
        ```sql
        SELECT i FROM Item i
        WHERE i.DTYPE IN ('B', 'M')
        ```
        
<br/>

### TREAT

- TREAT란?
    - JPA 2.1에 추가된 기능이다.
    - 자바의 타입 캐스팅과 비슷하다.
    - 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
    - JPA 표준: FROM, WHERE 절에서만 사용할 수 있다.
    - Hibernate: SELECT 절에서도 사용할 수 있다.
- 예시
    - JPQL 쿼리
        
        ```xml
        select i from Item i
        where treat(i as Book).author = 'kim'
        ```
        
        - treat를 통해 Item을 자식 타입인 Book으로 다룬다. 따라서 `author` 필드에 접근할 수 있다.
    - 결과 SQL
        
        ```sql
        SELECT i FROM Item i
        WHERE i.DTYPE='B' AND i.AUTHOR='kim'
        ```
        
<br/><br/>

## 사용자 정의 함수 호출

### 문법

`function_invocation::= FUNCTION(function_name {, function_arg}*)`

- 예시
    
    ```xml
    select function('group_concat', i.name) from Item i
    ```
    
<br/>

### 조건

- 하이버네이트 구현체를 사용하면, 아래와 같이 방언 클래스를 상속해서 구현하고 사용할 DB 함수를 미리 등록해야 한다.

```java
public class MyH2Dialect extends H2Dialect {

	public MyH2Dialect() {
		registerFunction("group_concat", new StandardSQLFunction(
				"group_concat", StandardBasicTypes.STRING
		));
	}

}
```

- 그리고 하애와 같이 `hibernate.dialect` 에 해당 방언을 등록해야 한다.
    - `persistence.xml`
        
        ```xml
        <property name="hibernate.dialect" value="hello.MyH2Dialect"/>
        ```
        
<br/><br/>

## 기타 정리

### 기타

- `enum` 은 `=` 비교 연산만 지원한다.
- 임베디드 타입은 비교를 지원하지 않는다.

<br/>

### EMPTY STRING

- JPA 표준은 ''을 길이 0인 Empty String으로 정했다.
- 하지만 DB에 따라 ''을 NULL로 사용하는 경우도 있다.
- 따라서 Empty String을 사용하기 전에 미리 확인해야한다.

<br/>

### NULL 정의

- 조건을 만족하는 데이터가 하나도 없으면 NULL이다.
- NULL은 알 수 없는 값이다. NULL과의 모든 수학적 계산 결과는 NULL이다.
- `NULL == NULL` 은 알 수 없는 값이다.
- `NULL is NULL` 은 참이다.

<br/>

- **AND 연산과 NULL**

|AND|TRUE|FALSE|NULL|
|----|----|----|----|
|TRUE|TRUE|FALSE|NULL|
|FALSE|FALSE|FALSE|FALSE|
|NULL|NULL|FALSE|NULL|

<br/>

- **OR 연산과 NULL**
    
|OR|TRUE|FALSE|NULL|
|----|----|----|----|
|TRUE|TRUE|TRUE|TRUE|
|FALSE|TRUE|FALSE|NULL|
|NULL|TRUE|NULL|NULL|

<br/>

- **NOT 연산과 NULL**
    
|NOT||
|----|----|
|TRUE|FALSE|
|FALSE|TRUE|
|NULL|NULL|

<br/><br/>

## 엔티티 직접 사용

### 기본키 값

- 객체 인스턴스는 참조 값으로 식별하고, 테이블 로우(레코드)는 기본키 값으로 식별한다.
- **따라서 JPQL에서 엔티티 객체를 직접 사용하면, SQL에서는 해당 엔티티의 기본키 값을 사용한다.**
    - JPQL에서의 엔티티 객체 직접 사용 예시
        
        ```xml
        select count(m) from Member m //엔티티 직접 사용
        select count(m.id) from Member m //엔티티의 아이디 사용
        ```
        
- **엔티티를 직접 사용하면 JPQL이 SQL로 변환될 때, 해당 엔티티의 기본키를 사용한다. 따라서 실제 실행된 SQL은 둘 다 같다.**
    - SQL 결과
        
        ```sql
        SELECT COUNT(m.id) AS cnt
        FROM MEMBER m
        ```
        
<br/>

- 파라미터: 엔티티 직접 사용 vs 엔티티 기본키 사용
    - **엔티티를 파라미터로 직접 받는 코드**
        
        ```java
        String sqlString = "select m from Member m where m = :member";
        List resultList = em.createQuery(sqlString)
        		.setParameter("member", member)
        		.getResultList();
        ```
        
        - SQL 결과
            
            ```sql
            SELECT m.*
            FROM MEMBER m
            WHERE m.ID=?
            ```
            
    - **기본키를 파라미터로 받는 코드**
        
        ```java
        String sqlString = "select m from Member m where m.id = :memberId";
        List resultList = em.createQuery(sqlString)
        		.setParameter("memberId", 4L)
        		.getResultList();
        ```
        
        - SQL 결과
            
            ```sql
            SELECT m.*
            FROM MEMBER m
            WHERE m.ID=?
            ```
            
    - **'엔티티를 직접 파라미터에 전달하는 것'이나 '기본키를 파라미터에 전달하는 것'이나 결과는 동일하다.**

<br/>

### 외래키 값

- **외래키 대신에 엔티티를 직접 사용하는 코드**
    
    ```java
    Team team = em.find(Team.class, 1L);
    
    String sqlString = "select m from Member m where m.team = :team";
    
    List resultList = em.createQuery(sqlString)
    		.setParameter("team", team)
    		.getResultList();
    ```
    
    - SQL 결과
        
        ```sql
        SELECT m.*
        FROM MEMBER m
        WHERE m.TEAM_ID=?
        ```
     
- **외래키를 사용하는 코드**
    
    ```java
    Team team = em.find(Team.class, 1L);
    
    String sqlString = "select m from Member m where m.team.id = :team";
    
    List resultList = em.createQuery(sqlString)
    		.setParameter("team", team)
    		.getResultList();
    ```
    
    - SQL 결과
        
        ```sql
        SELECT m.*
        FROM MEMBER m
        WHERE m.TEAM_ID=?
        ```
        
- **이것 역시 결과가 동일하다.**
- **묵시적 조인이 일어나지 않는 이유**
    - `m.team.id` 를 보면, 묵시적 조인이 일어나야 할 것 같다.
    - **하지만 `Team`의 id값은 `Member` 테이블에 존재하므로 (외래키) 묵시적 조인이 일어나지 않는다.**

<br/><br/>

## Named 쿼리: 정적 쿼리

### JPQL 쿼리 종류

- **동적 쿼리**
    - JPQL을 문자로 완성해서 직접 넘기는 것
    - 예시) `select m from Member m`
- **정적 쿼리**
    - 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있는 기능
    - 이것을 **Named 쿼리**라고 한다.
    - Named 쿼리는 한번 정의하면 변경할 수 없다.

<br/>

### 정적 쿼리의 특징

- Named 쿼리는 **애플리케이션 로딩 시점에 JPQL 문법을 체크**하고 파싱해둔다.
    - 따라서 오류를 빨리 확인할 수 있다.
    - 또한 성능상 이점도 있다.
    - 데이터베이스의 조회 성능 최적화

<br/>

### Named 쿼리 작성 방법: 애너테이션에 정의하기

- Named 쿼리 작성
    
    ```java
    @Entity
    @NamedQuery(
    	name = "Member.findByUsername",
    	query = "select m from Member m where m.username = :username"
    )
    public class Member {
    	//...
    }
    ```
    
    - `@NamedQuery` 애너테이션을 통해, Named 쿼리를 작성한다.
        - `name` 속성: 쿼리 이름 부여
        - `query` 속성: 사용할 쿼리 작성

- Named 쿼리 사용
    
    ```java
    List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
    		.setParameter("username", "회원1")
    		.getResultList();
    ```
    
    - `em.createNamedQuery("Named쿼리 이름", 반환타입)`
    
    > Named 쿼리 이름 형식: `엔티티명.쿼리제목` 으로 설정해야 관리도 쉽고, 충돌을 방지할 수 있다.
    > 

- 여러 Named 쿼리 정의 시, `@NamedQueries` 애너테이션을 사용하면 된다.

<br/>

### Named 쿼리 작성 방법: XML에 정의하기

- Named 쿼리를 작성할 때는 XML에 정의하는 것이 편리하다.

- Named 쿼리 작성: `META-INF/ormMember.xml`
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    
    	<named-query name="Member.findByUsername">
    		<query><CDATA[
    			select m
    			from Member m
    			where m.username = :username
    		]></query>
    	</named-query>
    
    	<named-query name="Member.count">
    		<query>select count(m) from Member m</query>
    	</named-query>
    
    </entity-mappings>
    ```
    
    - `<CDATA[]>` 를 사용하면 대괄호 사이에 있는 문자을 그대로 출력한다. 그러므로 xml의 예약문자(`>` , `<` 등)을 쿼리에서 사용할 수 있다.
- 그리고 `ormMember.xml` 파일을 인식하도록 `META-INF/persistence.xml`에 아래 코드를 추가해야 한다.
    
    ```xml
    <persistence-unit name="jpabook">
    	<mapping-file>META-INF/ormMember.xml</mapping-file>
    ...
    ```
    
    > META-INF/orm.xml 은 JPA가 기본 매핑파일로 인식해서 별도의 설정을 하지 않아도 된다.
    > 
    
<br/>

### 환경에 따른 설정

- 만약 xml과 애너테이션에 같은 설정이 있으면, **xml을 우선으로 적용**한다.
    - 만약 같은 이름의 Named 쿼리가 xml과 애너테이션에 모두 정의되어 있다면, xml에 정의된 것이 사용된다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>