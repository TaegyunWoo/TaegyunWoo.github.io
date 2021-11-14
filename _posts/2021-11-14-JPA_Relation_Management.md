---
category: JPA
tags: [JPA]
title: "[JPA] 영속성 전이와 고아 객체"
date:   2021-11-14 19:00:00 
lastmod : 2021-11-14 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 다룰 내용

이번 포스팅에서 다룰 내용은 아래와 같다.

- **영속성 전이, 고아 객체**
    - 연관된 객체를 함께 저장하거나 함께 삭제할 수 있다.

하나씩 알아보자.

<br/><br/><br/>

# 영속성 전이

## CASCADE

### 영속성 전이란?

- 특정 엔티티를 영속 상태로 만들 때, 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용하는 기능이다.
- JPA는 `CASCADE` 옵션으로 영속성 전이를 제공한다.
- 예시를 통해, 기존의 방식과 무엇이 다른지 알아보자.

<br/><br/>

## 기본 예시

### 연관관계 설정 기존 방식

- 엔티티 관계
    
    ![Untitled](/assets/img/2021-11-14-JPA_Relation_Management/Untitled%2050.png)

<br/>

- 부모 엔티티
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	@OneToMany(mappedBy = "parent")
    	private List<Child> children = new ArrayList<Child>();
    
    }
    ```

<br/>

- 부모 자식 저장 로직
    
    ```java
    private static void saveNoCascade(EntityManager em) {
    
    	//부모 저장
    	Parent parent = new Parent();
    	em.persist(parent); //부모를 먼저 저장해야한다. (영속 상태로 변경)
    
    	//1번 자식 저장
    	Child child1 = new Child();
    	child1.setParent(parent); //자식 -> 부모 연관관계 설정
    	parent.getChildren().add(child1); //부모 -> 자식 연관관계 설정
    	em.persist(child1); //자식1 저장 (영속 상태로 변경)
    
    	//2번 자식 저장
    	Child child2 = new Child();
    	child2.setParent(parent); //자식 -> 부모 연관관계 설정
    	parent.getChildren().add(child2); //부모 -> 자식 연관관계 설정
    	em.persist(child2); //자식2 저장 (영속 상태로 변경)
    }
    ```
    
    - `em.persist(parent)`
        - 부모 엔티티를 먼저 저장하고, 영속 상태로 만들었다.
        - 다른 엔티티에서 `parent` 엔티티가 연관관계로 사용되려면, 영속 상태이어야 한다.
    - `em.persist(child1)` , `em.persist(child2)`
        - 각각 자식 엔티티를 영속상태로 만듦과 동시에 저장한다.
        - *너무 번거롭다...*

- **이때 영속성 전이를 사용하면, 부모 엔티티만 영속 상태로 만들어도 연관된 자식 엔티티까지 영속 상태로 만들 수 있다.**

<br/><br/>

## 영속성 전이: 저장

### 영속성 전이를 적용하여 저장하기

- 위 예시 코드를 아래와 같이 수정하여, 영속성 전이 기능을 적용하자.
- 부모 엔티티
    
    ```java
    @Entity
    public class Parent {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	//영속성 전이 설정
    	@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    	private List<Child> children = new ArrayList<Child>();
    
    }
    ```
    
    - `cascade = CascadeType.PERSIST`
        - 위와 같이 설정 시, 부모를 영속화할 때 연관된 자식들도 함께 영속화하라고 할 수 있다.
- 부모 자식 저장 로직
    
    ```java
    private static void saveCascade(EntityManager em) {
    
    	//부모 저장
    	Parent parent = new Parent();
    	//em.persist(parent); 부모를 먼저 저장할 필요가 없다.
    
    	//1번 자식 저장
    	Child child1 = new Child();
    	child1.setParent(parent);
    	parent.getChildren().add(child1);
    	//em.persist(child1); 필요없는 코드이다.
    
    	//2번 자식 저장
    	Child child2 = new Child();
    	child2.setParent(parent);
    	parent.getChildren().add(child2);
    	//em.persist(child2); 필요없는 코드이다.
    
    	//부모 엔티티 영속화 및 저장
    	em.persist(parent); //이때, 연관된 자식까지 영속화하고 저장한다.
    }
    ```
    
    - `em.persist(parent);` 호출시
        - 먼저 parent를 영속화한 뒤, child를 영속화한다. 그 다음, 각 엔티티를 저장한다.
        - 따라서 최종적인 연관관계 설정이 해당 코드 한줄로 끝난다.

<br/>

### 영속성 전이 저장 시각화

![Untitled](/assets/img/2021-11-14-JPA_Relation_Management/Untitled%2051.png)

- **즉, 부모만 영속화하면 자식 엔티티까지 함께 영속화해서 저장한다.**

<br/><br/>

## 영속성 전이 특징

- 영속성 전이는 연관관계를 매핑하는 것과는 관련이 없다.
- **영속성 전이는 영속화할 때, 연관된 엔티티도 같이 영속화하는 편리함을 제공할 뿐이다.**

<br/><br/>

## 영속성 전이: 삭제

### 영속성 전이를 적용하여 삭제하기

- 부모·자식 엔티티를 모두 제거하기 위한 기존 코드
    
    ```java
    Parent findParent = em.find(Parent.class, 1L);
    Child findChild1 = em.find(Child.class, 1L);
    Child findChild2 = em.find(Child.class, 2L);
    
    em.remove(findChild1);
    em.remove(findChild2);
    em.remove(parent);
    ```
    
    - 기존 방식은 위와 같이 각각 하나씩 제거해야 한다.
    - 외래키 제약조건을 고려하여, 연관된 자식 엔티티부터 삭제해야 한다.

<br/>

- 영속성 전이를 사용하여, 부모·자식 엔티티를 모두 제거하기
    - 먼저 부모 엔티티에 `CascadeType.REMOVE` 를 설정한다.
    
    ```java
    Parent findParent = em.find(Parent.class, 1L);
    //Child findChild1 = em.find(Child.class, 1L); 필요없는 코드
    //Child findChild2 = em.find(Child.class, 2L);
    
    //em.remove(findChild1); 필요없는 코드
    //em.remove(findChild2);
    em.remove(parent);
    ```
    
    - `em.remove(parent);`
        - 먼저 연관된 Child 엔티티를 모두 삭제한다.
        - 그 후, findParent를 삭제한다.

<br/>

### 삭제 순서

- **연관 엔티티 삭제 시, 외래키 제약조건을 고려해야 한다.**
- 삭제 순서
    1. 자식 삭제 (FK를 갖는 엔티티 먼저 삭제)
    2. 부모 삭제 (FK에 들어가는 엔티티 삭제)
- **위와 같이 삭제해야, 외래키 무결성을 위반하는 경우를 방지할 수 있다.**

<br/><br/>

## CASCADE 종류

### 종류

```java
public enum CascadeType {
	ALL, //모두 적용
	PERSIST, //영속
	MERGE, //병합
	REMOVE, //삭제
	REFRESH, //REFRESH
	DETACH //DETACH
}
```

<br/>

### 여러 속성 적용하기

- 여러 속성을 동시에 적용할 수도 있다.
- 예시
    
    ```java
    cascade = {CascadeType.PERSIST, CascadeType.REMOVE}
    ```
    
<br/>

### 참고사항

- `CascadeType.PERSIST` 와 `CascadeType.REMOVE`
    - 위 두 가지 경우는 `em.persist()` , `em.remove()` 를 실행할 때 바로 전이되지 않는다.
    - **플러시를 호출할 때 전이된다.**

<br/><br/><br/>

# 고아 객체

## 개요

### 고아 객체란?

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티

<br/>

### 고아 객체 제거 기능

- JPA는 고아 객체를 자동으로 삭제하는 기능을 제공한다.
- **해당 기능을 통해, 부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자식 엔티티가 자동으로 삭제되게 할 수 있다.**

<br/><br/>

## 예시

### 고아 객체 제거 기능 설정

```java
@Entity
public class Parent {

	@Id @GeneratedValue
	private Long id;

	//고아 객체 자동 제거 설정
	@OneToMany(mappedBy = "parent", orphanRemoval = true)
	private List<Child> children = new ArrayList<Child>();

}
```

- `orphanRemoval = true`
    - 해당 속성을 설정하여 고아 객체 제거 기능을 활성화했다.
    - 이제 컬렉션에서 제거한 엔티티는 자동으로 삭제된다.

<br/>

### 사용 예시

```java
Parent parent1 = em.find(Parent.class, id);

//ArrayList에 저장된 첫번째 연관 엔티티를 컬렉션에서 삭제
parent1.getChildren().remove(0);
```

- **위와 같이, 컬렉션에서 연관 엔티티를 제거하기만 해도 DB의 데이터가 삭제된다.**
- **이 기능은 영속성 컨텍스트를 플러시할 때 적용되므로, 플러시 시점에 DELETE SQL 이 실행된다.**

<br/>

- 모든 자식 엔티티 제거 예시
    
    ```java
    parent1.getChildren().clear(); //ArrayList 비우기
    ```
    
<br/>

### 주의사항

- **참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 판단하여 삭제한다.**
    - 따라서 이 기능은 '특정 엔티티가 개인 소유하는 엔티티'에만 적용해야 한다.
    - 그래서 `orphanRemovel` 속성은 `@OneToOne` , `@OneToMany` 에만 사용할 수 있다.

- **고아 객체 제거 기능: 부모를 먼저 제거하면, 자식도 같이 제거된다.**
    - `CascadeType.REMOVE` 와 동일하다.

<br/><br/>

## 영속성 전이 + 고아 객체: 생명주기

만약 `CascadeType.ALL` 과 `orphanRemoval = true` 를 동시에 사용하면 어떻게 될까?

<br/>

### 생명주기: 영속성 전이, 고아객체 제거 기능을 사용하지 않을 때

- 일반적으로 `EntityManager.persist()` 를 통해 엔티티를 영속화하고, `EntityManager.remove()` 를 통해 엔티티를 제거한다.
    - 직접 자식엔티티가 `persist()` , `remove()` 한다.
- **즉, 엔티티 스스로 생명주기를 관리한다는 것과 같다.**

<br/>

### 생명주기: 영속성 전이, 고아객체 제거 기능을 모두 사용할 때

- 부모 엔티티를 통해서 자식의 생명주기가 관리된다.
    - 부모 엔티티에서만 `persist()`, `remove()` 해도, 자식 엔티티까지 처리된다.
- 예시: 자식을 저장하려면 부모에 등록만하면 된다.
    
    ```java
    Parent parent = em.find(Parent.class, parentId);
    parent.addChild(child1);
    ```
    
- 예시: 자식을 삭제하려면 부모에 제거만하면 된다.
    
    ```java
    Parent parent = em.find(Parent.class, parentId);
    parent.getChildren().remove(removeObj);
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