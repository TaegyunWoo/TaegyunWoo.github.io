---
category: JPA
tags: [JPA]
title: "[JPA] 고급매핑 (1)"
date:   2021-11-07 18:36:00 
lastmod : 2021-11-07 18:36:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 이번 포스팅에서 다룰 내용

이번 포스팅에서 다룰 고급 매핑 내용은 아래와 같다.

- 상속 관계 매핑
- `@MappedSuperclass`

아래는 다음 포스팅에서 다룰 고급 매핑 내용이다.

- 복합 키와 식별 관계 매핑
- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑하기

하나의 게시글에서 다루기엔 분량이 꽤 많으므로, 총 3번에 걸쳐서 설명하겠다.

<br/><br/>

# 상속 관계 매핑

## 개요

- 관계형 DB에는 상속이라는 개념이 없다.
- 대신, 슈퍼타입 서브타입 관계라는 모델링 기법이 존재한다.
    - 이것은 객체의 상속 개념과 가장 유사하다.
- ORM에서의 상속 관계 매핑이란?
    - 객체의 상속 구조와 데이터베이스의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

<br/>

### 슈퍼타입 서브타입 논리 모델

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2020.png)

<br/>

### 객체 상속 모델

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2021.png)

<br/>

### 슈퍼타입 서브타입 논리 모델을 테이블로 구현하는 방법의 종류

총 3가지의 방법이 있다.

- **각각 테이블로 변환하는 방법**
    - 각각을 모두 테이블로 만든다.
    - 조회할때 조인을 사용한다.
    - 이것을 JPA에서는 **조인 전략** 이라고 한다.

<br/>

- **통합 테이블로 변환하는 방법**
    - 테이블을 하나만 사용해서 통합한다.
    - 이것을 JPA에서는 **단일 테이블 전략** 이라고 한다.

<br/>

- **서브타입 테이블로 변환하는 방법**
    - 서브 타입마다 하나의 테이블을 만든다.
    - 이것을 JPA에서는 **구현 클래스마다 테이블 전략** 이라고 한다.

지금부터 하나씩 알아보자.

<br/><br/>

## 조인 전략

### 조인 전략이란?

- 엔티티 각각을 모두 테이블로 만들다.
- 자식 테이블과 부모 테이블이 식별관계인 전략이다.
    - 식별관계란, '자식 테이블이 부모 테이블의 기본 키를 받아서, 기본 키 + 외래 키로 사용하는 것'
- 타입을 구분하는 컬럼을 추가해야 한다. 여기서는 DTYPE 컬럼을 구분 컬럼으로 사용한다.
    - 왜냐하면, 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없다.

- 조인전략 테이블 시각화
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2022.png)
    
<br/>

### 예제 코드

- **부모 클래스**
    
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.JOINED)
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;
    
    	private String name;
    	private int price;
    
    	//getter, setter 생략
    
    }
    ```

<br/>

- **자식 클래스**
    
    ```java
    @Entity
    @DiscriminatorValue("A")
    public class Album extends Item {
    
    	private String artist;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    @DiscriminatorValue("M")
    public class Movie extends Item {
    
    	private String director;
    	private String actor;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    @DiscriminatorValue("B")
    public class Book extends Item {
    
    	private String author
    	private String isbn;
    
    	//getter, setter 생략
    
    }
    ```
    
    - `ITEM_ID` 칼럼과 자동으로 매핑된다.
        - 부모 클래스로부터 기본키 필드를 물려받는다.
        - **물려받은 PK 필드를 '자신과 매핑된 테이블(자식테이블)의 PK·FK 칼럼'과 자동으로 매핑한다.**
        - 이때, 매핑할 PK·FK 칼럼명으로 **부모의 PK칼럼명을 그대로 사용** 한다. (기본설정)

<br/>

- 상세 설명
    - `@Inheritance(strategy = InheritanceType.JOINED)`
        - 상속 매핑을 사용하려면, 부모 클래스에 `@Inheritance` 를 설정해야 한다.
        - 그리고 조인 전략을 사용할 것이므로, 해당 애너테이션의 속성을 `strategy = InheritanceType.JOINED` 로 설정한다.
    - `@DiscriminatorColumn(name = "DTYPE")`
        - 부모 클래스에 구분 칼럼을 지정한다.
        - 이 칼럼으로 저장된 자식 테이블을 구분할 수 있다.
        - `name` 속성의 기본값이 `DTYPE` 이므로, `@DiscriminatorColumn` 으로 줄여서 사용해도 된다.
    - `@DiscriminatorValue("M")`
        - 엔티티를 저장할 때, 구분 칼럼에 입력할 값을 지정한다.
        - 만약 영화 엔티티를 저장하면, 구분 칼럼 DTYPE에 값 M이 저장된다.

<br/>

- '자식 테이블의 PK·FK 칼럼명'이 '부모 테이블의 PK 칼럼명'과 다를 때
    - 위에서 설명했듯, '부모 클래스의 PK 필드'는 자식 클래스가 물려받아 '자식 클래스의 PK·FK 필드'로 사용한다.
    - 이때, '자식 클래스의 PK·FK 필드가 매핑되는 자식 테이블의 칼럼명'은 '부모 테이블의 PK 칼럼명'을 그대로 사용한다.
    - 만약 매핑되는 칼럼명을 변경하고 싶다면, `@PrimaryKeyJoinColumn` 애너테이션을 사용하면 된다.
    
    ```java
    @Entity
    @DiscriminatorValue("M")
    @PrimaryKeyJoinColumn(name = "BOOK_ID")
    //상속받은 PK·FK 필드는 ITEM_ID 칼럼이 아닌, BOOK_ID 칼럼과 매핑된다.
    public class Movie extends Item {
    
    	private String director;
    	private String actor;
    
    	//getter, setter 생략
    
    }
    ```
    
<br/>

### 조인 전략의 장단점과 특징

조인 전략을 사용했을 때의 장점과 단점은 아래와 같다.

- 장점
    - 테이블이 정규화된다.
    - 외래 키 참조 무결성 제약조건을 활용할 수 있다.
    - 저장공간을 효율적으로 사용한다.
- 단점
    - 조회할 때, 조인이 많이 사용되므로 성능이 저하될 수 있다.
    - 조회 쿼리가 복잡하다.
    - 데이터를 등록할 때, INSERT SQL을 두 번 실행한다.

아래는 조인 전략의 특징이다.

- JPA 표준 명세는 구분 컬럼을 사용하도록 한다.
- 하지만, 하이버네이트를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다.
- 즉, `@DiscriminatorColumn` 을 사용하지 않아도 동작한다.

<br/><br/>

## 단일 테이블 전략

### 단일 테이블 전략이란?

- 테이블을 하나만 사용하는 전략이다.
- 구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분한다.
- 조회할 때, 조인을 사용하지 않으므로 일반적으로 가장 빠르다.

- 단일 테이블 시각화
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2023.png)
    

- **자식 엔티티가 매핑한 칼럼은 모두 null을 허용해야 한다!**
    - 예시) Book 엔티티 저장시
    - `ITEM_ID` , `NAME` , `PRICE` , `AUTHOR` , `ISBN` , `DTYPE` 칼럼만 사용한다.
    - `ARTIST` , `DIRECTOR` , `ACTOR` 칼럼은 모두 비워져있다.

<br/>

### 예시 코드

- **부모 클래스**
    
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE) //전략 변경
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;
    
    	private String name;
    	private int price;
    
    	//getter, setter 생략
    
    }
    ```
    
<br/>

- **자식 클래스**
    
    > 자식 클래스는 변경사항이 없다.
    > 
    
    ```java
    @Entity
    @DiscriminatorValue("A")
    public class Album extends Item {
    
    	private String artist;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    @DiscriminatorValue("M")
    public class Movie extends Item {
    
    	private String director;
    	private String actor;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    @DiscriminatorValue("B")
    public class Book extends Item {
    
    	private String author
    	private String isbn;
    
    	//getter, setter 생략
    
    }
    ```
    
<br/>

- 상세 설명
    - `@Inheritance` 의 속성을 `strategy = InheritanceType.SINGLE_TABLE` 로 변경하여, 단일 테이블 전략으로 설정한다.
    - **이때, 테이블 하나에 모든 것을 통합하므로, 구분 컬럼을 필수로 사용해야 한다.**

<br/>

### 단일 테이블 전략의 장단점과 특징

단일 테이블 전략을 사용했을 때의 장점과 단점은 아래와 같다.

- 장점
    - 조인 필요 없으므로, 일반적으로 조회 성능이 빠르다.
    - 조회 쿼리가 단순하다.
- 단점
    - 자식 엔티티가 매핑한 칼럼은 모두 null을 허용해야 한다.
    - 단일 테이블에 모든 것을 저장하므로, 테이블이 커질 수 있다.
        - 그러므로 상황에 따라서, 조회 성능이 오히려 느려질 수 있다.

아래는 단일 테이블 전략의 특징이다.

- **구분 컬럼 (`DTYPE`)을 반드시 사용해야 한다.**
    - 즉, `@DiscriminatorColumn` 을 사용해야 한다.
    - 왜냐하면 `DTYPE` 칼럼이 없다면, 자식 엔티티를 구분할 방법이 없다.
- `@DiscriminatorValue` 를 지정하지 않으면, 기본적으로 엔티티 이름을 사용한다.
    - 예시) `Movie` , `Book` , `Album`

<br/><br/>

## 구현 클래스마다 테이블 전략

### 구현 클래스마다 테이블 전략이란?

- 아래 그림과 같이, 자식 엔티티마다 테이블을 만드는 전략이다.
- 자식 테이블 각각에 필요한 칼럼이 모두 존재한다.

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2024.png)

<br/>

### 예시 코드

- **부모 클래스**
    
    > `@DiscriminatorColumn` 가 필요없다.
    > 
    
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) //전략 변경
    // @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;
    
    	private String name;
    	private int price;
    
    	//getter, setter 생략
    
    }
    ```
<br/>

- **자식 클래스**
    
    > `@DiscriminatorValue` 가 필요없다.
    > 
    
    ```java
    @Entity
    // @DiscriminatorValue("A")
    public class Album extends Item {
    
    	private String artist;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    //@DiscriminatorValue("M")
    public class Movie extends Item {
    
    	private String director;
    	private String actor;
    
    	//getter, setter 생략
    
    }
    ```
    
    ```java
    @Entity
    //@DiscriminatorValue("B")
    public class Book extends Item {
    
    	private String author
    	private String isbn;
    
    	//getter, setter 생략
    
    }
    ```
    
<br/>

- 상세 설명
    - `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`
        - 구현 클래스마다 테이블 전략을 사용하도록 설정한다.
    - **이 전략은 일반적으로 추천하지 않는 전략이다!**
        - 이 전략의 단점이 너무 크다. 아래에서 설명한다.

<br/>

### 구현 클래스마다 테이블 전략의 장단점과 특징

구현 클래스마다 테이블 전략을 사용했을 때의 장점과 단점은 아래와 같다.

- 장점
    - 서브 타입을 구분해서 처리할 때 효과적이다.
    - not null 제약조건을 사용할 수 있다.
- 단점
    - 여러 자식 테이블을 함께 조회할 때, 성능이 느리다.
        - SQL에 UNION을 사용해야 하기 때문이다.
    - 자식 테이블을 통합해서 쿼리하기 어렵다.

아래는 구현 클래스마다 테이블 전략의 특징이다.

- 구분 컬럼을 사용하지 않는다.
- **웬만하면 조인 전략이나 단일 테이블 전략을 고려하자!**

<br/><br/>

# `@MappedSuperclass` 애너테이션

## 개요

### `@MappedSuperclass` 이란?

- '부모 클래스'는 테이블과 매핑하지 않고, '부모 클래스를 상속 받는 자식 클래스'에게 매핑 정보만 제공하고 싶을 때, 해당 애너테이션을 사용하면 된다.
- 즉 추상 클래스와 비슷하지만...
    - `@Entity` 는 실제 테이블과 매핑된다.
    - `@MappedSuperclass` 는 실제 테이블과 매핑되지는 않는다.
- **그러므로 `@MappedSuperclass` 는 단순히 매핑 정보를 상속할 목적으로만 사용된다.**
    - 즉, 공통속성(필드)를 묶기 위해서 사용된다.

<br/>

### `@MappedSuperclass` 를 적용한 객체 시각화

- 테이블 관계
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2025.png)
    

- 엔티티 관계
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_1/Untitled%2026.png)
    

- 회원(`MEMBER`)와 판매자(`SELLER`)는 서로 관계가 없는 테이블이다.
- 회원(`Member`)와 판매자(`Seller`)는 서로 관계가 없는 엔티티이다.
- 여기서 테이블은 그대로 두고, 객체 모델의 `id` , `name` 두 공통 속성을 부모 클래스로 모으고 객체 상속 관계로 만들어보자.

<br/><br/>

## 예시 코드

### 부모 클래스

```java
@MappedSuperclass
public abstract class BaseEntity {

	@Id @GeneratedValue
	private Long id;
	private String name;

	//getter, setter 생략

}
```

- 이 클래스를 통해, 공통 속성을 묶는다.

<br/>

### 자식 클래스

- Member 클래스
    
    ```java
    @Entity
    public class Member extends BaseEntity {
    
    	// ID 매핑정보 상속 받음 => 'id 필드', 'MEMBER.ID 칼럼' 매핑
    	// NAME 매핑정보 상속 받음 => 'name 필드', 'MEMBER.NAME 칼럼' 매핑
    	private String email;
    
    	//getter, setter 생략
    
    }
    ```
    
- Seller 클래스
    
    ```java
    @Entity
    public class Seller extends BaseEntity {
    
    	// ID 매핑정보 상속 받음 => 'id 필드', 'SELLER.ID 칼럼' 매핑
    	// NAME 매핑정보 상속 받음 => 'name 필드', 'SELLER.NAME 칼럼' 매핑
    	private String shopName;
    
    	//getter, setter 생략
    
    }
    ```
    
<br/>

- 상세 설명
    - `BaseEntity` 클래스
        - 객체들이 주로 사용하는 공통 매핑 정보를 정의했다.
        - 테이블과 매핑할 필요가 없다.
        - **이 클래스는 단지, 자식 엔티티에게 공통으로 사용되는 매핑 정보만 제공하면 된다.**
        - 따라서, `@MappedSuperclass` 를 적용했다.
    - 자식 엔티티
        - 상속을 통해, `BaseEntity` 의 매핑 정보를 물려받았다.

<br/>

### 물려받은 매핑 정보, 연관관계 재정의

- 부모로부터 물려받은 매핑 정보를 재정의하려면 아래 애너테이션을 사용하면 된다.
    - `@AttributeOverrides`
    - `@AttributeOverride`
    - 예시 코드 - 1
        
        ```java
        @Entity
        @AttributeOverride(
        	name = "id",
        	column = @Column(name = "MEMBER_ID")
        )
        public class Member extends BaseEntity {
        
        	// ...
        
        }
        ```
        
        - 부모에게 상속받은 `id` 속성(필드)과 매핑되는 칼럼명을 `MEMBER_ID` 로 재정의했다.
        - 따라서 `Member.id` 필드는 `ID` 가 아닌, `MEMBER_ID` 칼럼과 매핑된다.

    - 예시 코드 - 2
        
        ```java
        @Entity
        @AttributeOverrides({
        	@AttributeOverride(name = "id", column = @Column(name = "Member_ID")),
        	@AttributeOverride(name = "name", column = @Column(name = "Member_NAME"))
        })
        public class Member extends BaseEntity {
        
        	// ...
        
        }
        ```
        
        - `@AttributeOverrides` 를 통해, 여러 매핑 정보를 오버라이딩할 수 있다.

<br/>

- 부모로부터 물려받은 연관관계를 재정의하려면 아래 애너테이션을 사용하면 된다.
    - `@AssociationOverrides`
    - `@AssociationOverride`
    > 따로 설명하지는 않겠다.

<br/><br/>

## 정리

### `@MappedSuperclass` 특징

- 테이블과 매핑되지 않고, 자식 클래스에 **엔티티의 매핑 정보를 상속**하기 위해 사용한다.
- `@MappedSuperclass` 로 지정한 클래스는 엔티티가 아니므로, `em.find()` 나 JPQL에서 사용할 수 없다.
- 해당 애너테이션이 적용된 클래스는 직접 사용될 일이 없다.
    - 따라서 추상 클래스로 만드는 것을 권장한다.
- 즉, `@MappedSuperclass` 는 단순히 공통으로 사용하는 매핑 정보를 모아주는 역할만을 한다.

<br/>

### 엔티티가 상속받을 수 있는 것

- 엔티티(`@Entity`)는 엔티티(`@Entity`)이거나, `@MappedSuperclass` 로 지정한 클래스만 상속받을 수 있다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>