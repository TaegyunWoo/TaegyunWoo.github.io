---
category: JPA
tags: [JPA]
title: "[JPA] 객체지향 쿼리 언어 - 네이티브 쿼리"
date:   2021-12-03 00:03:00 
lastmod : 2021-12-03 00:03:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [[JPA] 객체지향 쿼리 언어 - 소개](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_Begin)
    2. [[JPA] 객체지향 쿼리 언어 - JPQL 기초](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin)
    3. [[JPA] 객체지향 쿼리 언어 - JPQL 조인](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Join)
    4. [[JPA] 객체지향 쿼리 언어 - JPQL 심화](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Expert)
    5. [[JPA] 객체지향 쿼리 언어 - QueryDSL](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_QueryDSL)

<br/><br/>

# 네이티브 SQL

## 개요

### 네이티브 SQL을 사용하는 이유

- JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL 함수들을 지원한다.
- 하지만 특정 데이터베이스에 종속적인 기능은 지원하지 않는다. 이때, 네이티브 SQL을 직접 작성하여 사용할 수 있다.

<br/>

### 종속적인 기능을 사용하는 방법

- **특정 DB만 사용하는 함수 사용하기**
    - 네이티브 SQL 함수 호출하여 사용할 수 있다.
    - 하이버네이트: DB 방언에 각 DB에 종속적임 함수들을 정의해두었고, 이것을 직접 호출할 수 있다.
- **특정 DB만 지원하는 SQL 쿼리 힌트 사용하기**
    - 하이버네이트를 포함한 몇몇 JPA 구현체들이 지원하여, 사용할 수 있다.
- **인라인 뷰(From 절에서 사용하는 서브쿼리), UNION, INTERSECT 사용하기**
    - 하이버네이트는 지원하지 않는다.
- **스토어 프로시저 사용하기**
    - JPQL에서 스토어드 프로시저를 호출할 수 있다.
- **특정 DB만 지원하는 문법 사용하기**
    - 너무 종속적인 SQL 문법은 지원하지 않는다. 이땐 네이티브 SQL을 사용해야 한다.

<br/>

### 네이티브 SQL이란

- 위처럼 다양한 이유로 JPQL을 사용할 수 없을 때, JPA는 SQL을 직접 사용할 수 있는 기능을 제공한다.
    - 이것을 네이티브 SQL이라고 한다.
- 네이티브 SQL은 SQL을 개발자가 직접 정의하는 것이다.
- **네이티브 SQL을 사용하면, 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.**
    - 반면 JDBC API를 직접 사용하면, 단순히 데이터의 나열을 조회할 뿐이다.

<br/><br/>

## 네이티브 SQL 사용

### 네이티브 쿼리 API 종류

- 결과 타입을 정의할 수 있을 때
    
    ```java
    public Query createNativeQuery(String sqlString, Class resultClass);
    ```
    
- 결과 타입을 정의할 수 없을 때
    
    ```java
    //결과 매핑 비사용시
    public Query createNativeQuery(String sqlString);
    
    //결과 매핑 사용시
    public Query createNativeQuery(String sqlString, String resultSetMapping);
    ```
    
<br/>

### 엔티티 조회

- `em.createNativeQuery(SQL문자열, 결과클래스)`
    - 네이티브 SQL은 해당 메서드를 사용한다.
    - 첫번째 파라미터
        - 네이티브 SQL 문자열
    - 두번째 파라미터
        - 조회할 엔티티 클래스의 타입
- 엔티티 조회 코드
    
    ```java
    //SQL 정의
    String sql =
    	"SELECT ID, AGE, NAME, TEAM_ID " +
    	"FROM MEMBER WHERE AGE > ?";
    
    //위치기반 파라미터 사용
    Query nativeQuery = em.createNativeQuery(sql, Member.class)
    		.setParameter(1, 20);
    
    List<Member> resultList = nativeQuery.getResultList();
    ```
    
    - **네이티브 SQL 사용 시, '위치 기반 파라미터'만 사용할 수 있다.**
    - **네이티브 SQL로 SQL만 직접 사용할 뿐이고, 나머지는 JPQL을 사용할 때와 같다.**
        - **조회한 엔티티 역시, 영속성 컨텍스트에서 관리된다.**

> JPA는 공식적으로 네이티브 SQL에서 이름 기반 파라미터를 지원하지 않고, 위치 기반 파라미터만 지원한다.  
**하지만 하이버네이트는 네이티브 SQL에 이름 기반 파라미터를 사용하는 것을 허용한다.**
> 

> `em.createNativeQuery()` 에 반환타입을 지정해도, `TypeQuery` 가 아닌 `Query` 를 리턴한다.  
이것은 JPA의 규약이 정의되어 그렇다. 너무 신경쓰지 말자.
> 

<br/>

### 값 조회

```java
//SQL 정의
String sql =
	"SELECT ID, AGE, NAME, TEAM_ID " +
	"FROM MEMBER WHERE AGE > ?";

//조회할 타입이 없다.
Query nativeQuery = em.createNativeQuery(sql)
		.setParameter(1, 20);

//따라서, Object[] 형으로 값을 받는다.
List<Object[]> resultList = nativeQuery.getResultList();

for (Object[] row : resultList) {
	System.out.println("id = " + row[0]);
	System.out.println("age = " + row[1]);
	System.out.println("name = " + row[2]);
	System.out.println("team_id = " + row[3]);
}
```

- `em.createNativeQuery()`
    - 여러 값으로 조회하려면, 해당 메서드의 두번째 파라미터를 사용하지 않으면 된다.
    - 이때, `Object[]` 에 담아서 반환된다.
- 스칼라 값들을 조회한 것이므로, 영속성 컨텍스트에서 관리하지 않는다.

<br/>

### 결과 매핑 사용 - 1

- 매핑이 복잡해지면, `@SqlResultSetMapping` 애너테이션을 정의해서 결과 매핑을 사용해야 한다.
    - 매핑 복잡화 예시) 엔티티와 스칼라 값을 함께 조회할 때

<br/>

- 예시 코드: 결과 매핑 사용
    
    ```java
    //SQL 정의
    String sql =
    	"SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
    	"FROM MEMBER M " +
    	"LEFT JOIN " +
    		"(SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
    		"FROM ORDERS O, MEMBER IM " +
    		"WHERE O.MEMBER_ID = IM.ID) I" +
    	"ON M.ID = I.ID";
    
    //"memberWithOrderCount" : 결과 매핑 정보 이름
    Query nativeQuery = em.createNativeQuery(sql, "memberWithOrderCount");
    
    List<Object[]> resultList = nativeQuery.getResultList();
    
    for (Object[] row : resultList) {
    	Member member = (Member) row[0];
    	BigInteger orderCount = (BigInteger) row[1];
    
    	System.out.println("member = " + member);
    	System.out.println("orderCount = " + orderCount);
    }
    ```
    
    - `em.createNativeQuery(sql, "memberWithOrderCount")`
        - 두 번째 파라미터에 결과 매핑 정보의 이름이 사용되었다.

- 예시 코드: 결과 매핑 정보 정의
    
    ```java
    @Entity
    @SqlResultSetMapping(
    	name = "memberWithOrderCount",
    	entities = {@EntityResult(entityClass = Member.class)},
    	columns = {@ColumnResult(name = "ORDER_COUNT")}
    )
    public class Member {
    	//...
    }
    ```
    
    - 회원 엔티티 및 `ORDER_COUNT` 칼럼을 매핑했다.
        - `ID` , `AGE` , `NAME` , `TEAM_ID` 는 Member 엔티티와 매핑된다.
        - `ORDER_COUNT` 는 단순히 값으로 매핑된다.

<br/>

### 결과 매핑 사용 - 2

JPA 표준 명세이 있는 예제 코드로 결과 매핑을 어떻게 하는지 좀 더 자세히 알아보자.

- 표준 명세 예제 - SQL
    
    ```java
    Query q = em.createNativeQuery(
    	"SELECT o.id AS order_id, " +
    		"o.quantity AS order_quantity, " +
    		"o.item AS order_item, " +
    		"i.name AS item_name, " +
    	"FROM Order o, Item i " +
    	"WHERE (order_quantity > 25) AND (order_item = i.id)"
    , "OrderResults");
    ```
    
- 표준 명세 예제 - 매핑 정보
    
    ```java
    @SqlResultSetMapping(
    	name = "OrderResults",
    	entities = {
    		@EntityResult(entityClass = com.acme.Order.class, fields={
    			@FieldResult(name="id", column="order_id"),
    			@FieldResult(name="quantity", column="order_quantity"),
    			@FieldResult(name="item", column="order_item")
    		})
    	},
    	columns = {
    		@ColumnResult(name = "item_name")
    	}
    )
    ```
    
- 상세 설명
    - `@FieldResult`
        - 해당 애너테이션을 사용해서, 칼럼명과 필드명을 직접 매핑한다.
        - SQL에서 프로젝션이 전부 별칭으로 설정되어 있기 때문이다.
    - `@FieldResult` 를 한번이라도 사용하면, 전체 필드를 `@FieldResult`로 매핑해야 한다.

<br/>

- 아래처럼 두 엔티티를 조회하는데 칼럼명이 중복될 때도, `@FieldResult`를 사용해야 한다.
    
    ```sql
    // 칼럼명 충돌이 일어난다.
    SELECT A.ID, B.ID FROM A, B
    
    // 칼럼명 충돌 방지
    SELECT
    	A.ID AS A_ID,
    	B.ID AS B_ID
    FROM A, B
    ```
    
    - 이것을 `@FieldResult` 으로 매핑해야 한다.

<br/>

### 결과 매핑 애너테이션

- `@SqlResultSetMapping` 속성

| 속성 | 기능 |
| --- | --- |
| name | 결과 매핑 이름 |
| entities | @EntityResult 를 사용해서 엔티티를 결과로 매핑한다. |
| columns | @ColumnResult 를 사용해서 칼럼을 결과로 매핑한다. |

<br/>

- `@EntityResult` 속성

| 속성 | 기능 |
| --- | --- |
| entityClass | 결과로 사용할 엔티티 클래스를 지정한다. |
| fields | @FieldResult 를 사용해서 결과 칼럼을 필드와 매핑한다. |
| discriminatorColumn | 엔티티의 인스턴스 타입을 구분하는 필드 (상속에서 사용된다.) |

<br/>

- `@FieldResult` 속성

| 속성 | 기능 |
| --- | --- |
| name | 결과를 받을 필드명 |
| column | 결과 칼럼명 |

<br/>

- `@ColumnResult` 속성

| 속성 | 기능 |
| --- | --- |
| name | 결과 칼럼명 |

<br/><br/>

## Named 네이티브 SQL

### 엔티티 조회 예시 코드

- Named 네이티브 SQL 정의
    
    ```java
    @Entity
    @NamedNativeQuery(
    	name = "Member.memberSQL",
    	query = "SELECT ID, AGE, NAME, TEAM_ID " +
    		"FROM MEMBER WHERE AGE > ?",
    	resultClass = Member.class
    )
    public class Member {...}
    ```
    
- Named 네이티브 SQL 사용
    
    ```java
    TypedQuery<Member> nativeQuery =
    		em.createNamedeQuery("Member.memberSQL", member.class)
    		.setParameter(1, 20);
    ```
    

- 상세 설명
    - JPQL Named 쿼리와 같은 `createNamedQuery()` 메서드를 사용하여, 네이티브 Named 쿼리를 사용한다.
    - 차이점은 오직 `@NamedNativeQuery` ↔ `@NamedQuery` 밖에 없다.

<br/>

- 결과 매핑을 사용하는 Named 네이티브 SQL 정의
    
    ```java
    @Entity
    @NamedNativeQuery(
    	name = "Member.memberWithOrderCount",
    	query = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
    		"FROM MEMBER " +
    		"LEFT JOIN " +
    			"(SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
    			"FROM ORDERS O, MEMBER IM " +
    			"WHERE O.MEMBER_ID = IM.ID) I" +
    		"ON M.ID = I.ID",
    	resultSetMapping = "memberWithOrderCount" //해당 결과 매핑 사용
    )
    public class Member {...}
    ```
    
    - `resultSetMapping = "memberWithOrderCount"`
        - 해당 속성을 통해, `memberWithOrderCount` 결과 매핑을 사용한다.

<br/>

### NamedNativeQuery 속성

| 속성 | 기능 |
| --- | --- |
| name | 네임드 쿼리 이름 (필수) |
| query | SQL 쿼리 (필수) |
| hints | 벤더 종속적인 힌트 |
| resultClass | 결과 클래스 |
| resultSetMapping | 결과 매핑 사용 |

<br/>

### 여러 Named 네이티브 쿼리 선언

여러 Named 네이티브 쿼리를 선언하려면 아래처럼 `@NamedNativeQueries` 애너테이션을 사용하면 된다.

```java
@NamedNativeQueries({
	@NamedNativeQuery(...),
	@NamedNativeQuery(...)
)
```

<br/><br/>

## 네이티브 SQL를 XML에 정의하기

### 예시 코드

- ormMember.xml
    
    ```xml
    <entity-mappings ...>
    
    	<named-native-query name="Member.memberWithOrderCountXml"
    		result-set-mapping="memberWithOrderCountResultMap">
    		<query><CDATA[
    			SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT FROM MEMBER M
    			LEFT JOIN
    				(SELECT IM.ID, COUNT(*) AS ORDER_COUNT
    				FROM ORDERS O, MEMBER IM
    				WHERE O.MEMBER_ID = IM.ID)
    			ON M.ID = I.ID
    		]></query>
    	</named-native-query>
    
    	<sql-result-set-mapping name="memberWithOrderCountResultMap">
    		<entity-result entity-class="jpabook.domain.Member"/>
    		<column-result name="ORDER_COUNT"/>
    	</sql-result-set-mapping>
    
    </entity-mappings>
    ```
    
- 상세설명
    - XML에 정의할 때는 순서를 지켜야 한다. 순서는 아래와 같다.
        1. `<named-native-query>` 정의
        2. `<sql-result-set-mapping>` 정의
    
    > 애너테이션 보다는 xml을 사용하는 것이 더 편리하다.
    > 

<br/>

### 네이티브 SQL 정리

- 네이티브 SQL도 JPQL을 사용할 때와 마찬가지로 `Query` , `TypedQuery` 를 반환한다.
    - 따라서 JPQL API를 그대로 사용할 수 있다.
- 네이티브 SQL은 JPQL이 자동 생성하는 SQL을 수동을 직접 정의하는 것이다.
    - 따라서 JPA가 제공하는 기능 대부분을 그대로 사용할 수 있다.

<br/>

- 권장 방식
    - 될 수 있으면 표준 JPQL을 사용하라.
    - 기능이 부족하면 하이버네이트와 같은 JPA 구현체가 제공하는 기능을 사용하라.
    - 그래도 안된다면, 마지막 방법으로 네이티브 SQL을 사용하자.

<br/><br/>

## 스토어드 프로시저(JPA 2.1)

### 스토어드 프로시저란?

- 여러 쿼리를 하나의 함수처럼 사용하는 것을 말한다. 이것은 DB에서 제공하는 기능이다.

<br/>

### 스토어드 프로시저 사용

- MySQL DB에 선언된 프로시저
    
    ```sql
    DELIMITER //
    
    CREATE PROCEDURE proc_multiply (INOUT inParam INT, INOUT outParam INT)
    BEGIN
    	SET outParam = inParam * 2;
    END //
    ```
    
    - 매개변수 (자세한 것은 DB 관련 자료를 찾아보자.)
        - `IN`
            - 호출자에게 전달받은 값을 복사해서, 프로시저 내부에서만 사용한다.
            - **프로시저 내부 값 전달 O, 값 반환 X**
        - `OUT`
            - 값을 전달받지는 않고, 프로시저 내부에서의 최종값을 호출자에게 전달한다.
            - 초기값=NULL
            - **프로시저 내부 값 전달 X, 값 반환 O**
        - `INOUT`
            - `IN` + `OUT`
            - **프로시저 내부 값 전달 O, 값 반환 O**

<br/>

- JPA를 통해, 스토어드 프로시저 호출
    
    ```java
    //사용할 프로시저 객체 생성
    StoredProcedureQuery spq =
    	em.createStoredProcedureQuery("proc_multiply"); //정의된 프로시저를 불러온다.
    
    //첫번째 매개변수 정의
    spq.registerStoredProcedureParameter(1, Integer.class, ParameterMode.IN);
    
    //두번째 매개변수 정의
    spq.registerStoredProcedureParameter(2, Integer.class, ParameterMode.OUT);
    
    //첫번째 파라미터에 100 전달
    spq.setParameter(1, 100);
    
    //프로시저 실행
    spq.execute();
    
    //Output 파라미터인 2번 파라미터 값 확인
    Integer out = (Integer) spq.getOutputParameterValue(2);
    
    System.out.println("out = " + out);
    //결과: out = 200
    ```
    

- 상세 설명
    - `em.createStoredProcedureQuery()` 메서드
        - 파라미터에 사용할 스토어드 프로시저 이름을 넘겨줘야 한다.
    - `registerStoredProcedureParameter()` 메서드
        - 프로시저에 사용할 파라미터를 아래와 같은 순서로 정의한다.
            1. 순서
            2. 타입
            3. 파라미터 모드
        - 파라미터 모드 종류
            - `IN` , `INOUT` , `OUT` , `REF_CURSOR`
        - 파라미터에 순서 대신 이름을 사용할 수 있다.
            
            ```java
            spq.setparameter("inParam", 100);
            spq.execute();
            ```
            
<br/>

### Named 스토어드 프로시저 사용

- 스토어드 프로시저 쿼리에 아래와 같이 이름을 부여해서 사용하는 것을 **Named 스토어드 프로시저**라고 한다.
    
    ```java
    @NamedStoredProcedureQuery(
    	name = "multiply",
    	procedureName = "proc_multiply",
    	parameters = {
    		@StoredProcedureParameter(name = "inParam",
    			mode = ParameterMode.IN, type = Integer.class),
    		@StoredProcedureParameter(name = "outParam",
    			mode = ParameterMode.OUT, type = Integer.class),
    	}
    )
    @Entity
    public class Member {...}
    ```
    

- Named 스토어드 프로시저 사용방법
    1. `@NamedStoredProcedureQuery` 로 사용할 프로시저를 정의한다.
    2. `name` 속성으로 이름을 부여한다.
    3. `procedureName` 속성에 실제 호출할 프로시저 이름을 작성한다.
    4. `@StoredProcedureParameter` 를 사용해서 파라미터 정보를 저으이한다.
    
    > 둘 이상을 정의하려면 `@NamedStoredProcedureQueries` 를 사용하면 된다.
    > 

<br/>

### Named 스토어드 프로시저 XML에 정의하기

- xml 정의
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    
    	<named-stored-procedure-query name="multiply"
    		procedure-name="proc_multiply">
    		<parameter name="inParam" mode="IN" class="java.lang.Integer"/>
    		<parameter name="outParam" mode="OUT" class="java.lang.Integer"/>
    	</named-stored-procedure-query>
    
    </entity-mappings>
    ```
    
- 사용하기
    
    ```java
    //xml에 정의한 프로시저 생성
    StoredProcedureQuery spq =
    	em.createNamedStoredProcedureQuery("multiply");
    
    //매개변수에 값 전달
    spq.setParameter("inParam", 100);
    
    //실행
    spq.execute();
    
    Integer out = (Integer) spq.getOutputParameterValue("outParam");
    System.out.println("out = " + out);
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