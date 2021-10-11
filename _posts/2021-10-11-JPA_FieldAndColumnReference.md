---
category: JPA
tags: [JPA]
title: "[JPA] 필드와 칼럼 매핑 Reference"
date:   2021-10-11 23:06:00 
lastmod : 2021-10-11 23:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 필드와 칼럼 매핑 관련 Reference

## 개요

- 본 게시글은 JPA가 제공하는 필드와 칼럼 매핑용 애너테이션들을 레퍼런스 형식으로 정리한 것이다.
- 간단히 훑어만 보고, 필요한 매핑을 사용할 일이 있을 때 찾아서 자세히 읽어보자.

<br/><br/>

## `@Column` 애너테이션

- `@Column` 은 객체 필드를 테이블 칼럼에 매핑한다.
- 가장 많이 사용되고 기능도 많다.
- 주요 속성
    - **name**
        - 필드와 매핑할 테이블의 컬럼 이름을 지정한다.
        - 기본값 = 객체의 필드 이름
    - **nullable**
        - DDL 관련 속성
        - null 값의 허용 여부를 설정한다.
        - false로 설정하면 DDL 생성시에 not null 제약조건이 붙는다.
        
        ```java
        //적용 예시
        @Column(nullable = false)
        private String data;
        
        //결과: 생성된 DDL
        data VARCHAR(255) NOT NULL
        ```
        
    - **unique**
        - DDL 관련 속성
        - 한 칼럼에 간단히 유니크 제약조건을 걸 때 사용한다.
        - 두 컬럼 이상을 사용해서 유니크 제약조건을 사용하려면 클래스 레벨에서 `@Table.uniqueConstraints` 를 사용해야 한다.
        
        ```java
        //적용 예시
        @Column(UNIQUE = TRUE)
        private String data;
        
        //결과: 생성된 DDL
        ALTER TABLE tablename ADD CONSTRAINT uk_xxx UNIQUE (data)
        ```
        
    - **columnDefinition**
        - DDL 관련 속성
        - DB 칼럼 정보를 직접 줄 수 있다.
        
        ```java
        //적용 예시
        @Column(columnDefinition = "varchar(100) default 'Empty'")
        private String data;
        
        //결과: 생성된 DDL
        data VARCHAR(100) DEFAULT 'Empty'
        ```
        
    - **length**
        - DDL 관련 속성
        - 문자 길이 제약조건
        - String 타입에만 사용한다.
        
        ```java
        //적용 예시
        @Column(length = 400)
        private String data;
        
        //결과: 생성된 DDL
        data VARCHAR(400)
        ```
        
    - **precision, scale**
        - DDL 관련 속성
        - BigDecimal 타입에서 사용한다.
        - precision: 소수점을 포함한 전체 자릿수
        - scale: 소수의 자릿수
        - 아주 큰 숫자나 정밀한 소수를 다뤄야할 때만 사용
        
        ```java
        //적용 예시
        @Column(precision = 10, scale = 2)
        private BigDecimal cal;
        
        //결과: 생성된 DDL
        cal NUMERIC(10,2) //H2
        cal NUMBER(10,2) //오라클
        cal DECIMAL(10,2) //MySQL
        ```

<br/>

### `@Column` 생략시 주의사항

- `@Column` 생략시 주의해야하는 사항이 있다.
- 자바 기본 타입에서 `@Column` 생략시
    - DDL에서 not null 제약조건이 자동으로 추가된다.
    - 왜냐하면, 기본 타입에 null이 들어올 수 없기 때문이다.
- 객체 타입에서 `@Column` 생략시
    - DDL에서 not null 제약조건이 추가되지 않는다.

<br/><br/>

## `@Enumerated` 애너테이션

- `@Enumerated` 애너테이션은 자바의 enum 타입을 매핑할 때 사용한다.

- 주요 속성
    - **value**
        - `EnumType.ORDINAL` : enum 순서를 DB에 저장한다. (기본값)
        - `EnumType.STRING` : enum 이름을 DB에 저장한다.

- 사용 예시
    
    ```java
    enum RoleType {
    	ADMIN, USER
    }
    ```
    
    ```java
    @Enumerated(EnumType.STRING) // enum 이름을 DB에 저장
    pritvate RoleType roleType1;
    
    @Enumerated // enum 순서를 DB에 저장 (EnumType.ORDINAL 은 기본값이므로 생략가능)
    pritvate RoleType roleType2;
    ```
    
    - `EnumType.STRING`
        - enum 이름 그대로 'ADMIN'이나 'USER'라는 문자를 그대로 DB에 저장한다.
        - 장점: enum의 순서가 바뀌거나 enum이 추가되어도 안전하다.
        - 단점: `EnumType.ORDINAL` 보다는 데이터 크기가 크다.
    - `EnumType.ORDINAL`
        - enum에 정의된 순서대로 `ADMIN` 은 0, `USER` 는 1 값이 DB에 저장된다.
        - 장점: DB에 저장되는 데이터 크기가 작다.
        - 단점: enum의 순서가 바뀌거나 enum이 추가되면 DB의 데이터가 잘못된 값을 의미하게 된다.

<br/><br/>

## `@Temporal` 애너테이션

- 이 애너테이션은 날짜 타입을 매핑할 때 사용한다.
- 매핑되는 타입
    - `java.util.Date`
    - `java.util.Calendar`

- 주요 속성
    - **value**
        - `TemporalType.DATE` : 날짜, DB date 타입과 매핑 (예: 2021-10-11)
        - `TemporalType.TIME` : 시간, DB time 타입과 매핑 (예: 10:30:57)
        - `TemporalType.TIMESTAMP` : 날짜와 시간, DB timestamp 타입과 매핑 (예: 2021-10-11 10:30:57)

- 사용 예시
    
    ```java
    @Temporal(TemporalType.DATE)
    private Date date; //날짜
    
    @Temporal(TemporalType.TIME)
    private Date time; //시간
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date timestamp; //날짜와 시간
    
    //==생성된 DDL==//
    date DATE,
    time TIME,
    timestamp TIMESTAMP
    ```
    

- `@Temporal` 생략시 DDL에서 timestamp로 정의된다.

<br/><br/>

## `@Lob` 애너테이션

- DB의 BLOB, CLOB 타입과 매핑된다.
- 속성이 없다.
- 매핑하는 필드 타입이 문자일때 ⇒ CLOB 으로 매핑
    - String, char[], java.sql.CLOB
- 나머지 ⇒ BLOB 으로 매핑
    - byte[], java.sql.BLOB

- 사용 예시
    
    ```java
    @Lob
    private String lobString;
    
    @Lob
    private Byte[] lobByte;
    ```

<br/><br/>

## `@Transient` 애너테이션

- 해당 애너테이션이 적용된 필드는 매핑하지 않는다.
- 객체에 임시로 어떤 값을 보관하고 싶을 때 사용한다.
- 사용 예시
    
    ```java
    @Transient
    private String temp;
    ```
    
<br/><br/>

## `@Access` 애너테이션

- 해당 애너테이션은 JPA가 엔티티 데이터에 접근하는 방식을 지정한다.
- 접근 방식 종류
    - **필드 접근**
        - `AccessType.FIELD` 로 지정한다.
        - 필드에 직접 접근한다.
        - 필드 접근 권한이 private이어도 접근할 수 있다.
        - "엔티티.필드명"으로 접근
    - **프로퍼티 접근**
        - `AccessType.PROPERTY` 로 지정한다.
        - 접근자를 사용한다.
        - "엔티티.get필드명()"으로 접근

<br/>

### `@Id` 위치에 따른 접근 방식 결정

- `@Access` 를 설정하지 않으면 `@Id` 의 위치를 기준으로 접근 방식이 설정된다.
- **필드 접근**
    
    ```java
    @Entity
    public class Member {
    
    	@Id
    	private String id;
    	
    	//...
    }
    ```
    
    - `@Access(AccessType.FIELD)` 를 클래스 레벨에 적용한 것과 동일하다.
- **프로퍼티 접근**
    
    ```java
    @Entity
    public class Member {
    
    	private String id;
    	
    	@Id
    	public String getId() {
    		return id;
    	}	
    }
    ```
    
    - `@Access(AccessType.PROPERETY)` 를 클래스 레벨에 적용한 것과 동일하다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>