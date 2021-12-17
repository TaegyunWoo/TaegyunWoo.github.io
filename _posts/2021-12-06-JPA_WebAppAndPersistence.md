---
category: JPA
tags: [JPA]
title: "[JPA] 웹 애플리케이션과 영속성 관리"
date:   2021-12-06 16:19:00 
lastmod : 2021-12-06 16:19:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Spring과 영속성 관리

## 트랜잭션 범위의 영속성 컨텍스트

- 순수하게 J2SE 환경에서 JPA를 사용하면, 개발자가 직접 엔티티 매니저를 생성하고 트랜잭션도 관리해야 한다.
- 스프링이나 J2EE 컨테이너 환경에서 JPA를 사용하면, 컨테이너가 제공하는 전략을 따라야 한다.

<br/>

### 스프링 컨테이너의 기본 전략

- 스프링 컨테이너는 **트랜잭션 범위의 영속성 컨텍스트 전략**을 기본으로 사용한다.
    - **트랜잭션 범위의 영속성 컨텍스트 전략?**
        - 트랜잭션의 범위와 영속성 컨텍스트의 생존 범위가 같다.
        - **즉, 트랜잭션을 시작할 때 영속성 컨텍스트를 생성하고, 트랜잭션이 끝날 때 영속성 컨텍스트를 종료한다. 그리고 같은 트랜잭션 안에서는 항상 같은 영속성 컨텍스트에 접근한다.**
        
        ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled.png)
        

<br/>

- 스프링 프레임워크 사용 시, 비즈니스 로직을 시작하는 서비스 계층에 `Transactional` 애너테이션을 선언해서 트랜잭션을 시작한다.
    - **해당 애너테이션이 있으면, 호출한 메서드를 실행하기 직전에 스프링의 트랜잭션 AOP가 먼저 동작한다.**
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%201.png)
    
    - 스프링 트랜잭션 AOP는 대상 메서드를 호출하기 직전에 트랜잭션을 시작하고, 대상 메서드가 정상 종료되면 트랜잭션을 커밋하면서 종료한다.
        - **이때 트랜잭션을 커밋하면, JPA는 먼저 영속성 컨텍스트를 "플러시"해서 변경 내용을 DB에 반영한 후에 DB 트랜잭션을 커밋한다.**
    - 만약 예외가 발생하면 트랜잭션을 롤백하고 종료하는데, 이때는 플러시를 호출하지 않는다.

예시 코드를 통해 자세히 알아보자.

<br/>

### 트랜잭션 범위의 영속성 컨텍스트 전략 예시 코드

- 컨트롤러 클래스
    
    ```java
    @Controller
    public class HelloController {
    
    	@Autowired
    	HelloService helloService;
    
    	public void hello() {
    		//**상세설명: 4**
    		//반환된 member 엔티티는 준영속 상태이다.
    		Member member = helloService.logic();
    	}
    
    }
    ```
    

- 서비스 클래스
    
    ```java
    @Service
    class HelloService {
    
    	@Autowired
    	Repository1 repository1;
    	@Autowired
    	Repository2 repository2;
    
    	//----------트랜잭션 시작----------
    	@Transactional	//**상세설명: 1**
    	public void logic() {
    		repository1.hello();
    
    		//**상세설명: 2**
    		//member는 영속상태이다.
    		Member member = repository2.findMember();
    		return member;
    	}
    	//----------트랜잭션 종료----------
    	//**상세설명: 3**
    }
    ```
    

- 레파지토리 클래스 1
    
    ```java
    @Repository
    public class Repository1 {
    
    	//엔티티 매니저 주입
    	@PersistenceContext
    	EntityManager em;
    
    	public void hello() {
    		em.xxx(); //영속성 컨텍스트 접근
    	}
    
    }
    ```
    

- 레파지토리 클래스 2
    
    ```java
    @Repository
    public class Repository2 {
    
    	//엔티티 매니저 주입
    	@PersistenceContext
    	EntityManager em;
    
    	public Member findMember() {
    		return em.find(Member.class, "id1"); //영속성 컨텍스트 접근
    	}
    
    }
    ```
    

- 상세 설명
    1. `HelloService.logic()` 에서 `@Transactional` 을 선언해서 메서드를 호출할 때 트랜잭션을 먼저 시작한다.
    2. `repository2.findMember()`
        - 해당 메서드를 통해 조회한 member 엔티티는 트랜잭션 범위 안에 있으므로 영속성 컨텍스트의 관리를 받는다.
        - 따라서 지금은 영속 상태이다.
    3. **`@Transactional` 을 선언한 메서드가 정상 종료되면 트랜잭션을 커밋한다.**
        - **이때 영속성 컨텍스트를 종료한다.**
        - **영속성 컨텍스트가 사라졌으므로 조회한 엔티티(member)는 이제부터 준영속 상태가 된다.**
    4. 서비스 메서드가 끝나면서 트랜잭션과 영속성 컨텍스트가 종료되었다.
        - 따라서 컨트롤러에 반환된 member 엔티티는 준영속 상태이다.

<br/>

### 트랜잭션이 같으면 같은 영속성 컨텍스트를 사용한다.

- 트랜잭션 범위의 영속성 컨텍스트 전략
    - **해당 전략은 다양한 위치에서 엔티티 매니저를 주입받아 사용해도, 트랜잭션이 같으면 항상 같은 영속성 컨텍스트를 사용한다.**
    - **따라서 엔티티 매니저는 달라도 같은 영속성 컨텍스트를 사용한다.**
        - 위 예시에서 `Repository1` 과 `Repository2` 에서 사용하는 `EntityManager` 는 각각 다르지만, 모두 같은 영속성 컨텍스트를 사용한다.
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%202.png)
    
<br/>

### 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.

- 아래 그림과 같이, **여러 쓰레드에서 동시에 요청이 와서 같은 엔티티 매니저를 사용해도 트랜잭션에 따라 접근하는 영속성 컨텍스트가 다르다.**
    - 즉, 스프링 컨테이너는 쓰레드마다 각각 다른 트랜잭션을 알아서 할당해준다.
    - 따라서 멀티 쓰레드 상황에서도 안전할 수 있다.
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%203.png)
    
<br/><br/>

## 준영속 상태와 지연 로딩

### 스프링에서의 엔티티 상태

- 다시 말하지만, 스프링이나 J2EE 컨테이너는 **트랜잭션 범위의 영속성 컨텍스트 전략** 을 기본으로 사용한다.
- 따라서 엔티티 상태는 아래와 같다.
    - 서비스, 레파지토리 계층
        - 조회한 엔티티가 **영속 상태** 를 유지한다.
    - 컨트롤러, 뷰 계층 (프리젠테이션 계층)
        - 조회한 엔티티가 **준영속 상태** 가 된다.
        
        > **트랜잭션이 서비스 계층에서 끝나고, 이와 동시에 영속성 컨텍스트가 종료되었으므로**
        > 

<br/>

### 발생하는 문제

엔티티 상태가 위와 같이 변화하는 것에서 오는 문제에 대해 예시코드로 알아보자.

- 주문 엔티티
    
    ```java
    @Entity
    public class Order {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	//지연 로딩 전략
    	@ManyToOne(fetch = FetchType.LAZY)
    	private Member member; //주문 회원
    
    	// 이하 생략...
    }
    ```
    
- 컨트롤러 로직
    
    ```java
    @Controller
    public class OrderController {
    
    	public String view(Long orderId) {
    
    		Order order = orderService.findOne(orderId);
    		Member member = order.getMember();
    		member.getName(); // 이때 프록시 객체 member가 로딩된다. => 예외 발생!
    
    	}
    
    }
    ```
    
    - `member.getName()`
        - 이때 예외가 발생한다.
        - `order` 엔티티는 준영속 상태이고, 이와 연관된 `member` 엔티티는 현재 프록시 객체인데, 프록시 객체에 접근하면 DB에 접근해서 실제 데이터를 가져온다.
        - 하지만 `order` 엔티티가 준영속 상태이므로 실제 데이터를 가져올 수 없다. 따라서 예외가 발생한다.

<br/>

### 준영속 상태와 변경 감지

- **변경 감지 기능 (Dirty Checking)**
    - 해당 기능은 영속성 컨텍스트가 살아 있는 서비스 계층까지만 동작하고, 영속성 컨텍스트가 종료된 프리젠테이션 계층에서는 동작하지 않는다.
- **단순히 데이터를 보여주기만 하는 프리젠테이션 계층에서 데이터를 수정할 일은 거의 없다.**
    - 비즈니스 로직은 서비스 계층에서 끝내고, 프리젠테이션 계층은 데이터를 보여주는 데 집중해야 한다.
    - **따라서 변경 감지 기능이 프리젠 테이션 계층에서 동작하지 않는 것은 특별히 문제가 되지 않고, 오히려 좋다.**

<br/>

### 준영속 상태와 지연 로딩

- 준영속 상태의 가장 골치 아픈 문제가 바로 **지연 로딩 기능이 동작하지 않는다는 것** 이다.
- 이것을 해결하기 위한 방법은 아래 2가지가 있다.
    - **뷰가 필요한 엔티티를 미리 로딩해두는 방법**
    - **OSIV를 사용해서 엔티티를 항상 영속 상태로 유지하는 방법**

가장 먼저, **뷰가 필요한 엔티티를 미리 로딩해두는 방법** 에 대해 알아보자.

- 뷰가 필요한 엔티티를 미리 로딩해두는 방법?
    - **영속성 컨텍스트가 살아 있을 때, 뷰에 필요한 엔티티들을 미리 다 로딩하거나 초기화해서 반환하는 방법이다.**
    - **따라서 엔티티가 준영속 상태로 변해도 연관된 엔티티를 이미 다 로딩해두어서 지연 로딩이 발생하지 않는다.**
    - 뷰가 필요한 엔티티를 미리 로딩해두는 방법의 종류
        - **글로벌 페치 전략 수정**
        - **JPQL 페치 조인**
        - **강제로 초기화**

아래에서 하나씩 설명하겠다.

<br/><br/>

## 뷰가 필요한 엔티티를 미리 로딩해두는 방법: 글로벌 페치 전략 수정

### 글로벌 페치 전략이란?

- 뷰가 필요한 엔티티를 미리 로딩해두는 아주 간단한 방법이다.
- 그저 글로벌 페치를 지연 로딩에서 즉시 로딩으로 변경하면 된다.

<br/>

### 예시 코드

- 엔티티 클래스
    
    ```java
    @Entity
    public class Order {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	@ManyToOne(fetch = FetchType.EAGER) //즉시 로딩 전략
    	private Member member; // 주문 회원
    
    }
    ```
    
- 프리젠테이션 로직
    
    ```java
    Order order = orderService.findOne(orderId); //order = 준영속상태
    Member member = order.getMember();
    member.getName(); //이미 로딩된 엔티티 사용
    ```
    

- 상세 설명
    - 엔티티 매니저로 주문 엔티티를 조회하면 연관된 `member` 엔티티도 항상 함께 로딩한다.
    - 하지만 이렇게 글로벌 페치 전략을 즉시 로딩으로 설정하면 2가지 문제가 발생한다.

<br/>

### 글로벌 페치 전략에 즉시 로딩 사용 시 문제점

- **사용하지 않는 엔티티를 로딩한다.**
    - 만약 화면B(뷰)에서 `order` 엔티티만을 필요로 해도, `member` 엔티티를 로딩하게 된다.
- **N+1 문제가 발생한다.**
    - `em.find()` 메서드로 엔티티를 조회할 때 연관된 엔티티를 로딩하는 전략이 즉시 로딩이면, **DB에 JOIN 쿼리를 사용해서 한번에 연관된 엔티티까지 조회**한다.
    - Java 예시 코드
        
        ```java
        Order order = em.find(Order.class, 1L);
        ```
        
    - 실행된 SQL
        
        ```sql
        SELECT o.*, m.*
        FROM ORDER o
        LEFT OUTER JOIN MEMBER m ON o.MEMBER_ID=m.MEMBER_ID
        WHERE o.ID=1
        ```
        
    - 여기까지 보았을 땐, 딱히 문제가 없다. 하지만 **문제는 JPQL을 사용할 때 발생**한다. 아래 예시를 보자
    - JPQL 사용 예시 코드
        
        ```java
        List<Order> orders = em.createQuery("select o from Order o", Order.class)
        			.getResultList();
        ```
        
    - 실행된 SQL
        
        ```sql
        SELECT o.* FROM ORDER o;
        
        //JOIN으로 연관 데이터를 조회하지 않고,
        //일일히 가져온다.
        SELECT m FROM MEMBER m WHERE m.ID=?; //EAGER로 실행된 SQL
        SELECT m FROM MEMBER m WHERE m.ID=?; //EAGER로 실행된 SQL
        SELECT m FROM MEMBER m WHERE m.ID=?; //EAGER로 실행된 SQL
        SELECT m FROM MEMBER m WHERE m.ID=?; //EAGER로 실행된 SQL
        ```
        
    - **JPA가 JPQL을 분석해서 SQL을 생성할 때는 글로벌 페치 전략을 참고하지 않고 오직 JPQL만을 사용한다.**
        - **즉 JPQL 쿼리 자체에만 충실하게 SQL을 만든다.**
    - 내부 흐름 순서
        1. `select o from Order o` JPQL을 분석해서 `SELECT o.* FROM ORDER o` SQL을 생성한다.
        2. DB에서 결과를 받아 `order` 엔티티 인스턴스들을 생성한다.
        3. `Order.member` 의 글로벌 페치 전략이 즉시 로딩이므로, `order` 를 로딩하는 즉시 연관된 `member` 도 로딩해야 한다.
        4. 연관된 `member` 를 영속성 컨텍스트에서 찾는다.
        5. **만약 영속성 컨텍스트에 없으면, `SELECT * FROM MEMBER WHERE id=?` SQL을 조회한 `order` 엔티티 수만큼 실행한다.**
    - **즉 조회한 `order` 엔티티가 10개이면, `member` 를 조회하는 SQL도 10번 실행한다.**
    - 이처럼 **처음 조회한 데이터 수만큼 다시 SQL을 사용해서 조회하는 것을 N+1 문제**라고 한다.
    - **이런 N+1 문제는 JPQL 페치 조인으로 해결할 수 있다.**

<br/><br/>

## 뷰가 필요한 엔티티를 미리 로딩해두는 방법: JPQL 페치 조인

### JPQL 페치 조인이란?

> **페치 조인에 대해서는 [이전 게시글](https://taegyunwoo.github.io/jpa/JPA_ObjectQuery_JPQL_Join#17)을 참고하자.**
> 

<br/>

### 예시 코드

- 페치 조인 사용 전
    
    ```xml
    JPQL: select o from Order o
    SQL:  select * from Order
    ```
    
- 페치 조인 사용 후
    
    ```xml
    JPQL:
    	select o
    	from Order o
    	join fetch o.member
    
    SQL:
    	select o.*, m.*
    	from Order o
    	inner join Member m
    	on o.MEMBER_ID=m.MEMBER_ID
    ```
    

- 상세 설명
    - 페치 조인을 사용하면, SQL JOIN을 사용해서 페치 조인 대상까지 함께 조회한다.
    - **따라서 N+1 문제가 발생하지 않는다.**

<br/>

### JPQL 페치 조인의 단점

- **무분별하게 해당 방법을 사용하면, 화면에 맞춘 레파지토리 메서드가 증가할 수 있다.**
    - 결국 프리젠테이션 계층이 알게 모르게 데이터 접근 계층을 침범하게 된다.
- 예시
    - 화면 A는 `order` 엔티티만 필요로 한다.
    - 화면 B는 `order` 와 연관된 엔티티 `member` 둘 다 필요하다.
    - 결국 두 화면을 모두 최적화하기 위해, 이 둘을 지연 로딩으로 설정하고 레파지토리에 아래 2가지 메서드를 만들게 된다.
        - `order`만 조회하는 `repository.findOrder()` 메서드  
        (화면 A를 위해)
        - `order`와 `member` 모두 조회하는 `repository.findOrderWithMember()` 메서드  
        (화면 B를 위해)
- **위 예시처럼 메서드를 각각 만들면 최적화는 할 수 있지만, 뷰와 레파지토리 간에 논리적인 의존관계가 발생한다.**

<br/>

### JPQL 페치 조인 단점 보완 방법

위 예시를 사용하여, 단점을 보완하는 방법에 대해 설명하겠다.

- `repository.findOrder()` 하나만 만든다.
    - **여기서 페치 조인으로 `order` 와 `member` 를 함께 로딩한다.**
- 그리고 화면 A와 B 둘 다 `repository.findOrder()` 메서드를 사용하도록 한다.
    - 물론 `order` 엔티티만 필요한 화면 B는 약간의 로딩 시간이 증가하겠지만, 페치 조인은 JOIN을 사용해서 쿼리 한번으로 필요한 데이터를 조회하기 때문에, 성능에 미치는 영향은 미비하다.

<br/><br/>

## 뷰가 필요한 엔티티를 미리 로딩해두는 방법: 강제로 초기화

### 강제로 초기화하기?

- 영속성 컨텍스트가 살아있을 때, 프리젠테이션 계층이 필요한 엔티티를 강제로 초기화해서 반환하는 방법이다.
    - **즉, 프록시 객체를 강제로 로딩시키고 프리젠테이션 계층에게 전달해주는 방법이다.**

> **프록시 객체는 [이전 게시글을 참고](https://taegyunwoo.github.io/jpa/JPA_Proxy)하자.**
> 

<br/>

### 예시 코드

- 서비스 계층의 클래스
    
    ```java
    @Service
    public class OrderService {
    
    	@Autowired
    	private OrderRepository orderRepository;
    
    	@Transactional
    	public Order findOrder(Long id) {
    		Order order = orderRepository.findOrder(id);
    		order.getMember().getName(); //프록시 객체를 강제로 초기화한다!
    		return order;
    	}
    
    }
    ```
    
- 상세 설명
    - 프록시 객체는 `member.getName()` 처럼 직접 사용하는 시점에 초기화된다.
    - 트랜잭션 단계에선 `order` 엔티티가 영속 상태이므로, 프록시 객체를 초기화할 수 있다.
    - 그리고 이미 초기화된 프록시 객체를 프리젠테이션 계층에 전달하면, 준영속 상태에서도 사용할 수 있다.
    - 하이버네이트를 사용하면, `initialize()` 메서드를 사용해서 프록시를 강제로 초기화할 수도 있다.
        
        ```java
        org.hibernate.Hibernate.initialize(order.getMember()); //프록시 초기화
        ```
        
        > JPA 표준에는 프록시 초기화 메서드가 없다.  
        단순히 초기화 여부만 확인할 수 있다. (관련 메서드는 따로 찾아보자.)
        > 

<br/>

### 강제로 초기화 시 발생하는 문제

- 위 예시를 다시보자.
- **프록시를 초기화하는 역할을 서비스 계층이 담당하면, 뷰가 필요한 엔티티에 따라 서비스 계층의 로직을 변경해야 한다.**
    - **즉 프리젠테이션 계층이 서비스 계층을 침범하게 된다.**
- 따라서 비즈니스 로직을 담당하는 서비스 계층에서 '프리젠테이션 계층을 위한 프록시 초기화 역할'을 분리해야 한다.
    - **FACADE 계층이 그 역할을 수행한다.**

<br/>

### FACADE 계층 추가

- FACADE 계층이란?
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%204.png)
    
    - 위 그림은 프리젠테이션 계층과 서비스 계층 사이에 FACADE 계층을 하나 더 두는 방법을 나타낸다.
    - **이제부터 '뷰를 위한 프록시 초기화'는 이곳에서 담당한다.**
    - **이를 통해, 서비스 계층과 프리젠테이션 계층 사이에 논리적인 의존성을 분리할 수 있다.**
    - 프록시를 초기화하려면 영속성 컨텍스트가 필요하므로, FACADE 에서 트랜잭션을 시작해야 한다.

<br/>

- FACADE 계층의 역할과 특징
    - 프리젠테이션 계층과 도메인 모델 계층 간의 논리적 의존성을 분리해준다.
    - 프리젠테이션 계층에서 필요한 프록시 객체를 초기화한다.
    - 서비스 계층을 호출해서 비즈니스 로직을 실행한다.
    - 레파지토리를 직접 호출해서 뷰가 요구하는 엔티티를 찾는다.

<br/>

- FACADE 계층 추가 예시 코드
    - FACADE 계층 클래스
        
        ```java
        public class OrderFacade {
        
        	@Autowired
        	OrderService orderService;
        
        	public Order findOrder(Long id) {
        		Order order = orderService.findOrder(id);
        		//프리젠테이션 계층이 필요한 프록시 객체를 강제로 초기화한다.
        		order.getMember().getName();
        		return order;
        	}
        
        }
        ```
        
    - 서비스 계층 클래스
        
        ```java
        public class OrderService {
        
        	public Order findOrder(Long id) {
        		return orderRepository.findOrder(id);
        	}
        
        }
        ```
        
    - 상세 설명
        - `OrderService` 에 있던 프록시 초기화 코드를 `OrderFacade` 로 이동했다.
        - FACADE 계층을 사용해서 서비스 계층과 프리젠테이션 계층 간에 논리적 의존관계를 제거했다.

<br/>

- FACADE의 단점
    - **중간에 계층이 하나 더 끼어든다는 점이 최대 단점이다.**

<br/><br/>

## 준영속 상태와 지연 로딩의 문제점

### 지금까지 알아본 방법들에 대한 문제

- 지금까지 준영속 상태일 때, 지연 로딩 문제를 극복하기 위해 여러가지 전략을 알아보았다.
- 하지만 이러한 방법들은 오류가 발생할 가능성이 높다.
    - 왜냐하면 보통 뷰를 개발할 때는 엔티티 클래스를 보고 개발하지, 이것이 초기화되어 있는지 확인하기 위해 FACADE나 서비스 클래스를 열어보지는 않기 때문이다.

<br/>

### 더 나은 방법

- **모든 문제는 엔티티가 프리젠테이션 계층에서 준영속 상태이기 때문에 발생한다.**
- **따라서 영속성 컨텍스트가 뷰까지 살아있게 열어두면 되는 것 아닌가?**
- 그럼 뷰에서도 지연 로딩을 사용할 수 있을 것이다.
    - 그리고 이것을 OSIV라고 한다.

<br/><br/>

## OSIV

### OSIV란?

- OSIV = Open Session In View
- 영속성 컨텍스트를 뷰까지 열어둔다는 뜻이다.
- 따라서 뷰에서도 지연 로딩을 사용할 수 있다.

<br/>

### 과거 OSIV: 요청 당 트랜잭션

- OSIV의 핵심
    - 뷰에서도 지연 로딩이 가능하도록 하는 것
- 단순한 구현 방법
    - 클라이언트 요청이 들어오자마자 서블릿 필터나 스프링 인터셉터에서 트랜잭션을 시작하고 요청이 끝날 때 트랜잭션도 끝낸다.
    - 이것을 **요청 당 트랜잭션 방식의 OSIV** 라고 한다.
- 요청 당 트랜잭션 방식의 OSIV 시각화
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%205.png)
    
    - **요청이 들어오자마자 서블릿 필터나 스프링 인터셉터에서 영속성 컨텍스트를 만들고, 트랜잭션을 시작하고 요청이 끝날 때 트랜잭션과 영속성 컨텍스트를 함께 종료한다.**
    - 이제 뷰에서도 지연 로딩을 할 수 있으므로 엔티티를 미리 초기화할 필요가 없다.

<br/>

### 요청 당 트랜잭션 방식의 OSIV 문제점

- 컨트롤러나 뷰 같은 프리젠테이션 계층이 엔티티를 변경할 수 있다.
- 아래 예시를 보자.
- 예시) 고객 예제를 출력해야 하는데, 보안상의 이유로 고객이름을 XXX로 변경해서 출력해야 한다고 가정하자.
    
    ```java
    public class MemberController {
    
    	public String viewMember(Long id) {
    		Member member = memberService.getMember(id);
    		member.setName("XXX"); //보안상 이유로 고객 이름을 XXX로 변경했다.
    		model.addAttribute("member", member);
    
    		//이하 생략...
    
    	}
    
    }
    ```
    
    - 컨트롤러에서 고객 이름을 XXX로 변경해서 렌더링할 뷰에 넘겨주었다.
    - 개발자의 의도
        - 단순히 뷰에 노출할 때만 고객 이름을 XXX로 변경하고 싶은 것이지, 실제 DB에 있는 고객 이름까지 변경하고 싶은 것은 아니었다.
    - 하지만 요청당 트랜잭션 방식의 OSIV는 뷰를 렌더링한 후에 트랜잭션을 커밋한다.
    - 트랜잭션을 커밋하면 영속성 컨텍스트를 플러시하고, Dirty Checking이 동작해서 실제 DB에 반영해버린다.
    - 즉 DB의 고객이름이 XXX로 변경된다.
- 이런 문제를 해결하려면, 프리젠테이션 계층에서 엔티티를 수정하지 못하게 막으면 된다.
- 프리젠테이션 계층에서 엔티티를 수정하지 못하게 막는 방법들
    - 엔티티를 읽기 전용 인터페이스로 제공하기
    - 엔티티 래핑
    - DTO만 반환
- **하지만 이러한 방법들은 너무 번거롭고, 효율적이지 않다.**
    - 따라서 비즈니스 계층에서만 트랜잭션을 유지하는 방식의 OSIV를 사용하는 것이 좋다.
    - **스프링 프레임워크가 제공하는 OSIV가 바로 이런 방식을 사용하는 OSIV이다.**

<br/>

### 스프링 OSIV: 비즈니스 계층 트랜잭션

OSIV를 서블릿 필터에서 적용할지, 스프링 인터셉터에서 적용할지에 따라 원하는 클래스를 선택해서 사용하면 된다.

- 스프링 프레임워크가 제공하는 OSIV 라이브러리 종류
    - **하이버네이트 OSIV 서블릿 필터**
        - `org.springframework.orm.hibernate4.support.OpenSessionInViewFilter`
    - **하이버네이트 OSIV 스프링 인터셉터**
        - `org.springframework.orm.hibernate4.support.OpenSessionInViewInterceptor`
    - **JPA OEIV 서블릿 필터**
        - `org.springframework.orm.jpa.support.OpenSessionInViewFilter`
    - **JPA OEIV 스프링 인터셉터**
        - `org.springframework.orm.jpa.support.OpenSessionInViewInterceptor`

- 스프링 OSIV 적용하기
    - 서블릿 필터에 적용하기
        - `OpenEntityManagerInViewFilter` 를 서블릿 필터에 등록한다.
    - 스프링 인터셉터에 적용하기
        - `OpenEntityManagerInViewInterceptor` 를 스프링 인터셉터에 등록한다.

<br/>

### 스프링 OSIV 분석

- 이전에 설명했던 요청 당 트랜잭션 방식의 OSIV는 프리젠테이션 계층에서 데이터를 변경할 수 있다는 문제가 있다.
- **스프링 OSIV은 비즈니스 계층에서 트랜잭션을 사용하는 OSIV이다.**
    - 즉 영속성 컨텍스트는 요청처리가 끝날 때까지 유지되지만, 트랜잭션은 비즈니스 계층에서만 동작한다.
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%206.png)
    

- 동작 원리
    1. 클라이언트 요청이 들어오면 서블릿 필터나, 스프링 인터셉터에서 **영속성 컨택스트를 생성**한다.
        - **단 이때 트랜잭션은 시작하지 않는다.**
    2. 서비스 계층에서 `@Transactional` 로 트랜잭션을 시작할 때, **1번에서 미리 생성해둔 영속성 컨텍스트를 찾아와서 트랜잭션을 시작**한다.
    3. 서비스 계층이 끝나면 트랜잭션을 커밋하고, 영속성 컨텍스트를 플러시한다.
        - **단 이때 트랜잭션은 끝내지만, 영속성 컨텍스트는 종료하지 않는다.**
    4. 컨트롤러와 뷰까지 영속성 컨텍스트가 유지되므로, **조회한 엔티티는 영속 상태를 유지**한다.
    5. 서블릿 필터나 스프링 인터셉터로 요청이 돌아오면, **영속성 컨텍스트를 종료**한다.
        - **이때 플러시를 호출하지 않고 바로 종료한다.**

<br/>

### 트랜잭션 없이 읽기

- 영속성 컨텍스트를 통한 모든 변경은 트랜잭션 안에서 이루어져야 한다.
- **엔티티를 변경하지 않고, 단순히 조회만 할 때는 트랜잭션이 없어도 된다.**
    - **이것을 '트랜잭션 없이 읽기'라고 한다.**
- **프록시를 초기화하는 지연 로딩도 조회 기능이므로, '트랜잭션 없이 읽기'가 가능하다.**

<br/>

- 정리
    - **영속성 컨텍스트는 트랜잭션 범위 안에서 엔티티를 조회하고 수정할 수 있다.**
    - **영속성 컨텍스트는 트랜잭션 범위 밖에서 엔티티를 조회만 할 수 있다.**
        - 이것을 '트랜잭션 없이 읽기'라고 한다.

<br/>

- **스프링이 제공하는 OSIV를 사용하면 프리젠테이션 계층에서는 트랜잭션이 없으므로 엔티티를 수정할 수 없다.**
    - 따라서 프리젠테이션 계층에서 엔티티를 수정할 수 있는 기존 OSIV의 단점을 보완했다.
    - 그리고 트랜잭션 없이 읽기를 사용해서 프리젠테이션 계층에서 지연 로딩 기능을 사용할 수 있다.
- 스프링 OSIV의 특징
    - 영속성 컨텍스트를 프리젠테이션 계층까지 유지한다.
    - 프리젠테이션 계층에는 트랜잭션이 없으므로 엔티티를 수정할 수 없다.
    - **프리젠테이션 계층에는 트랜잭션이 없지만, '트랜잭션 없이 읽기'를 사용해서 지연 로딩을 할 수 있다.**

<br/>

### 스프링 OSIV 적용 후, 예시 코드

```java
public class MemberController {

	@Autowired
	MemberService memberSerivce;

	public String viewMember(Long id) {
		Member member = memberService.getMember(id); //영속 상태 (수정 불가)
		member.setName("XXX"); //보안상 이유로 고객 이름을 XXX로 변경했다.
		model.addAttribute("member", member);
	}

}
```

- 상세 설명
    - 컨트롤러에서 회원 엔티티를 `member.setName("XXX")` 로 변경했다.
    - 프리젠테이션 계층이지만, 아직 영속성 컨텍스트가 살아있다.
    - 플러시가 동작하지 않는 이유
        - 트랜잭션을 사용하는 서비스 계층이 끝날 때, 트랜잭션이 커밋되면서 이미 플러시해버렸다. 요청이 끝나면 플러시를 호출하지 않고, `em.close()` 로 영속성 컨텍스트만 종료해 버리므로 플러시가 일어나지 않는다.
        - 트랜잭션 범위 밖이므로 데이터를 수정할 수 없다.

<br/>

### 스프링 OSIV 주의사항

- 프리젠테이션 계층에서 엔티티를 수정한 직후에 트랜잭션을 시작하는 서비스 계층을 호출하면 아래와 같은 문제가 발생한다.
- 문제 발생 예시 코드
    
    ```java
    public class MemberController {
    
    	@Autowired
    	MemberService memberSerivce;
    
    	public String viewMember(Long id) {
    		Member member = memberService.getMember(id); //영속 상태 (수정 불가)
    		member.setName("XXX"); //보안상 이유로 고객 이름을 XXX로 변경했다.
    		model.addAttribute("member", member);
    
    		//트랜잭션 시작
    		memberService.biz(); //비즈니스 로직 (@Transactional)
    		//트랜잭션 종료
    
    		return "view";
    	}
    
    }
    ```
    
- 문제 시각화
    
    ![Untitled](/assets/img/2021-12-06-JPA_WebAppAndPersistence/Untitled%207.png)
    
    1. 컨트롤러에서 회원 엔티티를 조회하고 이름을 `member.setName("XXX")` 로 수정했다.
    2. `biz()` 메서드를 실행해서 트랜잭션이 있는 비즈니스 로직이 실행되었다.
    3. 트랜잭션 AOP가 동작하면서 영속성 컨텍스트에 트랜잭션을 시작한다. 그리고 `biz()` 메서드를 실행한다.
    4. `biz()` 메서드가 끝나면 트랜잭션 AOP는 트랜잭션을 커밋하고 영속성 컨텍스트를 플러시한다. **이때 변경 감지(Dirty Checking)이 동작하면서 회원 엔티티의 수정 사항을 DB에 반영한다.**

<br/>

- 문제 원인
    - 컨트롤러에서 엔티티를 수정하고 즉시 뷰를 호출한 것이 아니라, 트랜잭션이 동작하는 비즈니스 로직을 호출했으므로 이런 문제가 발생한다.
- 문제 해결 방법
    - **트랜잭션이 있는 비즈니스 로직을 모두 호출하고 나서 엔티티를 변경하면 된다.**
    
    ```java
    //트랜잭션 시작
    memberService.biz(); //비즈니스 로직 (@Transactional)
    //트랜잭션 종료
    
    Member member = memberService.getMember(id); //영속 상태 (수정 불가)
    member.setName("XXX"); //보안상 이유로 고객 이름을 XXX로 변경했다.
    model.addAttribute("member", member);
    
    return "view";
    ```
    
- 스프링 OSIV는 같은 영속성 컨텍스트를 여러 트랜잭션이 공유할 수 있으므로 이런 문제가 발생한다.

### OSIV 정리

- 스프링 OSIV의 특징
    - OSIV는 클라이언트의 요청이 들어올 때 영속성 컨텍스트를 생성해서, 요청이 끝날 때까지 같은 영속성 컨텍스트를 유지한다.
    - 엔티티 수정은 트랜잭션이 있는 계층에서만 동작한다. 트랜잭션이 없는 프리젠테이션 계층은 **지연 로딩을 포함해서 조회만** 할 수 있다.
- 스프링 OSIV의 단점
    - OSIV를 적용하면 같은 영속성 컨텍스트를 여러 트랜잭션이 공유할 수 있다는 점을 주의해야 한다.
        - 특히 트랜잭션 롤백 시 주의해야 한다.
    - 프리젠테이션 계층에서 엔티티 수정하고나서 비즈니스 로직을 수행하면 엔티티가 수정될 수 있다.
    - 프리젠테이션 계층에서 지연 로딩에 의한 SQL이 실행된다. 따라서 성능 튜닝 시에 확인해야 할 부분이 넓다.
- OSIV를 사용하는 방법이 만능은 아니다.
    - 복잡한 화면을 구성할 때는 이 방법이 효과적이지 않은 경우가 많다.
        - 수많은 테이블을 조인해서 보여주어야 하는 경우 등...
    - 이땐 JPQL로 필요한 데이터들만 조회해서 DTO로 반환하는 것이 더 나은 해결책일 수 있다.
- OSIV는 같은 JVM을 벗어난 원격 상황에서는 사용할 수 없다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>