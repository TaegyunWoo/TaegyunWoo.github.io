---
category: JPA
tags: [JPA]
title: "[JPA] 고급매핑 (2)"
date:   2021-11-08 13:36:00 
lastmod : 2021-11-08 13:36:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    - [[JPA] 고급매핑 (1)](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_1)

<br/>

# 이번 포스팅에서 다룰 내용

[이전 포스팅](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_1)에서 다룬 고급 매핑 내용은 아래와 같다.

- 상속 관계 매핑
- `@MappedSuperclass`

아래는 이번 포스팅에서 다룰 고급 매핑 내용이다.

- 복합 키와 식별 관계 매핑

아래는 다음 포스팅에서 다룰 고급 매핑 내용이다.

- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑하기

<br/><br/><br/>

# 복합 키와 식별 관계 매핑

## 개요

- 기본 복합 키를 매핑하는 방법
- 식별 관계, 비식별 관계를 매핑하는 방법

위 두가지에 대해 지금부터 설명하겠다.

<br/><br/>

## 식별 관계 vs 비식별 관계

DB 테이블 사이 관계는 '외래키가 기본키에 포함되는지 여부'에 따라 식별 관계와 비식별 관계로 구분한다. 지금부터 각 관계의 특징을 이해해보자.

<br/>

### 식별 관계

- 식별 관계는 **부모 테이블의 기본 키를 내려받아서 '자식 테이블의 기본키+외래키'로 사용하는 관계**이다.

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2027.png)

- 위 그림을 보자. `PARENT` 테이블의 기본키 `PARENT_ID` 가 `CHILD` 테이블의 기본키이자 외래키로 사용되었다.
- 이것을 식별관계라고 한다.

<br/>

### 비식별 관계

- 비식별 관계는 **부모 테이블의 기본키를 받아서 '자식 테이블의 외래키'로 사용하는 관계**이다.
- 비식별 관계에는 두 가지 종류가 있다.
    - 필수적 비식별 관계
    - 선택적 비식별 관계

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2028.png)

- 위 그림을 보자. `PARENT` 테이블의 기본키 `PARENT_ID` 를 `CHILD` 테이블의 외래키로만 사용한다.
- 이것을 비식별 관계라고 한다.
- 필수적 비식별 관계 vs 선택적 비식별 관계
    - **필수적 비식별 관계**
        - `CHILD` 테이블의 `PARENT_ID` 칼럼이 NOT NULL 제약조건을 갖는다.
        - 연관관계를 필수적으로 맺어야 한다.
    - **선택적 비식별 관계**
        - `CHILD` 테이블의 `PARENT_ID` 칼럼이 NULL을 허용한다.
        - 연관관계를 맺을지 말지 선택할 수 있다.

> **최근에는 비식별 관계를 주로 사용하고, 꼭 필요한 곳에만 식별 관계를 사용하는 추세이다.**

<br/><br/>

## 복합 키: 비식별 관계 매핑

### 복합 키를 매핑하기 위한 기본적인 지식

- JPA에서 식별자를 둘 이상 사용하려면, **별도의 식별자 클래스**를 만들어야 한다.
- 즉 아래 코드는 **불가능한 코드**이다.
    
    ```java
    @Entity
    public class Hello {
    	@Id
    	private String id1;
    
    	@Id
    	private String id2; // 실행 시점에 매핑 예외가 발생한다.
    }
    ```
    

- JPA는 **영속성 컨텍스트에 엔티티를 보관할 때 '엔티티의 식별자'를 키로 사용**한다.
    - 식별자를 구분하기 위해, `equals` 와 `hashCode` 메서드를 사용하여 동등성 비교를 한다.
    - 엔티티를 저장하는 공간인, (영속성 컨텍스트의) **1차 캐시는 HashMap과 유사한 구조**를 갖는다.
    - **따라서 어떤 엔티티를 저장하거나 조회할 때, 키에 해당하는 객체(여기선 식별자 객체)의 `equals` 와 `hashCode` 메서드가 필요하다.**
        
        > `equals` 와 `hashCode` 메서드에 대해선, [이전에 다룬 포스팅](https://taegyunwoo.github.io/java/Java_EqualsAndHashcode)을 참고하자
        > 

<br/>

- JPA는 복합 키를 지원하기 위해 아래 방법들을 제공한다.
    - `@IdClass`
        - 관계형 DB에 가까운 방법이다.
    - `@EmbeddedId`
        - 객체지향에 가까운 방법이다.
- 지금부터 위 방법들에 대해 하나씩 알아보자.

<br/>

### `@IdClass`

예시를 통해 `@IdClass` 애너테이션을 알아보자.

- 예시
    - 가정: 테이블 상태는 아래 그림과 같다.
        
        ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2029.png)
        
    - 이때, `PARENT` 테이블은 기본 복합키를 사용한다. (기본키 칼럼이 2개이다.)
    - 따라서 기본 복합키를 매핑하기 위해, 식별자 클래스를 별도로 만들어야 한다.
    - **부모 클래스**
        
        ```java
        @Entity
        @IdClass(ParentId.class) // 사용할 식별자 클래스 지정
        public class Parent {
        
        	@Id
        	@Column(name = "PARENT_ID1")
        	private String id1; //ParentId.id1과 연결된다.
        
        	@Id
        	@Column(name = "PARENT_ID2")
        	private String id2; //ParentId.id2과 연결된다.
        
        	// getter, setter 생략
        }
        ```
        
    - **부모 식별자 클래스**
        
        ```java
        //식별자 클래스는 Serializable 인터페이스를 implements 해야한다.
        public class ParentId implements Serializable {
        
        	private String id1; //Parent.id1 매핑
        	private String id2; //Parent.id2 매핑
        
        	public ParentId() {} // 기본 생성자
        
        	public ParentId(String id1, String id2) {
        		this.id1 = id1;
        		this.id2 = id2;
        	}
        
        	//영속성 컨텍스트의 1차 캐시에서 key로 사용되기 위해 위해 필요한 메서드
        	@Override
        	public boolean equals(Object o) {
        		//...
        	}
        
        	@Override
        	public int hashCode() {
        		//...
        	}
        
        }
        ```
        
<br/>

- `@IdClass(A.class)` 애너테이션의 의미
    - **'해당 애너테이션이 붙은 엔티티 클래스'의 각 속성(필드) 중, `@Id` 가 붙은 속성(필드)를 `A 식별자 클래스`의 속성(필드)와 매핑시키겠다는 의미이다.**
    - 이때, 각 속성의 이름을 비교하여 매핑시킨다.
    - 즉, 아래처럼 매핑된다.
        - **테이블의 기본키 칼럼 ↔ 식별자 클래스의 속성 ↔ 엔티티 클래스의 속성**

<br/>

- 식별자 클래스의 조건
    - **'식별자 클래스의 속성명'과 '엔티티에서 사용하는 식별자의 속성명'이 같아야 한다.**
        - `Parent.id1` 과 `ParentId.id1` 끼리 매핑된다.
        - `Parent.id2` 와 `ParentId.id2` 끼리 매핑된다.
    - **`Serializable` 인터페이스를 구현해야 한다.**
    - **`equals` , `hashCode` 를 구현해야 한다.**
    - **기본 생성자가 있어야 한다.**
    - **식별자 클래스는 `public` 클래스이어야 한다.**

<br/>

- 식별자 클래스의 필드
    - **식별자 클래스의 각 필드는 '복합키를 갖는 엔티티의 각 기본키 필드와 매핑'된다.**
    - 복합키를 갖는 엔티티의 종류
        - 엔티티의 기본키 필드가 객체형인 경우
            - 해당 엔티티가 다른 엔티티와 연관관계를 맺는다는 뜻이다.
            - **따라서, 식별자 클래스의 필드는 '다른 엔티티의 기본키'를 담아야 한다.**
            - 왜냐하면, 다른 엔티티를 구분해야 하기 때문이다.
        - 엔티티의 기본키 필드가 기본형인 경우
            - **식별자 클래스의 필드는 그대로 '해당 엔티티의 필드값'을 담는다.**

<br/>

- 복합 키를 사용하는 엔티티 저장하기
    
    ```java
    Parent parent = new Parent();
    parent.setId1("myId1"); //식별자 설정
    parent.setId2("myId2"); //식별자 설정
    parent.setName("parentName");
    em.persist(parent);
    ```
    
    - 위 코드는 parent 엔티티를 저장하는 코드이다. 여기서 이상한 점을 눈치챘는가?
    - **식별자 클래스인 `ParentId` 와 관련된 코드가 존재하지 않는다!**
        - `em.persist()` 를 호출시 영속성 컨텍스트에 엔티티를 등록하기 직전에, **내부에서 `Parent.id1` , `Parent.id2` 값을 사용하여 식별자 클래스 `ParentId` 를 생성하고 영속성 컨텍스트의 키로 사용**한다.

<br/>

- 복합 키를 사용하는 엔티티 조회하기
    
    ```java
    ParentId parentId = new ParentId("myId1", "myId2"); //복합키 생성
    Parent parent = em.find(Parent.class, parentId); // 식별자 인스턴스로 조회하기
    ```
    

<br/>

- 복합키를 가진 부모 엔티티와 '선택적 비식별 연관관계'를 갖는 자식 클래스 추가
    
    ```java
    @Entity
    public class Child {
    
    	@Id
    	private String childId;
    
    	@ManyToOne
    	@JoinColumns({
    		@JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
    		@JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2")
    	})
    	private Parent parent;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- `@JoinColumns` 애너테이션
    - 외래키 매핑시, 여러 칼럼을 매핑해야 할 때 `@JoinColumns` 를 사용한다.
        - 위 경우 **부모 테이블이 기본 복합키를 사용**하므로, **자식 테이블의 외래키도 '기본 복합키로 사용된 여러 칼럼'을 외래키**로 사용해야 한다.
    - `name` 속성
        - `CHILD` 테이블의 FK 칼럼명
    - `referencedColumnName` 속성
        - `CHILD` 테이블의 FK 칼럼과 매핑될 '`PARENT` 테이블'의 칼럼명
        - `name` 값과 `referencedColumnName` 값이 같다면, `referencedColumnName` 은 생략할 수 있다.

<br/>

### `@EmbeddedId`

`@EmbeddedId` 는 좀 더 객체지향적인 방법이다. 이것 역시, 예시를 통해 알아보자.

<br/>

- 예시
    - 가정: 테이블 상태는 위(`@IdClass` 설명 부분)와 같다.
    - **부모 클래스**
        
        ```java
        @Entity
        public class Parent {
        
        	@EmbeddedId
        	private ParentId parentId;
        
        	private String name;
        
        	// getter, setter 생략
        }
        ```
        
    - **부모 식별자 클래스**
        
        ```java
        @Embeddable
        public class ParentId implements Serializable {
        
        	@Column(name = "PARENT_ID1") // PARENT 테이블의 PARENT_ID1 칼럼과 매핑된다.
        	private String id1;
        
        	@Column(name = "PARENT_ID2") // PARENT 테이블의 PARENT_ID2 칼럼과 매핑된다.
        	private String id2;
        	
        	public ParentId() {} // 기본 생성자
        
        	public ParentId(String id1, String id2) {
        		this.id1 = id1;
        		this.id2 = id2;
        	}
        
        	//영속성 컨텍스트의 1차 캐시에서 key로 사용되기 위해 위해 필요한 메서드
        	@Override
        	public boolean equals(Object o) {
        		//...
        	}
        
        	@Override
        	public int hashCode() {
        		//...
        	}
        }
        ```
        
        - `@Column`
            - `@IdClass` 와는 다르게, 식별자 클래스의 필드에 `@Column` 을 붙여줘야 한다.
            - 왜냐하면 '복합키를 쓰는 엔티티'에서 `@Column` 을 사용하지 않아, 각 필드를 어떤 칼럼과 매핑해야하는지 모르기 때문이다.
            - **즉 `@IdClass` 와는 다르게, `@Embeddable` 를 적용한 식별자 클래스는 식별자 클래스에 기본키를 직접 매핑한다.**

<br/>

- 식별자 클래스의 조건
    - `@Embeddable` 애너테이션을 붙여야 한다.
    - `Serializable` 인터페이스를 구현해야 한다.
    - `equals` , `hashCode` 메서드를 오버라이딩해야 한다.
    - 기본 생성자가 있어야 한다.
    - 식별자 클래스는 `public` 클래스이어야 한다.

<br/>

- 복합키를 사용하는 엔티티 저장하기
    
    ```java
    Parent parent = new Parent();
    ParentId parentId = new ParentId("myId1", "myId2"); // 키 생성
    parent.setId(parentId); //식별자 설정
    parent.setName("parentName");
    em.persist(parent);
    ```
    
    - `@IdClass` 와는 다르게, `ParentId` 를 직접 생성해서 사용한다.

<br/>

- 복합키를 사용하는 엔티티 조회하기
    
    ```java
    ParentId parentId = new ParentId("myId1", "myId2"); // 키 생성
    Parent parent = em.find(Parent.class, parentId);
    ```
    
<br/>

### 복합 키와 `equals()` , `hashCode()`

- JPA는 **영속성 컨텍스트에 엔티티를 보관할 때 '엔티티의 식별자'를 키로 사용**한다.
- 식별자를 구분하기 위해, `equals` 와 `hashCode` 메서드를 사용하여 동등성 비교를 한다.
- 엔티티를 저장하는 공간인, (영속성 컨텍스트의) **1차 캐시는 HashMap과 유사한 구조**를 갖는다.
- **따라서 어떤 엔티티를 저장하거나 조회할 때, 키에 해당하는 객체(여기선 식별자 객체)의 `equals` 와 `hashCode` 메서드가 필요하다.**
    
    > `equals` 와 `hashCode` 메서드에 대해선, [이전에 다룬 포스팅](https://taegyunwoo.github.io/java/Java_EqualsAndHashcode)을 참고하자.

<br/>

### `@IdClass` vs `@EmbeddedId`

- `@EmbeddedId` 가 대체로 좋아보이지만, 반드시 그렇지는 않다.
- `@EmbeddedId` 는 특정 상황에서 JPQL이 좀더 길어질 수 있다.

<br/>

### 참고사항

- 복합키에는 `@GenerateValue` 를 사용할 수 없다.
- 복합 키를 구성하는 여러 칼럼 중 하나에도 사용할 수 없다!

<br/><br/>

## 복합 키: 식별 관계 매핑

- 위에서 비식별 관계를 매핑하는 방법에 대해 다뤘다.
- 이제 식별 관계를 매핑하는 방법을 알아보자.

<br/>

### 설명을 위한 기본 가정

- 테이블 상태는 아래 그림과 같다고 가정한 뒤, 설명을 계속하도록 하겠다.

![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2030.png)

- 위 그림은 부모, 자식, 손자까지 계속 기본키를 전달하는 식별관계이다.
- 식별 관계에서 자식·손자 테이블은 부모 테이블의 기본키를 포함해서 복합키를 구성한다.
    - 따라서, `@IdClass` 나 `@EmbeddedId` 를 사용해서 식별자를 매핑해야 한다.
- 이제 본격적으로 식별관계 매핑에 대해 알아보자.

<br/>

### `@IdClass` 와 식별 관계

바로 예시 코드를 통해 알아보자.

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @Column(name = "PARENT_ID")
    	private String id;
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **자식 클래스**
    
    ```java
    @Entity
    @IdClass(ChildId.class) // 식별자 클래스 설정
    public class Child {
    
    	@Id
    	@ManyToOne
    	@JoinColumn(name = "PARENT_ID") // CHILD 테이블의 PARENT_ID와 매핑
    	private Parent parent;
    
    	@Id @Column(name = "CHILD_ID")
    	private String childId;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **자식 식별자 클래스**
    
    ```java
    public class ChildId implements Serializable {
    
    	//Child.parent와 매핑된다
    	//**Parent 엔티티의 기본키 값을 담는다**
    	private String parent; //필드명이 같아야 한다.
    
    	//Child.childId와 매핑된다
    	//**Child 엔티티의 기본키 childId 값을 담는다**
    	private String childId; //필드명이 같아야 한다.
    	
    	public ChildId() {} // 기본 생성자
    
    	public ChildId(String id1, String id2) {
    		this.id1 = id1;
    		this.id2 = id2;
    	}
    
    	//영속성 컨텍스트의 1차 캐시에서 key로 사용되기 위해 위해 필요한 메서드
    	@Override
    	public boolean equals(Object o) {
    		//...
    	}
    
    	@Override
    	public int hashCode() {
    		//...
    	}
    }
    ```
    
<br/>

- **손자 클래스**
    
    ```java
    @Entity
    @IdClass(GrandChildId.class) // 식별자 클래스 설정
    public class GrandChild {
    
    	@Id
    	@ManyToOne
    	@JoinColumns({
    		@JoinColumn(name = "PARENT_ID"),
    		@JoinColumn(name = "CHILD_ID")
    	}) // 여러 칼럼으로 구성된 PK를 가져와, FK로 사용하므로 @JoinColumns 를 사용한다.
    	private Child child;
    
    	@Id @Column(name = "GRANDCHILD_ID")
    	private String id;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **손자 식별자 클래스**
    
    ```java
    public class GrandChildId implements Serializable {
    
    	// GrandChild.child 와 매핑된다
    	// **Child 엔티티의 기본키 값이 담긴다**
    	private ChildId child; //기본 복합키이므로, 식별자 클래스 객체를 담는다
    
    	// GrandChild.id 와 매핑된다
    	// **GrandChild 엔티티의 기본키 id 값이 담긴다**
    	private String id;
    }
    ```
    
<br/>

- 상세 설명
    - 식별 관계는 기본키와 외래키를 같이 매핑해야 한다.
    - 따라서, `@Id` 와 `@ManyToOne` 을 같이 사용한다.

<br/>

### `@EmbeddedId` 와 식별 관계

`@EmbeddedId` 로 식별 관계를 구성할 땐, `@MapsId` 를 사용해야 한다. 바로 예시를 통해 알아보자.

<br/>

- **부모 클래스**
    
    > 부모 클래스는 변경사항이 없다.
    > 
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @Column(name = "PARENT_ID")
    	private String id;
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **자식 클래스**
    
    ```java
    @Entity
    public class Child {
    
    	@EmbeddedId
    	private ChildId id;
    
    	@MapsId("parentId") //ChildId.parentId 와 매핑 (연관관계 정보 전달)
    	@ManyToOne
    	@JoinColumn(name = "PARENT_ID") // CHILD 테이블의 PARENT_ID와 매핑
    	private Parent parent;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
    - **만약 자식 테이블의 칼럼 `PARENT_ID`가 단순히 PK 칼럼이라면, `parent` 필드를 `Child` 엔티티 클래스에서 선언할 필요가 없다. (물론 이런 경우에는 `parent` 필드가 기본 타입이다.)**
        - 왜냐하면 식별자 클래스를 통해, 매핑 설정을 할 수 있기 때문이다.
    - **하지만 식별 관계인 경우(자식 테이블의 칼럼 `PARENT_ID`가 PK·FK 칼럼인 경우), `parent` 필드를 `Child` 엔티티 클래스에서 선언해야 한다.**
        - 왜냐하면 **식별자 클래스에서 'pk 매핑 설정과 함께 연관관계를 설정'할수는 없기 때문에,** `Child` 클래스에서 연관관계를 설정해야하기 때문이다.
    - `@MapsId("parentId")`
        - 식별자 클래스의 `parentId` 필드에 아래 연관관계를 적용하겠다는 의미와 같다.
            - `@ManyToOne` , `@JoinColumn`
- `@EmbeddedId` 사용 시, 기본키·외래키 설정 위치

    |매핑 설정|설정 위치|
    |---------|---------|
    |PK 설정|식별자 클래스에서 설정한다.|
    |FK 설정|먼저 엔티티 클래스에서 설정한다. <br/> 그 후, 식별자 클래스에게도 연관관계 정보를 전달한다.|

<br/>

- **자식 식별자 클래스**
    
    ```java
    @Embeddable
    public class ChildId implements Serializable {
    
    	// @MapsId가 전달해준 연관관계 매핑
    	private String parentId;
    
    	@Column("CHILD_ID")
    	private String id;
    	
    	public ChildId() {} // 기본 생성자
    
    	public ChildId(String parentId, String id) {
    		this.parentId = parentId;
    		this.id = id;
    	}
    
    	//영속성 컨텍스트의 1차 캐시에서 key로 사용되기 위해 위해 필요한 메서드
    	@Override
    	public boolean equals(Object o) {
    		//...
    	}
    
    	@Override
    	public int hashCode() {
    		//...
    	}
    }
    ```
    
<br/>

- **손자 클래스**
    
    ```java
    @Entity
    public class GrandChild {
    
    	@EmbeddedId
    	private GrandChildId id;
    
    	@MapsId("childId") // GrandChildId.childId 와 매핑 (연관관계 정보 전달)
    	@ManyToOne
    	@JoinColumns({
    		@JoinColumn(name = "PARENT_ID"),
    		@JoinColumn(name = "CHILD_ID")
    	})
    	private Child child;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **손자 식별자 클래스**
    
    ```java
    @Embeddable
    public class GrandChildId implements Serializable {
    
    	// @MapsId가 전달해준 연관관계 매핑
    	private ChildId childId;
    
    	@Column("GRANDCHILD_ID")
    	private String id;
    	
    	public GrandChildId() {} // 기본 생성자
    
    	public GrandChildId(ChildId childId, String id) {
    		this.childId = childId;
    		this.id = id;
    	}
    
    	//영속성 컨텍스트의 1차 캐시에서 key로 사용되기 위해 위해 필요한 메서드
    	@Override
    	public boolean equals(Object o) {
    		//...
    	}
    
    	@Override
    	public int hashCode() {
    		//...
    	}
    }
    ```
    
<br/>

- 상세 설명
    - '식별 관계로 사용할 연관관계의 속성(필드)'에 `@MapsId` 를 적용하면 된다.
    - `@IdClass` 와의 차이점
        - `@IdClass` 는 엔티티의 연관관계 속성(필드)에 `@Id` 를 사용한다.
        - `@EmbeddedId` 는 엔티티의 연관관계 속성(필드)에 `@MapsId` 를 사용한다.
    - `@MapsId` 의 의미
        - **'외래키와 매핑한 연관관계'를 기본키에도 매핑하겠다는 뜻이다.**
        - **즉, 연관관계 정보를 식별자 클래스에게도 넘겨 매핑하겠다는 뜻이다.**
        - **따라서 속성 값으로 '식별자 클래스의 기본키 필드'를 지정하면 된다.**

<br/>

### 비식별 관계로 구현

위에서 설명한 식별 관계를 비식별 관계로 변경하고, 이에 따라 코드가 어떻게 변화하는지 보자.

- 테이블 상태
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2031.png)
    

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **자식 클래스**
    
    ```java
    @Entity
    public class Child {
    
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    
    	@ManyToOne
    	@JoinColumn(name = "PARENT_ID") // CHILD 테이블의 PARENT_ID와 매핑
    	private Parent parent;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **손자 클래스**
    
    ```java
    @Entity
    public class GrandChild {
    
    	@Id @GeneratedValue
    	@Column(name = "GRANDCHILD_ID")
    	private Long id;
    
    	@ManyToOne
    	@JoinColumn(name = "CHILD_ID")
    	private Child child;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
- 기본 복합키를 사용하지 않으니, 상당히 단순해졌다.
    - 식별자 클래스를 만들 필요가 없다.
- 매핑도 매우 간단하다.

<br/>

### 일대일 식별관계

일대일 식별 관계는 약간 특별하다. 바로 예제를 통해 알아보자.

<br/>

- 테이블 상태
    
    ![Untitled](/assets/img/2021-11-07-JPA_Mapping_Expert_2/Untitled%2032.png)
    
    - **일대일 식별관계는 '자식 테이블의 기본키 값으로 부모 테이블의 기본키 값만 사용'한다.**
    - 그래서 **부모 테이블의 기본키가 복합키가 아니면**, **자식 테이블의 기본 키는 복합 키로 구성하지 않아**도 된다.

<br/>

- **부모 클래스 (`BOARD`)**
    
    ```java
    @Entity
    public class Board {
    
    	@Id @GeneratedValue
    	@Column(name = "BOARD_ID")
    	private Long id;
    
    	private String title;
    
    	//양방향
    	@OneToOne(mappedBy = "board")
    	private BoardDetail boardDetail;
    
    	// getter, setter, 편의메서드 생략
    }
    ```
    
<br/>

- **자식 클래스 (`BOARDDETAIL`)**
    
    ```java
    @Entity
    public class BoardDetail {
    
    	@Id
    	private Long boardId;
    
    	@MapsId // BoardDetail.boardId 와 매핑 (연관관계 정보 전달)
    	@OneToOne
    	@JoinColumn(name = "BOARD_ID")
    	private Board board;
    
    	private String content;
    
    	// getter, setter, 편의메서드 생략
    }
    ```
    
    - **`@MapsId` 를 통해, 자신의 PK 필드에 연관관계를 전달한다.**

<br/>

- 상세 설명
    - **`BoardDetail` 처럼 식별자가 단순히 칼럼 하나인 경우, `@MapsId` 를 사용하고, 속성값은 비워두면 된다.**
    - 이때 `@MapsId` 는 `@Id` 를 사용해서 식별자로 지정한 `BoardDetail.boardId` 와 매핑된다.

<br/>

- 일대일 식별관계를 사용하는 코드
    
    ```java
    public void save() {
    	Board board = new Board;
    	board.setTitle("제목");
    	em.persist(board);
    
    	BoardDetail boardDetail = new BoardDetail();
    	boardDetail.setContent("내용");
    	boardDetail.setBoard(board);
    	em.persist(boardDetail);
    }
    ```
    
<br/>

### 식별, 비식별 관계의 장단점

- 식별 관계의 단점
    - 부모 테이블의 키본키를 자식 테이블로 전파하면서, 자식 테이블의 기본키 칼럼이 점점 늘어난다.
        - 결국 조인할 때, SQL이 복잡해지고 기본키 인덱스가 불필요하게 커질 수 있다.
    - 2개 이상의 칼럼을 합해서 복합 기본키를 만들어야 하는 경우가 많다.
    - 비즈니스 의미가 있는 자연키 칼럼을 조합하여 기본키로 사용하는 경우가 많다.
        - 식별 관계의 자연키 칼럼들이 자식에 손자까지 전파되면 변경하기 힘들다.
        - 비즈니스 요구사항은 언제가는 바뀐다.
        - 반면에, 비식별 관계에서는 주로 대리키를 기본키로 사용한다.
    - 부모 테이블의 기본키를 자식 테이블의 기본키로 사용하므로, 비식별 관계보다 테이블 구조가 유연하지 못하다.
    - 일대일 관계를 제외한 식별 관계는 2개 이상의 칼럼을 묶은 복합 기본키를 사용한다.
        - 따라서, 칼럼이 하나인 기본키를 매핑하는 것보다 많은 노력이 필요하다.

<br/>

- 식별 관계의 장점
    - 기본 키 인덱스를 활용하기 좋다.
    - 상위 테이블들의 기본키 칼럼을 자식 및 손자 테이블들이 가지고 있으므로, 특정 상황에 조인 없이 하위 테이블만으로 검색을 완료할 수 있다.

<br/>

- 정리
    - **웬만하면 비식별 관계를 사용하자.**
    - **그리고 기본키로 Long 타입의 대리키를 사용하자.**
        - 비즈니스가 변경되어도 유연하게 대처할 수 있다.
        - `@GeneratedValue` 를 통해, 간편하게 대리키를 생성할 수 있다.
        - 식별자 칼럼이 하나여서 쉽게 매핑할 수 있다.
    - **'선택적 비식별 관계'보단 '필수적 비식별 관계'를 사용하자.**
        - 선택적 비식별 관계는 NULL을 허용하므로, 조인할 때 **외부조인**을 사용해야 한다.
        - 필수적 비식별 관계는 NULL을 허용하지 않으므로, 조인할 때 **내부조인만**을 사용할 수 있다.
        
        > 외부조인: 연관되지 않은 것까지 포함하여 출력  
        내부조인: 연관되지 않은 것은 제외하고 출력

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>