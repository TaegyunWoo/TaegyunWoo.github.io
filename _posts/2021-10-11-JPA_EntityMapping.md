---
category: JPA
tags: [JPA]
title: "[JPA] 엔티티 매핑"
date:   2021-10-11 22:06:00 
lastmod : 2021-10-11 22:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- [예시 프로젝트 다운로드](https://github.com/TaegyunWoo/jpa-study/tree/master/BasicJpaApp)

<br/><br/>

# 엔티티 매핑

## 개요

- JPA를 사용하는 데 가장 중요한 일은 엔티티와 테이블을 정확히 매핑하는 것이다.
- 따라서 매핑 애너테이션을 숙지하고 사용해야 한다.
- 아래는 매핑 애너테이션의 종류를 분류한 것이다.

<br/>

### 매핑 애너테이션의 종류

- **객체와 테이블 매핑**
    - `@Entity`
    - `@Table`
- **기본 키 매핑**
    - `@Id`
- **필드와 컬럼 매핑**
    - `@Column`
- **연관관계 매핑**
    - `@ManyToOne`
    - `@JoinColumn`

이번 포스팅 글에서는 '객체와 테이블 매핑', '기본 키 매핑', '필드와 컬럼 매핑'에 대해 알아보겠다. '연관관계 매핑'은 추후에 포스팅할 예정이다.

<br/><br/>

## `@Entity`

### 애너테이션 설명

- 테이블과 매핑할 클래스는 `@Entity` 애너테이션을 필수적으로 붙여야한다.
- `@Entity` 가 붙은 클래스는 JPA가 관리하게 된다. 이러한 클래스를 **엔티티**라고 부른다.

- `@Entity` 속성 정리
  |속성|기능|기본값|
  |----|----|------|
  |name|- JPA에서 사용할 엔티티 이름을 지정한다.  <br/> - 보통 클래스 이름(기본값)을 사용한다. <br/> - 만약 다른 패키지에 이름이 같은 엔티티 클래스가 있다면, 이름을 지정해서 충돌하지 않도록 해야 한다.|클래스 이름을 그대로 사용한다.|

<br/>

### `@Entity` 적용시 주의사항

- **기본 생성자가 필수적으로 필요하다.**
    - 기본 생성자: 매개변수가 없는 생성자
    - JPA가 엔티티 객체를 생성할 때, 기본 생성자를 사용하므로 이 생성자가 반드시 필요하다.
- **final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.**
- **저장할 필드에 final을 사용하면 안된다.**

<br/><br/>

## `@Table`

### 애너테이션 설명

- 엔티티와 매핑할 테이블을 지정하는 애너테이션이다.
- 생략시, 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

[@Table 속성 정리](https://www.notion.so/faf0fc7178104a3c8171ba0db019bb40)

- `@Table` 속성 정리
  |속성|기능|기본값|
  |----|----|------|
  |name|- 매핑할 테이블의 이름|엔티티(클래스)이름 사용|
  |catalog|- catalog 기능이 있는 DB에서 catalog를 매핑한다.||
  |schema|-schema 기능이 있는 DB에서 schema를 매핑한다.||
  |uniqueConstraints|- DDL 생성 시에 유니크 제약조건을 만든다. <br/> - 2개 이상의 복합 유니크 제약조건도 만들 수 있다. <br/> - 자동생성기능을 사용해서 DDL을 만들 때만 사용된다. <br/> (추후 설명 예정)||

<br/><br/>

## 다양한 매핑 사용 예시

- 예시 코드는 글 상단을 참고하자.

<br/>

### 추가된 요구사항

기존에 작성한 회원 관리 프로그램에 아래 요구사항이 추가되었다고 하자.

1. 회원은 일반 회원과 관리자로 구분된다.
2. 회원 가입일과 수정일이 있어야 한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

<br/>

### `Member` 엔티티 수정

```java
@Entity
@Table(name="MEMBER")
public class Member {

  @Id
  @Column(name = "ID")
  private String id;

  @Column(name = "NAME")
  private String username;

  private Integer age;

	//--------추가 내용-----------
  @Enumerated(EnumType.STRING) //Enum Type 매핑시 필요한 애너테이션
  private RoleType roleType;

  @Temporal(TemporalType.TIMESTAMP) //날짜 매핑시 필요한 애너테이션
  private Date createdDate;

  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;

  @Lob
  private String description;

  // getter, setter 생략
}
```

- `roleType`
    - 자바의 enum을 사용해서 회원 타입을 구분한다.
    - 자바의 enum을 사용하려면 `@Enumerated` 애너테이션을 통해 매핑해야 한다.
- `createdDate` , `lastModifiedDate`
    - 자바의 날짜 타입은 `@Temporal` 을 사용해서 매핑한다.
- `description`
    - 회원을 설명하는 필드는 길이 제한이 없다.
    - DB의 VARCHAR 타입 대신, CLOB 타입으로 저장해야한다.
    - `@Lob` 애너테이션을 사용하면, CLOB, BLOB 타입을 매핑할 수 있다.

각 항목에 대해서는 글 후반부에서 자세히 다룬다.

<br/>

### `RoleType` Enum 클래스 추가

```java
public enum RoleType {
	ADMIN, USER
}
```

<br/><br/>

## DB 스키마 자동 생성

### JPA의 스키마 자동 생성이란?

- JPA는 DB 스키마를 자동으로 생성하는 기능을 지원한다.
- JPA는 이 매핑정보와 DB 방언을 사용해서 DB 스키마를 생성한다.

<br/>

### 스키마 자동 생성 기능 사용 설정

- 스키마 자동 생성 기능을 사용하기 위해, `persistence.xml` 에 아래 내용을 추가해야한다.

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

- **이 속성을 추가하면, APP 실행 시점에 DB 테이블을 자동으로 생성한다.**
- `value`의 값은 여러 속성으로 변경될 수 있다.

<br/>

### 스키마 자동 생성 특징

- 개발자의 수고를 덜어준다.
- 스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지는 않다.
    - 따라서, 개발 환경에서 사용하거나 매핑을 어떻게 해야 한는지 참호하는 정도로만 사용하는 것이 좋다.
- 스키마 자동 생성하기는 엔티티와 테이블을 어떻게 매핑해야 하는지 알려주는 가장 훌륭한 학습 도구이다.

<br/>

### hibernate.hbm2ddl.auto 속성

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

위 코드에서 `value` 의 값을 변경하여 다양한 옵션을 적용할 수 있다.

- hibernate.hbm2ddl.auto 속성
  |옵션|설명|
  |----|----|
  |create|- 기존 테이블이 존재한다면, 해당 테이블을 삭제하고 새로 생성한다.<br/> - DROP+CREATE|
  |create-drop|- create 옵션과 함께, 애플리케이션 종료시 생성한 DDL을 제거한다.<br/>- DROP+CREATE+DROP|
  |update|- DB 테이블과 엔티티 매핑정보를 비교해서 변경사항만 수정한다.|
  |validate|- DB 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다.<br/>- DDL을 수정하지 않는다.|
  |none|- 자동 생성 기능을 사용하지 않도록 한다.<br/>- 단순히 관련 \<property> 태그를 없애도 된다.|

<br/>

### 주의사항

- 운영서버에서 `create` , `create-drop` , `update` 처럼 DDL을 수정하는 옵션은 절대로 사용하면 안된다.
- 왜냐하면, 운영 DB의 정보가 날라갈 수 도 있고 임의적으로 변경될 수 있기 때문이다.
- 개발 환경에 따른 추천 전략
    - 테스트 서버
        - `update`
        - `validate`
    - 스테이징과 운영 서버
        - `validate`
        - `none`

<br/><br/>

## DDL 생성 기능

### 추가된 제약조건

- 회원 이름
    - 필수적으로 입력해야한다.
    - 10자를 초과하면 안된다.

위 제약조건을 예시 프로젝트에 추가해보자.

<br/>

### `Member` 엔티티 수정

```java
@Entity
@Table(name="MEMBER")
public class Member {

  @Id
  @Column(name = "ID")
  private String id;

  @Column(name = "NAME", nullable=false, length=10) //제약조건 추가
  private String username;

  // 나머지 생략...
}
```

- `nullable=false`
    - 자동 생성되는 DDL에 `not null` 제약조건을 추가할 수 있다.
- `length`
    - 자동 생성되는 DDL에 문자의 크기를 지정할 수 있다.

- 위 코드를 통해 자동으로 만들어진, 테이블 생성 SQL
    
    ```sql
    CREATE TABLE MEMBER (
    	ID varchar(255) not null,
    	NAME varchar(10) not nullm
    	...
    	PRIMARY KEY (ID)
    )
    ```
    
    - DDL의 NAME 컬럼을 보면, `not null` 제약조건이 추가되었음을 확인할 수 있다.
    - 또한, `varchar(10)` 으로 문자의 크기가 10자리로 제한되었다.

- 이번에는 유니크 제약조건을 만들어보자.
    - 유니크 제약조건: 중복된 값을 허락하지 않는 것, 여러 컬럼 조합에 대해서 중복이 되지 않도록 설정할 수도 있다.
    
    ```java
    @Entity
    @Table(name="MEMBER", uniqueConstraints = {
    	@UniqueConstraint(
    		name = "NAME_AGE_UNIQUE", // 제약조건 이름 설정
    		columnNames = {"NAME", "AGE"} //NAME과 AGE 칼럼의 조합이 중복되는 튜플 허가 X
    	)
    })
    public class Member {
    
      @Id
      @Column(name = "ID")
      private String id;
    
      @Column(name = "NAME")
      private String username;
    
      // 나머지 생략...
    }
    ```
    

- **이런 기능들은 오직 DDL을 자동 생성할 때만 사용된다.**
- **실제 JPA 실행 로직에는 영향을 주지 않는다.**
- 해당 기능을 사용하면 애플리케이션 개발자가 엔티티만 보고도 손쉽게 다양한 제약조건을 파악할 수 있는 장점이 있다.

<br/><br/>

## 기본 키 매핑

### 기본 키를 매핑하는 방법들

기존에는 `@Id` 애너테이션만 사용하여 기본키를 매핑하였다. 이러한 방식은 애플리케이션에서 (로직상) 직접 값을 할당하여 매핑하는 방식이었다. 그렇다면, DB가 생성해주는 값을 사용하려면 어떻게 매핑해야 할까?

아래는 JPA가 제공하는 DB 기본 키 생성 전략의 종류이다.

- **직접 할당**
    - 기본 키를 애플리케이션에서 직접 할당한다.
- **자동 생성**
    - 대리 키를 사용하는 방식이다.
    - **IDENTITY**
        - 기본 키를 애플리케이션에서 직접 할당한다.
    - **SEQUENCE**
        - DB 시퀀스를 사용해서 기본 키를 할당한다.
    - **TABLE**
        - 키 생성 테이블을 사용한다.

이처럼 다양한 방식으로 기본 키를 할당할 수 있다. 그 중 자동 생성을 통해 기본 키를 매핑하는 방식은 총 3가지 있는데, 이것은 DB 벤더(제조사)마다 지원하는 방식이 다르기 때문이다.

<br/>

### 설정 추가

키 생성 전략을 사용하기 전에, `persistence.xml` 에 아래 속성을 추가해야 한다.

```java
<property name="hibernate.id.new_generator_mappings" value="true" />
```

이제 본격적으로, 기본 키 할당 전략에 대해 자세히 알아보자.

<br/>

### 기본키 직접 할당 전략

- 기본 키를 직접 할당하려면 다음 코드와 같이 `@Id`로 매핑하면 된다.

```java
@Id
@Column(name = "id")
private String id;
```

- `@Id` 적용 가능한 자바 타입은 아래와 같다.
    - 자바 기본형
    - 자바 래퍼형 (wrapper)
    - String
    - java.util.Date
    - java.sql.Date
    - java.math.BigDecimal
    - java.math.BigInteger

- 기본키 직접 할당 전략은 `em.persist()` 로 **엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법**이다. 이에 대한 예시 코드는 아래와 같다.
    
    ```java
    Board board = new Board();
    board.setId("id1"); //기본 키 직접 할당
    em.persist(board);
    ```
    

- 만약 식별자 값 없이 저장(`em.persist()`)하면 예외가 발생한다.

<br/>

### IDENTITY 전략

- IDENTITY는 기본 키 생성을 DB에 위임하는 전략이다.
- 주로 MySQL 등에서 사용한다. 예시는 아래와 같다.
    
    ```sql
    CREATE TABLE BOARD (
    	ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    	MY_DATA VARCHAR(255)
    );
    
    INSERT INTO BOARD(MY_DATA) VALUES('A'); //ID값 없이 튜플을 저장한다.
    INSERT INTO BOARD(MY_DATA) VALUES('B'); //ID값 없이 튜플을 저장한다.
    ```
    
    - 여기서 **AUTO_INCREMENT** 옵션을 통해, DB가 직접 기본키 값을 생성해준다.
    - 따라서, 추가하려는 튜플의 ID 칼럼의 값을 지정할 필요가 없다.
- **IDENTITY 전략은 DB에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.**

<br/>

- DB가 생성한 키 값을 엔티티에 바인딩하려면 `@GeneratedValue` 애너테이션을 사용해야 한다. 이에 대한 예시코드는 아래와 같다.
    
    ```java
    @Entity
    public class Board {
    
    	@Id
    	@GeneratedValue(strategy = GenerationType.IDENTITY) //IDENTITY 전략 사용 설정
    	private long id;
    
    	// ...
    }
    ```
    
    - **`@GeneratedValue` 의 `strategy` 속성의 값을 `GenerationType.IDENTITY` 로 설정한다.**
    - **이 전략을 사용하면, JPA는 기본 키 값을 얻어오기 위해 DB를 추가로 조회한다.**

<br/>

- IDENTITY 전략을 통한, 엔티티 활용 예시 코드
    
    ```java
    private static void logic(EntityManager em) {
    	Board board = new Board();
    	em.persist(board); //기본키 지정없이 DB에 저장
    	System.out.println("board.id = " + board.getId());
    }
    // 출력: board.id = 1
    ```
    
    - 이때 출력된 값 '1'은 **저장 시점에 DB가 생성한 값을 JPA가 조회한 것**이다.

<br/>

- **IDENTITY 전략의 최적화**
    - 엔티티에 식별자 값을 할당하려면 JPA는 추가로 DB를 조회해야 한다.
    - 이때 성능이 저하된다.
    - `**Statement.getGeneratedKeys()` 를 사용하면, 데이터를 저장하면서 동시에 생성된 기본 키 값도 얻어 올 수 있다.**
    - 따라서, **하이버네이트는 이 메서드를 사용해서 DB와 한 번만 통신한다.**

<br/>

- **IDENTITY 전략과 영속성 컨텍스트**
    - 엔티티가 영속 상태가 되려면 식별자가 반드시 필요하다.
    - 하지만, IDENTITY 전략은 엔티티를 DB에 저장해야만 식별자를 구할 수 있다.
    - **따라서, `em.persist()` 를 호출하는 즉시 INSERT SQL이 DB에 전달된다.**
    - **결론적으로 이 전략은 트랜잭션을 지원하는 "쓰기 지연"이 작동하지 않는다.**
    - **DB에 데이터가 저장되고 난 후에서야, 엔티티가 영속성 컨텍스트에 등록된다.**

<br/>

### SEQUENCE 전략

- 데이터베이스 시퀀스란?
    - 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트이다.
- SEQUENCE 전략은 이 시퀀스를 사용해서 기본 키를 생성한다.
- 오라클, H2 등에서 사용할 수 있다.

<br/>

- 전략 사용법
    1. 시퀀스 생성
        - DB에 시퀀스를 생성해야한다.
        
        ```sql
        CREATE TABLE BOARD (
        	ID BIGINT NOT NULL PRIMARY KEY,
        	MY_DATA VARCHAR(255)
        )
        
        //시퀀스 생성
        CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
        ```
        
    2. 시퀀스 매핑
        - 애플리케이션으로 돌아와서, 시퀀스 매핑 코드를 아래와 같이 작성한다.
        
        ```java
        @Entity
        @SequenceGenerator(
        	name = "BOARD_SEG_GENERATOR", //식별자 생성기 이름
        	sequenceName = "BOARD_SEQ", // DB에 등록되어 있는 시퀀스 이름
        	initialValue = 1, // DDL 생성할 때, 처음 시작하는 수를 지정한다.
        	allocationSize = 1 //시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용)
        )
        public class Board {
        
        	@Id
        	@GeneratedValue(strategy = GenerationType.SEQUENCE,
        							generator = "BOARD_SEQ_GENERATOR") //사용할 시퀀스 생성기 이름
        	private long id;
        
        	// ...
        }
        ```
        
<br/>

- `@SequenceGenerator` 애너테이션
    - 해당 애너테이션을 사용해서, `BOARD_SEQ_GENERATOR` 라는 시퀀스 생성기를 등록한다.
    - 또한, JPA가 시퀀스 생성기를 실제 DB에서 매핑할 시퀀스인 `BOARD_SEQ` 와 연결한다.
    - 아래는 관련 속성에 대한 설명이다.
    - `name`
        - 식별자 생성기 이름을 설정한다.
    - `sequenceName`
        - DB에 등록되어 있는 시퀀스 이름을 설정한다.
    - `initialValue`
        - DDL 생성 시에만 사용된다.
        - 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다.
        - 기본값 = 1
    - `allocationSize`
        - 시퀀스 한 번 호출에 증가하는 수를 설정한다.
        - 성능 최적화에 사용된다.
        - 기본값 = 50
        - 자세한 설명은 나중에 한다.

<br/>

- `@GeneratedValue` 애너테이션
    - 키 생성 전략을 `GenerationType.SEQUENCE` 로 설정한다.
    - `generator` 속성을 통해, `BOARD_SEQ_GENERATOR` 시퀀스 생성기를 선택한다.
        - `BOARD_SEQ_GENERATOR` 는 위에서 `@SequenceGenerator` 를 통해 만든 생성기의 이름이다.
    - 그러면 이제부터 `id` 프로퍼티 식별자 값은 `BOARD_SEQ_GENERATOR` 시퀀스 생성기가 할당한다.

<br/>

- SEQUENCE 전략을 통한, 엔티티 활용 예시 코드
    
    ```java
    private static void logic(EntityManager em) {
    	Board board = new Board();
    	em.persist(board); //기본키 지정없이 DB에 저장
    	System.out.println("board.id = " + board.getId());
    }
    // 출력: board.id = 1
    ```
    
    - 코드 자체는 위에서 설명한 IDENTITY 전략과 같다. 하지만 내부 동작 방식은 다르다.

<br/>

- SEQUENCE 전략의 내부 동작 순서
    1. `em.persist()` 를 호출할 때, **먼저 DB 시퀀스를 사용해서 식별자를 조회**한다.
    2. **조회한 식별자를 엔티티에 할당한 후에, 엔티티를 영속성 컨텍스트에 저장**한다.
    3. 트랜잭션을 커밋하여 **플러시가 일어나면 엔티티를 DB에 저장**한다.

<br/>

- IDENTITY 전략 vs SEQUENCE 전략
    - IDENTITY 전략
        1. **엔티티를 먼저 DB에 저장한다.**
        2. 식별자를 조회해서 다시 엔티티에 바인딩한다.
        3. 이제 엔티티를 영속성 컨텍스트에 저장한다.
    - SEQUENCE 전략
        1. **먼저 시퀀스를 사용하여 식별자를 조회한다.**
        2. 조회된 식별자를 엔티티에 바인딩한다.
        3. 엔티티를 영속성 컨텍스트에 저장한다.
        4. 트랜잭션이 커밋되어 플러시가 일어나면, DB에 엔티티를 저장한다.

<br/>

- `@SequenceGenerator.allocationSize` 속성
    - JPA가 생성하는 기본 시퀀스
        
        ```sql
        CREATE SEQUENCE [sequence_name] START WITH 1 INCREMENT BY 50
        ```
        
        - 즉, JPA가 기본적으로 생성하는 시퀀스는 50씩 증가한다.
    - 따라서, `@SequenceGenerator.allocationSize` 의 기본값도 50으로 설정되어있다.
    - 만약 **DB 시퀀스 값이 1씩 증가한다면, 반드시 값을 1로 설정해야 한다.**

<br/>

- **SEQUENCE 전략과 최적화**
    - SEQUENCE 전략은 DB 시퀀스를 통해, 식별자를 조회하는 추가 작업이 필요하다.
    - 따라서, DB와 2번 통신하게 된다.
        - 먼저 식별자를 구하려고, DB 시퀀스를 조회하는 통신
        - 조회한 시퀀스를 기본 키 값으로 사용해서, DB에 저장하는 통신
    - 그에 따라, JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 `@SequenceGenerator.allocationSize` 를 사용한다.
    - `@SequenceGenerator.allocationSize`에 설정한 값만큼, 한 번에 시퀀스 값을 증가시키고 나서 그만큼 메모리에 시퀀스 값을 할당한다.

<br/>

### TABLE 전략

- TABLE 전략은 **키 생성 전용 테이블을 하나 만들고, 여기에 이름과 값으로 사용할 칼럼을 만들어 DB 시퀀스를 흉내내는 전략**이다.
- 이 전략은 모든 DB에 적용할 수 있다.

<br/>

- 전략 사용법
    1. 키 생성 테이블 만들기
        - 키 생성 용도로 사용할 테이블을 만들어야 한다.
        
        ```sql
        CREATE TABLE MY_SEQUENCES (
        	sequence_name VARCHER(255) NOT NULL,
        	next_val BIGINT,
        	PRIMARY KEY (sequence_name)
        }
        ```
        
        - `sequence_name` : 시퀀스 이름을 저장할 칼럼
        - `next_val` : 시퀀스 값을 저장할 칼럼
        - **위 칼럼명들은 TABLE 전략에서 사용되는 기본적인 이름이다.**
            - 칼럼명을 변경할 수 있지만, 나중에 설명하도록 하겠다.
    2. 시퀀스 매핑
        - 애플리케이션으로 돌아와서, 테이블 전략 매핑 코드를 아래와 같이 작성한다.
        
        ```java
        @Entity
        @TableGenerator(
        	name = "BOARD_SEG_GENERATOR", //식별자 생성기 이름
        	table = "MY_SEQUENCES", // DB에 등록되어 있는 키 생성 테이블 이름
        	pkColumnValue = "BOARD_SEQ" // 사용할 시퀀스 이름
        	allocationSize = 1 //시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용)
        )
        public class Board {
        
        	@Id
        	@GeneratedValue(strategy = GenerationType.TABLE,
        							generator = "BOARD_SEQ_GENERATOR") //사용할 테이블 키 생성기 이름
        	private long id;
        
        	// ...
        }
        ```
        
<br/>

- `@TableGenerator` 애너테이션
    - 해당 애너테이션을 사용하여, 테이블 키 생성기를 등록한다.
    - `BOARD_SEQ_GENERATOR` 라는 이름으로 테이블 키 생성기를 등록한다.
    - 위에서 생성한 `MY_SEQUENCES` 테이블을 키 생성용 테이블로 매핑한다.
    - 아래는 관련 속성에 대한 설명이다.
    - `name`
        - 식별자 생성기 이름을 설정한다.
    - `table`
        - DB의 키 생성 테이블 이름을 설정한다.
        - 기본값 = hibernate_sequences
    - `pkColumnName`
        - 시퀀스 칼럼명을 설정한다.
        - **기본값 = sequence_name**
    - `valueColumnName`
        - 시퀀스 값 칼럼명을 설정한다.
        - **기본값 = next_val**
    - `pkColumnValue`
        - 키로 사용할 값 이름을 설정한다.
        - **기본값 = 엔티티 이름**
    - `initialValue`
        - 초기 값을 설정한다.
        - 마지막으로 생성된 값이 기준이다.
    - `allocationSize`
        - 시퀀스 한 번 호출에 증가하는 수를 설정한다.
        - 성능 최적화에 사용된다.
        - 기본값 = 50
    - `uniqueConstraints(DDL)`
        - 유니크 제약 조건을 지정할 수 있다.

> 위 속성 중 table, pkColumnName, valueColumnName 의 기본값은 하이버네이트 기준이다.

<br/>

- `@GeneratedValue` 애너테이션
    - 해당 애너테이션을 사용하여, 등록된 테이블 키 생성기를 사용한다.
    - `GenerationType.TABLE` 을 선택하여 TABLE 전략을 선택한다.
    - `@GeneratedValue.generator` 속성에 방금 만든 테이블 키 생성기를 지정한다.

<br/>

- TABLE 전략을 통한, 엔티티 활용 예시 코드
    
    ```java
    private static void logic(EntityManager em) {
    	Board board = new Board();
    	em.persist(board); //기본키 지정없이 DB에 저장
    	System.out.println("board.id = " + board.getId());
    }
    // 출력: board.id = 1
    ```
    
    - 시퀀스 대신 테이블을 사용한다는 것을 제외하면, 시퀀스 전략과 내부동작이 같다.

<br/>

- `MY_SEQUENCES` 키 생성 테이블의 변화
    1. 초기 상태
        
        ![Untitled](/assets/img/2021-10-11-JPA_EntityMapping/Untitled%2021.png)
        
    2. 처음 키 생성기 사용시
        - 만약 `@TableGenerator.pkColumnValue` 으로 지정한 이름을 sequence_name 칼럼에서 찾지 못하면 튜플을 새로 추가한다.
        
        ![Untitled](/assets/img/2021-10-11-JPA_EntityMapping/Untitled%2022.png)
        
        - next_val 칼럼이 1로 설정되었다가, 해당 값을 생성기에 전달하고 1이 증가되었다.
        - 왜냐하면 next_val, 즉 다음 키 값을 의미하기 때문이다.
    3. 두 번째 키 생성기 사용시
        
        ![Untitled](/assets/img/2021-10-11-JPA_EntityMapping/Untitled%2023.png)
        
        - next_val 칼럼의 값을 생성기에 전달하고 1이 증가되었다.
    4. 같은 키 생성 테이블을 사용하지만, `@TableGenerator.pkColumnValue` 속성의 값이 'MEMBER_SEQ'인 다른 생성기 사용시
        
        ![Untitled](/assets/img/2021-10-11-JPA_EntityMapping/Untitled%2024.png)
        
        - 이와 같이, 다른 테이블에 대한 키 생성을 할 수 있다.

<br/>

- 흐름 정리
    1. `em.persist(엔티티A)`
    2. 키 생성 테이블에서 `@TableGenerator.pkColumnValue` 로 설정한 값을 pk로 갖는 튜플 찾기
        - 만약 튜플을 못찾을 경우: JPA가 알아서 INSERT 및 초기화
        - 만약 튜플을 찾을 경우: 해당 튜플의 next_val 칼럼의 값을 가져옴
    3. 가져온 next_val 칼럼의 값을 '엔티티A.기본키'에 바인딩
    4. 해당 튜플의 next_val 칼럼 값에 1을 더하여 값 UPDATE
    5. 엔티티A를 영속성 컨텍스트에 등록

<br/>

### AUTO 전략

- `[GenerationType.AUTO](http://GenerationType.AUTO)` 는 선택한 DB 방언에 따라, IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.
- 예시 코드
    
    ```java
    @Entity
    public class Board {
    	@Id
    	@GeneratedValue(strategy = GenerationType.AUTO) // AUTO 전략 선택
    	private Long id;
    
    	//...
    }
    ```

<br/>

- AUTO 전략 사용시 SEQUENCE나 TABLE 전략이 선택되면, **시퀀스나 키 생성 테이블을 미리 만들어 두어야 한다.**
    - 만약, 스키마 자동 생성 기능 (`<property name="hibernate.hbm2ddl.auto" value="create" />`) 을 사용한다면, JPA가 알아서 적절한 테이블이나 시퀀스를 생성해준다.

<br/>

### 정리

지금까지 여러가지 식별자 할당 전략을 살펴보았다. 지금까지 설명한 내용을 정리해보자.

- **직접 할당**
    - `em.persist()` 를 호출하기 전에 애플리케이션에서 직접 식별자 값을 할당해야 한다.
    - 만약 식별자 값이 없으면 예외가 발생한다.
- **IDENTITY**
    - DB에 엔티티를 저장하고 난 뒤, 식별자 값을 획득하여 엔티티에 바인딩하고, 영속성 컨텍스트에 엔티티를 저장한다.
- **SEQUENCE**
    - DB 시퀀스에서 식별자 값을 획득한 후, 엔티티에 바인딩하고, 영속성 컨텍스트에 저장한다.
- **TABLE**
    - DB 키 생성용 테이블에서 식별자 값을 획득한 후, 엔티티에 바인딩하고, 영속성 컨텍스트에 저장한다.
- **AUTO**
    - 알아서 적절한 전략을 선택하게 된다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>