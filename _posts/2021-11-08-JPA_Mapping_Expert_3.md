---
category: JPA
tags: [JPA]
title: "[JPA] 고급매핑 (3)"
date:   2021-11-08 17:20:00 
lastmod : 2021-11-08 17:20:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    - [[JPA] 고급매핑 (1)](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_1)
    - [[JPA] 고급매핑 (2)](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_2)

<br/><br/>

# 이번 포스팅에서 다룰 내용

[이전 포스팅1](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_1)에서 다룬 고급 매핑 내용은 아래와 같다.

- 상속 관계 매핑
- `@MappedSuperclass`

[이전 포스팅2](https://taegyunwoo.github.io/jpa/JPA_Mapping_Expert_2)에서 다룬 고급 매핑 내용은 아래와 같다. 

- 복합 키와 식별 관계 매핑

아래는 이번 포스팅에서 다룰 고급 매핑 내용이다.

- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑하기

<br/><br/><br/>

# 조인 테이블

## 개요

### DB 테이블 연관관계 설계 방법

DB 테이블의 연관관계를 설계하는 방법은 크게 2가지이다.

- **조인 칼럼을 사용하는 방법 (외래키 사용)**
- **조인 테이블을 사용하는 방법 (테이블 사용)**

이에 대해 하나씩 설명하겠다.

<br/>

### 조인 칼럼을 사용하는 방법

- 테이블 간의 관계는 주로 조인 칼럼이라 부르는 외래키 칼럼을 사용해서 관리한다.

- 조인 칼럼을 사용한 테이블 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2033.png)
    
    - `LOCKER_ID` 에 NULL을 허용한다.

- 조인 칼럼 데이터 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2034.png)
    

- 외래키에 NULL을 허용한다. ⇒ 이것을 **선택적 비식별 관계**라고 한다.
- 외래키에 NULL이 오는것을 허용하므로, **회원과 사물함을 조인할 때 외부조인을 사용**해야 한다.
- 회원과 사물함이 아주 가끔 관계를 맺는다면 외래키 값 대부분이 NULL로 저장되는 단점도 존재한다.

<br/>

### 조인 테이블을 사용하는 방법

- 조인 테이블을 사용한 테이블 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2035.png)
    

- 조인 테이블 데이터 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2036.png)
    

- 이 방법은 조인 테이블이라는 별도의 테이블을 사용해서 연관관계를 관리한다.
- 조인 칼럼을 사용하는 방법
    - 단순히 외래키 칼럼만 추가해서 연관관계를 맺는다.
- 조인 테이블을 사용하는 방법
    - 연관관계를 관리하는 조인 테이블을 추가하고, 여기서 두 테이블의 외래키를 가지고 연관관계를 관리한다.
        - **따라서 `MEMBER`와 `LOCKER` 테이블에는 외래키 칼럼이 없다!**
    - 단점은 테이블을 하나 추가해야 한다는 점이다.
- **따라서 기본적으로 조인 칼럼을 사용하되, 필요하다고 판단되면 조인 테이블을 사용하는 것이 좋다.**

<br/>

### 앞으로 설명할 내용: 조인 테이블

조인 테이블에 대해 설명할 내용은 다음과 같다.

- 객체와 테이블을 매핑할 때
    - 조인 칼럼은 `@JoinColumn` 으로 매핑한다.
    - 조인 테이블은 `@JoinTable` 로 매핑한다.
- 조인 테이블은 주로 다대다 관계를 일대다·다대일 관계로 풀어낼 때 사용된다.
    - 하지만 일대일, 일대다, 다대일 관계에서도 사용된다.
    - 이 경우에 대해 알아보자.

이제 조인 테이블에 대해 하나씩 알아보자.

<br/><br/>

## 일대일 조인 테이블

### 가정

- 일대일 관계를 만들려면 **조인 테이블의 외래키 칼럼 각각에 유니크 제약조건을 걸어야 한다.**
    - 유니크 제약조건: 중복값 허용 X
- 일대일 조인 테이블 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2037.png)
    
    - `PARENT_CHILD.PARENT_ID` 는 PK로 사용되므로 유니크 제약조건이 자동으로 걸려있다.
- 예시 코드를 통해, 일대일 조인 테이블을 어떻게 매핑하는지 알아보자.

<br/>

### 예시 코드

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    
    	private String name;
    
    	@OneToOne //mapped 속성을 사용하지 않는다. (단방향)
    	@JoinTable(
    		name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "PARENT_ID"),
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID"),
    	)
    	private Child child;
    
    	// getter, setter 생략
    }
    ```
    
    - `@JoinTable` 속성
        - name : 매핑할 조인 테이블의 이름
        - joinColumns : 현재 엔티티를 참조하는 외래키
        - inverseJoinColumns : 반대방향 엔티티를 참조하는 외래키

<br/>

- **자식 클래스**
    
    ```java
    @Entity
    public class Child {
    
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- 양방향으로 매핑하려면 다음 코드를 추가하면 된다.
    - 조인테이블의 경우 `PARENT` 테이블이 FK를 갖지 않지만, 연관관계의 주인이 될 수 있다.
    
    ```java
    @Entity
    public class Child {
    	// ...
    
    	@OneToOne(mappedBy = "child")
    	private Parent parent;
    
    	// ...
    }
    ```
    
<br/><br/>

## 일대다 조인 테이블

### 가정

- 일대다 관계를 만들려면, **조인 테이블의 칼럼 중 '다(N)와 관련된 칼럼 `CHILD_ID` '에 유니크 제약조건**을 걸어야 한다.
- 일대다 단방향 관계에 대해 매핑하는 방법을 알아보자.

- 조인 테이블을 사용한 일대다 관계 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2038.png)
    
    - `PARENT_CHILD.CHILD_ID` 가 PK이므로 자동으로 유니크 제약조건이 걸린다.

- 바로 예시코드를 통해, 일대다 단방향 관계를 조인 테이블로 표현했을 때 어떻게 매핑해야하는지 알아보자.

<br/>

### 예시 코드

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    
    	private String name;
    
    	@OneToMany
    	@JoinTable(
    		name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "PARENT_ID"),
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID"),
    	)
    	private List<Child> childs = new ArrayList<Child>();
    
    	// getter, setter 생략
    }
    ```
    
<br/>

- **자식 클래스**
    
    > 단방향이라면 자식 클래스는 변경사항이 없다.
    > 
    
    ```java
    @Entity
    public class Child {
    
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    
<br/><br/>

## 다대일 조인 테이블

### 기본 설명

- 다대일은 일대다에서 방향만 반대인 것이다.
    - 연관관계의 주인만이 바뀐다.
- 바로 예시 코드를 통해, 조인 테이블을 사용한 다대일 양방향 관계를 알아보자.

<br/>

### 예시 코드

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    
    	private String name;
    
    	@ManyToOne(mappedBy = "parent")
    	private List<Child> childs = new ArrayList<Child>();
    
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
    
    	private String name;
    
    	@ManyToOne(optional = false) //null 허용 X
    	@JoinTable(
    		name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "CHILD_ID"),
    		inverseJoinColumns = @JoinColumn(name = "PARENT_ID"),
    	)
    	private Parent parent;
    
    	// getter, setter 생략
    }
    ```
    
<br/><br/>

## 다대다 조인 테이블

### 가정

- 다대다 관계를 만들려면, **조인 테이블의 두 칼럼을 합해서 하나의 복합 유니크 제약조건**을 걸어야 한다.
- 조인 테이블을 사용한 다대다 관계 시각화
    
    ![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2039.png)
    
    - `PARENT_CHILD` 테이블의 `PARENT_ID` 와 `CHILD_ID` 가 복합키이므로 복합 유니크 제약조건이 걸려있다.
- 바로 예시코드를 통해, 다대다 관계를 조인 테이블로 표현했을 때 어떻게 매핑해야하는지 알아보자.

<br/>

### 예시 코드

- **부모 클래스**
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    
    	private String name;
    
    	@ManyToMany
    	@JoinTable(
    		name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "PARENT_ID"),
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
    	)
    	private List<Child> childs = new ArrayList<Child>();
    
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
    
    	private String name;
    
    	// getter, setter 생략
    }
    ```
    

- **이때 조인 테이블에 컬럼을 추가하면, `@JoinTable` 전략을 사용할 수 없다. 이때는 새로운 엔티티를 만들어서 조인 테이블과 매핑해야 한다.**
    - [이전 게시글](https://taegyunwoo.github.io/jpa/JPA_Relation_Mid#24)을 참고하자.

<br/>

지금까지 조인 테이블을 사용하여 여러 관계를 매핑하는 방법에 다루었다. 이제 엔티티 하나에 여러 테이블을 매핑하는 방법에 대해 알아보자.

<br/><br/><br/>

# 엔티티 하나에 여러 테이블 매핑

## 개요

### `@SecondaryTable` 애너테이션

- `@SecondaryTable` 애너테이션을 사용하면 한 엔티티에 여러 테이블을 매핑할 수 있다.

> 잘 사용되지는 않는다.

<br/><br/>

## 여러 테이블 매핑 예시

### 시각화

- 아래 그림은 '하나의 엔티티가 여러 테이블과 매핑된 것'을 시각적으로 나타낸 것이다.

![Untitled](/assets/img/2021-11-08-JPA_Mapping_Expert_3/Untitled%2040.png)

<br/>

### 예시 코드

```java
@Entity
@Table(name = "BOARD")
@SecondaryTable(
	name = "BOARD_DETAIL",
	pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID")
)
public class Board {

	@Id @GeneratedValue
	@Column(name = "BOARD_ID")
	private Long id;

	private String title;

	@Column(table = "BOARD_DETAIL")
	private String content;

	// getter, setter 생략
}
```

- `@Table` 을 사용하여, 기본적인 테이블 `BOARD` 와 매핑했다.
- `@SecondaryTable` 을 추가로 사용하여, 추가 테이블 `BOARD_DETAIL` 과 매핑했다.
- content 필드는 `@Column(table = "BOARD_DETAIL")` 을 사용해서, `BOARD_DETAIL` 테이블의 칼럼에 매핑했다.
- title 필드는 테이블을 지정하지 않았다. ⇒ 기본 테이블인 `BOARD` 에 매핑된다.

- `@SecondaryTable` 속성
    - name
        - 매핑할 다른 테이블의 이름
        - 위 예제에서는 테이블명을 `BOARD_DETAIL` 로 지정했다.
    - pkJoinColumns
        - 매핑할 다른 테이블의 기본키 칼럼 속성 설정
        - 위 예제에서는 `BOARD_DETAIL_ID` 로 지정했다.

<br/>

### 권장사항

- `@SecondaryTable` 을 사용하여 여러 테이블을 하나의 엔티티와 매핑하는 것을 권장하지 않는다.
- **테이블당 엔티티를 각각 만들어서 일대일 매핑하는 것을 권장한다.**

<br/><br/><br/>

# 총정리

## 지금까지 다룬 내용

지금까지 총 3번에 걸쳐 '고급매핑'에 대해 다뤘다. 지금까지 다룬 내용은 다음과 같다.

- **객체의 상속 관계를 DB에 매핑하는 방법**
- **매핑 정보만 상속하는 `@MappedSuperclass`**
- **복합키를 사용하는 식별 관계와 비식별 관계**
- **조인 테이블**
- **엔티티 하나에 여러 테이블을 매핑하는 방법**

<br/>

## 자주 사용되는 내용

- **상속 관계 매핑**
- **`@MappedSuperclass`**

사실 실제 프로젝트에선 위 두가지가 가장 많이 사용된다. 나머지는 사용되지는 않는다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>