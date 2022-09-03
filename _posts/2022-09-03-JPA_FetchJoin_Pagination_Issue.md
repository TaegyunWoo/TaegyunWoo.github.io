---
category: JPA
tags: [JPA, Hibernate]
title: "[JPA] 페치조인과 페이지네이션 사용시 주의사항"
date:   2022-09-03 22:10:00 
lastmod : 2022-09-03 22:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

## 개요

최근 프로젝트를 진행하면서, `Fetch Join` 과 `Pagination` 을 함께 사용했다.
이때 발생한 성능 저하 및 자원 낭비 이슈가 발생했는데, 이에 대해 포스팅을 하고자 한다.

<br/>

## 이슈 설명

- 특정 엔티티가 컬렉션 필드를 가지고 있을 때, 즉 1:N 관계일 때 `Fetch Join` 과 `Pagination` 을 사용하여 문제가 발생했다.
- 아래는 동일한 문제가 발생하는 예시 코드이다.

### 엔티티A 클래스
```java
public class EntityA {
  @Id @GeneratedValue
  private Long id;
  
  @OneToMany(mappedBy = "entityA")
  private List<EntityB> entityBList = new ArrayList<>();
}
```

### 엔티티B 클래스
```java
public class EntityB {
  @Id @GeneratedValue
  private Long id;
  
  @ManyToOne
  private EntityA entityA;
}
```
> 엔티티B : 엔티티A = N : 1

### Repository 클래스
```java
@Repository
public class Repository {
  @PersistenceContext
  private EntityManager em;
  
  public List<EntityA> findAll(int startIndex, int size) {
    String jpql = "SELECT a FROM EntityA a" +
                  " JOIN FETCH a.entityBList";

    return em.createQuery(jpql, EntityA.class)
              .setFirstResult(startIndex)
              .setMaxResults(size)
              .getResultList();
  }
}
```

### 이슈 발생 절차

1. `EntityA a1` 엔티티의 `entityBList` 필드에 `Entity b1` , `Entity b2` , `Entity b3` 를 Add 한다.

2. `EntityA a1` 엔티티를 DB에 저장한다.

3. `EntityA a2` 엔티티를 DB에 저장한다.

4. `Repository.findAll(0, 2)` 를 호출한다.

5. `firstResult/maxResults specified with collection fetch; applying in memory!` 라는 문구가 콘솔에 출력되지만, 정상적으로 `List<EntityA>` 가 반환된 것 같이 보인다.

6. 하지만 여기에는 심각한 성능 저하 문제가 있다.

### 원하는 결과

`firstResult/maxResults specified with collection fetch; applying in memory!` 라는 경고 메시지를 없애고, 성능저하 문제를 해결한다.

<br/>

## 문제 분석

### 발생 원인

- **`N:1` 관계에서 `Fetch Join` 과 `Pagination API` 를 함께 사용할 때 조회하는 대상이 `1`이라면, `firstResult/maxResults specified with collection fetch; applying in memory!` 가 발생한다.**
- 위와 같이 `Repository.findAll(0, 2)` 를 호출했을 때, 실행되는 실제 SQL 은 아래와 같다.

```sql
SELECT A.ID FROM EntityA as A
  INNER JOIN EntityB as B
  ON A.ID = B.EntityA_ID;
```

- **`LIMIT` 이나 `OFFSET` 절을 전혀 사용하지 않았음을 알 수 있다.**  
왜 그렇게 동작하는지 계속해서 알아보자.

- 만약 위 이슈 발생 절차대로 작업을 한다면 실제 조회 결과는 아래와 같다.

|A.ID|B.ID|B.EntityA_ID|
|----|-----|------------|
|1|1|1|
|1|2|1|
|1|3|1|

- **일반 `Fetch Join` 으로 조회를 하게 되면 `INNER JOIN`으로 쿼리를 실행하고, 동일한 `EntityA` 가 연관된 `EntityB` 의 개수만큼 중복되어 출력된다.**
- 따라서 정확하게 Pagination 을 할 수 없다.
- 그렇다면 왜 반환값 `List<EntityA>` 가 정확하게 기대하는 바와 같이 반환되었을까?
  - **그 이유는 JPA 가 모든 EntityA 를 INNER JOIN 으로 조회하고, 서버 메모리를 사용하여 직접 페이징 처리를 했기 때문이다.**
  - 따라서 성능 저하가 발생하고, 자원을 낭비하게 된다.

<br/>

## 해결방법
해결 방법에는 크게 두 가지가 있다.

1. 하나의 쿼리를 두 개로 쪼개는 방법
2. 1 -> N 으로 조회하지 않고, N -> 1 로 조회하는 방법

하나씩 설명하겠다.

### 1. 하나의 쿼리를 두 개로 쪼개기

기존 `Repository.findAll` 메서드의 로직은 아래와 같다.

```java
public List<EntityA> findAll(int startIndex, int size) {
  String jpql = "SELECT a FROM EntityA a" +
                " JOIN FETCH a.entityBList";

  return em.createQuery(jpql, EntityA.class)
            .setFirstResult(startIndex)
            .setMaxResults(size)
            .getResultList();
}
```

이것을 아래처럼 변경하여 문제를 해결할 수 있다.

```java
public List<EntityA> findAll(int startIndex, int size) {
  //먼저 EntityA 의 ID를 Pagination하여 조회한다.
  String jpql = "SELECT a.id FROM EntityA a" +
                " JOIN FETCH a.entityBList";
  List<Long> idResult = em.createQuery(jpql, Long.class)
                   .setFirstResult(startIndex)
                   .setMaxResults(size)
                   .getResultList();
  
  //추출된 ID를 통해, 원하는 결과를 얻는다.
  jpql = "SELECT a FROM EntityA a" +
          " JOIN FETCH a.entityBList" +
          " WHERE a.id IN (:ids)";
  List<EntityA> finalResult = em.createQuery(jpql, EntityA.class)
                    .setParameter("ids", idResult)
                    .getResultList();
  
  return finalResult;
}
```

### 2. 1 -> N 으로 조회하지 않고, N -> 1 로 조회하기

애초에 EntityB 에서 조회를 한다면 문제가 될 것이 없다.

아래는 그 예시 코드이다.

```java
public List<EntityB> findAll(int startIndex, int size) {
  String jpql = "SELECT b FROM EntityB b" +
                " JOIN FETCH b.entityA";

  return em.createQuery(jpql, EntityA.class)
            .setFirstResult(startIndex)
            .setMaxResults(size)
            .getResultList();
}
```

- 여기에서도 `Fetch Join` 과 `Pagination API` 를 사용한다.

- **하지만 해당 메서드를 실행했을 때, `firstResult/maxResults specified with collection fetch; applying in memory!` 메시지는 출력되지 않는다!**

- 그 이유는 성능상 크게 문제되는 부분이 없기 때문이다.

- 해당 메서드를 호출했을 때, 실제로 실행되는 SQL 쿼리는 아래와 같다.

```sql
SELECT B.ID, B.EntityA_ID FROM EntityB as B
  INNER JOIN EntityA as A
  ON B.EntityA_ID = A.ID;
```

- 그 결과는 아래와 같다.

|B.ID|B.EntityA_ID|A.ID|
|----|------------|----|
|1|1|1|
|2|1|1|
|3|1|1|

- 결과를 보면 `Pagination` 을 했을 때 전혀 문제가 될 것이 없다.

- **왜냐하면 어차피 `Pagination` 의 대상이 `EntityB` 이고, 결과적으로 중복되는 `EntityB` 는 없기 때문이다.**

> 물론 EntityA 는 중복된다.

## 정리

- 나의 경우, 첫번째 방법이 더 적합했다.
- 왜냐하면 `EntityA` 를 깔끔하게 구하고 싶었기 때문이다.
  - 두 번째 방법을 사용하면, `EntityA` 를 중복되지 않게 구하기 위한 로직이 별도로 필요하다.