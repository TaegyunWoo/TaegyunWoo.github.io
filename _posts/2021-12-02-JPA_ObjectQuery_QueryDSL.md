---
category: JPA
tags: [JPA]
title: "[JPA] 객체지향 쿼리 언어 - QueryDSL"
date:   2021-12-02 00:00:00 
lastmod : 2021-12-02 00:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [[JPA] 객체지향 쿼리 언어 - 소개](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_Begin)
    2. [[JPA] 객체지향 쿼리 언어 - JPQL 기초](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin)
    3. [[JPA] 객체지향 쿼리 언어 - JPQL 조인](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Join)
    4. [[JPA] 객체지향 쿼리 언어 - JPQL 심화](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Expert)

<br/><br/>

# QueryDSL

## 개요

### QueryDSL 이란?

- Criteria는 너무 복잡하고 어렵다.
- QueryDSL도 Criteria처럼 JPQL 빌더 역할을 수행하지만, 더욱 간결하고 쉽다.
- QueryDSL은 오픈소스 프로젝트이다.

<br/><br/>

## QueryDSL 기초

### 쿼리 타입(Q)

- 쿼리 타입이란?
    - QueryDSL을 사용하려면, 엔티티를 기반으로 쿼리 타입이라는 **쿼리용 클래스**를 생성해야 한다.
    - **바로 이, 쿼리용 클래스가 쿼리 타입이다.**

<br/>

### 시작하기

예제를 통해, QueryDSL을 어떻게 사용해야하는 건지 알아보자.

```java
public void queryDSL() {

	EntityManager em = emf.createEntityManager();

	// 1. QueryDSL 사용을 위한 객체 생성
	JPAQuery query = new JPAQuery(em);

	// 2. 쿼리타입(Q) 생성
	QMember qMember = new QMember("m"); //생성되는 JPQL의 별칭을 m으로 설정

	List<Member> members = query.from(qMember)
		.where(qMember.name.eq("회원1"))
		.orderBy(qMember.name.desc())
		.list(qMember);
}
```

1. `com.mysema.query.jpa.impl.JPAQuery` 객체 생성한다.
    - 이때 생성자를 통해, `EntityManager` 객체를 넘겨준다.
2. 사용할 쿼리 타입(Q)를 생성한다.
    - 이때 생성자를 통해, 엔티티의 별칭을 설정한다.

- 결과 SQL
    
    ```sql
    SELECT m FROM Member m
    WHERE m.name = ?1
    ORDER BY m.name DESC
    ```
    
<br/>

### 기본 Q 생성

- 쿼리 타입은 사용하기 편리하도록 아래와 같이 **기본 인스턴스를 보관**하고 있다.
    
    ```java
    public class QMember extends EntityPathBase<Member> {
    
    	public static final QMember member = new QMember("member1");
    
    }
    ```
    
    - **따라서 QMember 객체를 new 연산자로 생성하지 않고,  `QMember.member` 로 접근하여 QMember 객체를 사용할 수 있다.**
- 예시 코드
    
    ```java
    QMember qMember = new QMember("m"); //직접 지정
    QMember qMember = QMember.member; //기본 인스턴스 사용
    ```
    
<br/>

- `import static` 을 통해 기본 인스턴스 사용 시, 코드가 더욱 간결해진다.
    
    ```java
    import static jpabook.jpashop.domain.QMember.member; // 기본 인스턴스
    
    public void basic() {
    	EntityManager em = emf.createEntityManager();
    
    	//필요없는 코드
    	// QMember member = new QMember("member1");
    
    	JPAQuery query = new JPAQuery(em);
    	List<Member> members = query.from(member)
    			.where(member.name.eq("회원1"))
    			.orderBy(member.name.desc())
    			.list(member);
    }
    ```
    
<br/><br/>

## 검색 조건 쿼리

### QueryDSL 코드

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
		.where(item.name.eq("좋은상품").and(item.price.gt(20000)))
		.list(item); //조회할 프로젝션 지정
```

<br/>

### 변환된 JPQL

```xml
select i From Item i
where i.name=?1 and i.price > ?2
```

<br/>

### 상세설명

- QueryDSL의 where절에 `and` 나 `or`을 사용할 수 있다.
- 아래 코드처럼 간편하게 `and` 연산을 할수도 있다.
    
    ```xml
    .where(item.name.eq("좋은상품"), item.price.gt(20000))
    ```
    

- `where()` 에서 사용되는 주요 메서드
    - `item.price.between(10000, 20000);`
        - 가격이 10000원 ~ 20000원인 상품
    - `item.name.contains("상품1");`
        - 상품1이라는 이름을 포함한 상품
        - SQL: `%상품1%`
    - `item.name.startsWith("고급");`
        - 이름이 고급으로 시작하는 상품
        - SQL: `고급%`
    
    > 이외에도 여러가지 메서드가 존재한다. 자세한 것은 IDE의 도움을 받아 찾아보자.
    > 

<br/><br/>

## 결과 조회

### QueryDSL 코드

- **쿼리 작성이 끝나고, 결과 조회 메서드를 호출하면 실제 DB를 조회한다.**
    
    ```java
    List<Item> list = query.from(item)
    		.where(item.name.eq("좋은상품").and(item.price.gt(20000)))
    		.list(item); //이때 실제 DB 조회
    ```
    
<br/>

### 대표 결과 조회 메서드

- `uniqueResult(프로젝션)`
    - 조회 결과가 한 건일 때 사용하는 메서드
    - 조회 결과가 없음: null 반환
    - 조회 결과가 하나 이상: `com.mysema.query.NonUniqueResultException` 예외 발생
- `singleResult(프로젝션)`
    - 결과가 하나 이상이면 처음 데이터를 반환한다.
- `list(프로젝션)`
    - 결과가 하나 이상일 때 사용한다.
    - 결과가 없으면 빈 컬렉션을 반환한다.

<br/><br/>

## 페이징과 정렬

### QueryDSL 코드

```java
QItem item = QItem.item;
query.from(item)
	.where(item.price.gt(20000))
	.orderBy(item.price.desc(), item.stockQuantity.asc()) //정렬
	.offset(10).limit(20) //페이징
	.list(item);
```

- `.orderBy(item.price.desc(), item.stockQuantity.asc())`
    - price를 내림차순으로 정렬한다.
    - price가 동일할 때, stockQuantity를 오름차순으로 정렬한다.
- `.offset(10).limit(20)`
    - 0번부터 시작하는 index에서, 10번째 레코드부터 총 20개의 레코드를 검색한다.

<br/>

### 또다른 페이징 방법

```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L);

List<Item> list = query.from(item)
	.restrict(queryModifiers)
	.list(item);
```

- `QueryModifiers`
    - 해당 객체를 통해 페이징을 할 수 있다.
    - `new QueryModifiers(Long 검색_개수, Long 시작_인덱스)`
- `.restrict(QueryModifiers_객체)`
    - 페이징 정보가 담긴 객체를 전달하여 적용한다.

<br/>

### 검색된 전체 데이터 수 확인하기

- 실제 페이징 처리를 위해선, 검색된 전체 데이터 수를 알아야 한다.
- 이땐, `list()` 대신 `listResults()` 를 사용한다.

```java
SearchResults<Item> result = query.from(item)
		.where(item.price.gt(10000))
		.offset(10).limit(20)
		.listResults(item); //list() 대신 listResults()를 사용한다.

long total = result.getTotal(); // 검색된 전체 데이터 수
long limit = result.getLimit(); // Limit 값
long offset = result.getOffset(); // Offset 값

List<Item> results = result.getResults(); //페이징 적용 결과
```

- `listResults()` 를 사용하면, 전체 데이터 조회를 위한 `count쿼리`를 한번 더 실행한다.
- 그리고 그 결과로 `SearchResults` 를 반환한다. 이 객체에서 전체 데이터 수를 조회할 수 있다.

<br/><br/>

## 그룹

### QueryDSL 코드

```java
query.from(item)
	.groupBy(item.price) //같은 price끼리 그룹핑
	.having(item.price.gt(1000)) //그룹핑 결과에서 1000보다 큰 그룹만
	.list(item);
```

<br/><br/>

## 조인

### QueryDSL에서 사용가능한 조인 종류

- `innerJoin (join)`
- `leftJoin`
- `rightJoin`
- `fullJoin`

- 기타
    - `on` 절
    - `fetch` 조인

<br/>

### 조인 기본 문법

- 첫 번째 파라미터
    - 조인 대상 지정
- 두 번째 파라미터
    - 별칭으로 사용할 쿼리 타입 지정

`join(조인대상, 별칭으로_사용할_쿼리타입)`

<br/>

### QueryDSL 코드: 기본 조인

```java
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
	.join(order.member, member)
	.leftJoin(order.orderItems, orderItem)
	.list(order);
```

<br/>

### QueryDSL 코드: 조인 + ON

```java
QOrder order = QOrder.order;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
	.leftJoin(order.orderItems, orderItem)
	.on(orderItem.count.gt(2))
	.list(order);
```

<br/>

### QueryDSL 코드: 페치 조인

```java
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
	.innerJoin(order.member, member).fetch()
	.leftJoin(order.orderItems, orderItem).fetch()
	.list(order);
```

<br/>

### QueryDSL 코드: 세타 조인

```java
QOrder order = QOrder.order;
QMember member = QMember.member;

query.from(order, member)
	.where(order.member.eq(member))
	.list(order);
```

<br/><br/>

## 서브 쿼리

### QueryDSL 서브쿼리

- 서브 쿼리는 `com.mysema.query.jpa.JPASubQuery` 를 생성해서 사용한다.
- 서브 쿼리의 결과가 하나면 `unique()`, 여러 건이면 `list()` 를 사용할 수 있다.

<br/>

### QueryDSL 코드: 한 건

```java
QItem item = QItem.item; //주 쿼리에서 사용될 쿼리 타입 객체
QItem.itemSub = new QItem("itemSub"); //서브 쿼리에서 사용될 쿼리 타입 객체

query.from(item)
	.where(item.price.eq(
		new JPASubQuery().from(itemSub).unique(itemSub.price.max()) //서브쿼리
	))
	.list(item);
```

<br/>

### QueryDSL 코드: 여러 건

```java
QItem item = QItem.item; //주 쿼리에서 사용될 쿼리 타입 객체
QItem.itemSub = new QItem("itemSub"); //서브 쿼리에서 사용될 쿼리 타입 객체

query.from(item)
	.where(item.in(
		new JPASubQuery().from(itemSub).
			.where(item.name.eq(itemSub.name))
			.list(itemSub) //서브쿼리
	))
	.list(item);
```

<br/><br/>

## 프로젝션과 결과 반환

### 프로젝션이란?

- 프로젝션은 `select` 절에 조회 대상을 지정하는 것을 말한다.
    
    > 자세한 것은 [이전 게시글 참고](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin#14)
    > 

<br/>

### QueryDSL 코드: 프로젝션 대상이 하나인 경우

```java
QItem item = QItem.item;

List<String> result = query.from(item).list(item.name);
//프로젝션=item.name

for (String name : result) {
	System.out.println("name = " + name)
}
```

<br/>

### QueryDSL 코드: 여러 칼럼 반환과 튜플

```java
QItem item = QItem.item;

//List<Tuple> result = query.from(item).list(new QTuple(item.name, item.price));
List<Tuple> result = query.from(item).list(item.name, item.price); // 위 코드와 같다.
//프로젝션 = item.name , item.price

for (Tuple tuple : result) {
	System.out.println("name = " + tuple.get(item.name));
	System.out.println("price = " + tuple.get(item.price));
}
```

- 여러 칼럼을 반환할 땐, Tuple 객체를 사용한다.

<br/>

### QueryDSL 코드: 빈(bean) 생성

- **빈 생성 기능**
    - 쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶으면, 빈 생성 (Bean Population) 기능을 사용하면 된다.
        
        > JPQL에서 다룬 NEW 연산자와 유사하다.  
        자세한 것은 [이전 게시글을 참고](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Begin#20)하자.
        > 
    - QueryDSL의 객체(빈) 생성 방법
        - **프로퍼티 접근**
        - **필드 직접 접근**
        - **생성자 사용**
    - 위 방법 중, 원하는 방법을 지정하기 위해 아래 객체를 사용하면 된다.
        - `com.mysema.query.types.Projections`

<br/>

- 예시 코드: ItemDTO 에 값 채우기
    - ItemDTO 클래스
        
        ```java
        public class ItemDTO {
        
        	private String username;
        	private int price;
        
        	public ItemDTO() {}
        
        	public ItemDTO(String username, int price) {
        		this.username = username;
        		this.price = price;
        	}
        
        	//Getter, Setter
        	public String getUsername() {...}
        	public void setUsername(String username) {...}
        	public String getPrice() {...}
        	public void setPrice(int price) {...}
        
        }
        ```
        <br/>

    - 객체(Bean) 생성 방법: 프로퍼티 접근(Setter)
        
        ```java
        QItem item = QItem.item;
        
        List<ItemDTO> result = query.from(item).list(
        	Projections.bean(ItemDTO.class, item.name.as("username"), item.price)
        );
        ```
        
        - `Projections.bean()` 메서드는 수정자(`Setter`)를 사용해서, DTO의 값을 채운다.
        - '쿼리 결과'와 '매핑할 프로퍼티 이름'이 다르면, `as` 데서드를 사용해서 별칭을 줘서 채우면 된다.
        
        <br/>        

    - 객체(Bean) 생성 방법: 필드 직접 접근
        
        ```java
        QItem item = QItem.item;
        
        List<ItemDTO> result = query.from(item).list(
        	Projections.fields(ItemDTO.class, item.name.as("username"), item.price)
        );
        ```
        
        - `Projections.fields()` 메서드는 DTO 객체의 필드에 직접 접근해서 값을 채운다.
        - 필드를 `private` 로 설정해도 동작한다.
        
        <br/>

    - 객체(Bean) 생성 방법: 생성자 사용
        
        ```java
        QItem item = QItem.item;
        
        List<ItemDTO> result = query.from(item).list(
        	Projections.constructor(ItemDTO.class, item.name, item.price)
        );
        ```
        
        - `Projections.constructor()` 메서드는 DTO 객체의 생성자를 사용한다.
        - 생성자의 파라미터에 값을 전달하므로, `as` 메서드는 필요없다.
        - 물론 생성자의 '파라미터 순서'와 '전달값의 순서'를 맞추어야 한다.

<br/>

### DISTINCT

```java
query.distinct().from(item)...
```

- distinct 명령어는 위와 같이 사용하면 된다.

<br/><br/>

## 수정, 삭제 배치 쿼리

### 수정, 삭제 배치 쿼리의 특징

- QueryDSL은 'JPQL 배치 쿼리'와 같이, **영속성 컨텍스트를 무시하고 DB에 직접 쿼리**한다.

> JPQL 배치 쿼리는 다음 포스팅에서 다룬다.  
지금은 예시 코드 정도만 알아두자.
> 

<br/>

### QueryDSL 코드: 수정 배치 쿼리

```java
QItem item = QItem.item;

JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("매코매개 책"))
		.set(item.price, item.price.add(100))
		.execute();
```

- 위 예시는 '상품의 가격을 100원 증가'시킨다.

<br/>

### QueryDSL 코드: 삭제 배치 쿼리

```java
QItem item = QItem.item;

JPADeleteClause deleteClause = new JPAUpdateClause(em, item);
long count = deleteClause.where(item.name.eq("매코매개 책"))
		.execute();
```

- 위 예시는 '이름이 같은 상품을 삭제'한다.

<br/><br/>

## 동적 쿼리

### QueryDSL 코드

- `com.mysema.query.BooleanBuilder` 를 사용하면, 특정 조건(`if` 문)에 따른 동적 쿼리를 편리하게 생성할 수 있다.

```java
SearchParam param = new SearchParam();
param.setName("매코매개");
param.setPrice(10000);

QItem item = QItem.item;

BooleanBuilder builder = new BooleanBuilder();

// if문: 만약 param.name이 비어있지 않다면
// QueryDSL: .where(item.name.contains(param.getName()))
// JPQL: WHERE절에 [AND item.name LIKE '%매코매개%'] 추가
if (StringUtils.hasText(param.getName())) {
	builder.and(item.name.contains(param.getName()));
}

// if문: 만약 param.price이 비어있지 않다면
// QueryDSL: .where(item.price.gt(param.getPrice()))
// JPQL: WHERE절에 [AND item.price > 10000] 추가
if (param.getPrice() != null) {
	builder.and(item.price.gt(param.getPrice()));
}

List<Item> result = query.from(item)
	.where(builder)
	.list(list);
```

- 상품 이름과 가격 유무에 따라 동적으로 쿼리를 생성한다.
    - 즉 `if` 문을 사용하여, 동적으로 QueryDSL 쿼리를 작성할 수 있다.

<br/><br/>

## 메서드 위임

### 메서드 위임이란?

- 메서드 위임이란, 쿼리 타입에 검색조건을 직접 정의하는 기능이다.
- 기본적으로 제공되는 `eq()` , `gt()` , `contains()` 등의 메서드 이외로, 사용자가 직접 검색조건을 만들 수 있다.

<br/>

### 메서드 위임 절차

1. 정적 메서드를 만든다.
    - 첫번째 파라미터: 대상 엔티티의 쿼리 타입(Q) 지정
    - 두번째 파라미터: 필요한 나머지 파라미터
2. `com.mysema.query.annotations.QueryDelegate` 애너테이션를 정적 메서드에 적용한다.
    - 속성에는 메서드 위임 기능을 적용할 **엔티티를 지정**한다. (쿼리 타입 Q 클래스를 지정하는 것이 아니다.

<br/>

### QueryDSL 코드

- 검색 조건 정의
    
    ```java
    public class ItemExpression {
    
    	@QueryDelegate(Item.class)
    	public static BooleanExpression isExpensive(QItem item, Integer price) {
    
    		return item.price.gt(price); //정의한 검색조건
    
    	}
    
    }
    ```
    
    - 쿼리 타입(`QItem`)에 생성된 결과
        
        ```java
        public class QItem extends EntityPathBase<Item> {
        
        	//...
        
        	public com.mysema.query.types.expr.BooleanExpression
        			isExpensive(Integer price) { //쿼리 타입 파라미터는 생성되지 않는다.
        
        		return ItemExpression.isExpensive(this, price);
        
        	}
        
        }
        ```
        
- 메서드 위임 기능 사용
    
    ```java
    query.from(item).where(item.isExpensive(30000)).list(item);
    ```
    
<br/>

### 참고

- 필요하다면 `String` 이나 `Date` 와 같은 자바 기본 내장 타입에도 메서드 위임 기능을 사용할 수 있다.

```java
@QueryDelegate(String.class)
public static BooleanExpression isHelloStart(StringPath stringPath) {

	return stringPath.startsWith("Hello");

}
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