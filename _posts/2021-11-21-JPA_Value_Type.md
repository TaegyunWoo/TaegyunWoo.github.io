---
category: JPA
tags: [JPA]
title: "[JPA] 값 타입"
date:   2021-11-21 01:00:00 
lastmod : 2021-11-21 01:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 값 타입

## 개요

### JPA의 데이터 타입 종류

JPA의 데이터 타입 종류에는 크게 2가지가 있다.

> JPA의 데이터 타입: 쉽게 말해 '엔티티가 갖는 속성 타입'을 말한다.
> 

- **엔티티 타입**
    - `@Entity` 로 정의하는 객체
    - 식별자를 통해, 지속적으로 추적할 수 있다.
- **값 타입**
    - `int` , `Integer` , `String` 등과 같은 자바 기본 타입이나 객체
    - 식별자가 없고 숫자나 문자같은 속성만 존재하여, 추적할 수 없다.

<br/>

### 값 타입의 종류

여기서 **값 타입**은 다시 3가지 종류로 구분될 수 있다.

- **기본값 타입**
    - 자바 기본 타입 (`int` , `double` 등)
    - 래퍼 클래스 (`Integer` , `Long` 등)
    - `String`
- **임베디드 타입 (복합값 타입)**
    - JPA에서 사용자가 직접 정의한 값 타입
- **컬렉션 값 타입**
    - 하나 이상의 값 타입을 저장할 때 사용하는 타입

이제부터 예시코드와 함께 하나씩 알아보자.

<br/><br/>

## 값 타입: 기본값 타입

### 예시 코드

```java
@Entity
public class Member {

	@Id @GeneratedValue
	private Long id;

	//기본값 타입
	private String name;
	private int age;

	// 생성자, getter, setter 생략
}
```

- `Member` 엔티티
    - 다른 엔티티의 속성으로 사용될 수 있다. ⇒ 엔티티 타입
    - `id` 라는 식별자 값을 갖는다.
    - 생명주기도 역시 존재한다.
- `name` , `age` 속성
    - 값 타입 속성이다.
    - 식별자 값이 없다.
    - 생명주기도 없다.

우리가 지금까지 JPA를 공부하며, 자주 다룬 내용이다. 이것은 너무 간단하니 빠르게 넘어가겠다.

<br/><br/>

## 값 타입: 임베디드 타입 (복합값 타입)

### 임베디드 타입이란?

- JPA에서 새로운 값 타입을 직접 정의해서 사용하는 타입을 말한다.
- 중요사항
    - **직접 정의한 임베디드 타입도 `int` , `String` 과 같은 값 타입으로 취급된다!**

<br/>

### 예시 코드: 임베디드 타입 적용을 하지 않는다면

```java
@Entity
public class Member {

	@Id @GeneratedValue
	private Long id;

	private String name;

	//근무 기간
	@Temporal(TemporalType.DATE)
	java.util.Date startDate;

	@Temporal(TemporalType.DATE)
	java.util.Date endDate;

	//집주소
	private String city;
	private String street;
	private String zipcode;

	// 생성자, getter, setter 생략
}
```

- 근무 기간 속성
    - 근무 기간을 표현하기 위해 시작일, 종료일을 아래와 같이 표현하였다.
    - `Date startDate` , `Date endDate`
- 집주소 속성
    - 집주소를 표현하기 위해 도시, 지번, 우편번호를 아래와 같이 표현하였다.
    - `String city` , `String street` , `String zipcode`

<br/>

### 예시 코드: 임베디드 타입 적용을 한다면

- 위에서 설명한 방식으로 근무 기간과 집주소를 표현하는 것보다, 더욱 명확하게 설명할 수 있는 방법은 '**직접 작성한 클래스 타입으로 설정하는 것**'이다.
- 이에 대한 예시는 아래와 같다.
    - 근무 기간
        - `Date startDate` , `Date endDate` ⇒ `Period workPeriod`
    - 집 주소
        - `String city` , `String street` , `String zipcode` ⇒ `Address homeAddress`

<br/>

전체 예시 코드는 아래와 같다.

- 회원 엔티티 클래스
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	private String name;
    
    	//근무 기간
    	@Embedded
    	private Period workPeriod;
    
    	//집주소
    	@Embedded
    	private Address homeAddress;
    
    	// 생성자, getter, setter 생략
    }
    ```
    

- 기간 임베디드 타입
    
    ```java
    @Embeddable
    public class Period {
    
    	@Temporal(TemporalType.DATE)
    	java.util.Date startDate;
    
    	@Temporal(TemporalType.DATE)
    	java.util.Date endDate;
    
    	// 생성자, getter, setter 생략
    
    	//기타 필요한 메서드 추가 가능
    
    }
    ```
    

- 주소 임베디드 타입
    
    ```java
    @Embeddable
    public class Address {
    
    	@Column(name = "city") //생략가능
    	private String city;
    	private String street;
    	private String zipcode;
    
    	// 생성자, getter, setter 생략
    
    	//기타 필요한 메서드 추가 가능
    
    }
    ```
    
<br/>

- 회원 클래스
    - `startDate` , `endDate` 를 합쳐서 `Period` (기간) 클래스를 만들었다.
    - `city` , `street` , `zipcode` 를 합쳐서 `Address` (주소) 클래스를 만들었다.
    - 이를 통해, 기존 코드보다 훨씬 응집력있고 명확해졌다.

- 임베디드 타입 클래스
    - 해당 클래스를 통해, 사용자 임의의 타입을 만들고 매핑할 수 있다.
    - 그리고 해당 값 타입만 사용하는 의미 있는 메서드 역시 만들 수 있다.
    - **기본 생성자가 필수적으로 필요하다!**

- 임베디드 타입 애너테이션
    - `@Embeddable`
        - 값 타입을 정의하는 곳에 적용한다.
    - `@Embedded`
        - 값 타입을 사용하는 곳에 적용한다.

- 모든 값 타입들은 엔티티의 생명주기에 의존한다.
    - 즉 엔티티가 제거되면, 관련된 값 타입들도 제거된다.
    - 따라서 '엔티티'와 '임베디드 타입'의 관계를 UML로 표현하면, **컴포지션 관계**가 된다.
        - 컴포지션 관계
            
            ![Untitled](/assets/img/2021-11-21-JPA_Value_Type/Untitled.png)
            

> '하이버네이트에서의 임베디드 타입' == '컴포넌트'
> 

<br/>

### 임베디드 타입과 테이블 매핑

임베디드 타입 사용 시, DB 테이블에는 어떻게 매핑될까? 이는 아래 그림과 같다.

![Untitled](/assets/img/2021-11-21-JPA_Value_Type/Untitled%201.png)

- **임베디드 타입은 엔티티의 값일 뿐이다.**
    - 따라서 값이 속한 엔티티의 테이블에 매핑한다.

<br/>

### 임베디드 타입과 연관관계

- 임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있다.
    - 즉, 위에서 살펴본 `Period` 나 `Address` 클래스에서 값 타입을 포함하거나, 엔티티를 참조할 수 있다.

<br/>

- 예시 코드
    - 멤버 엔티티 클래스
        
        ```java
        @Entity
        public class Member {
        
        	@Embedded
        	Address address; //임베디드 타입 포함
        
        	@Embedded
        	PhoneNumber phoneNumber; //임베디드 타입 포함
        
        	// 생성자, getter, setter 생략
        
        }
        ```
        
    - 주소 임베디드 타입 클래스
        
        ```java
        @Embeddable
        public class Address {
        
        	@Embedded
        	Zipcode zipcode; //임베디드 타입 포함
        
        	String street;
        	String city;
        	String state;
        
        	//기본 생성자 생략
        }
        ```
        
    - 우편번호 임베디드 타입 클래스
        
        ```java
        @Embeddable
        public class Zipcode {
        
        	String zip;
        	String plusFour;
        
        	//기본 생성자 생략
        }
        ```
        
    - 전화번호 임베디드 타입 클래스
        
        ```java
        @Embeddable
        public class PhoneNumber {
        
        	@ManyToOne
        	PhoneServiceProvider provider; //엔티티 참조
        
        	String areaCode;
        	String localNumber;
        
        	//기본 생성자 생략
        }
        ```
        
    - 통신사 임베디드 타입 클래스
        
        ```java
        @Entity
        public class PhoneServiceProvider {
        
        	@Id
        	String name;
        
        	//이하 생략
        }
        ```
        
<br/>

- 상세설명
    - '값 타입 `Address`' 가 '값 타입 `Zipcode`'를 포함한다.
    - '값 타입 `PhoneNumber`'가 '엔티티 타입 `PhoneServiceProvider`'를 참조한다.

<br/>

### 속성 재정의: `@AttributeOverride`

- `@AttibuteOverride` 애너테이션을 통해, 임베디드 타입에 정의한 매핑정보를 재정의할 수 있다.

<br/>

- 예시 코드
    - 임베디드 타입을 재정의하지 않을 때
        
        ```java
        @Entity
        public class Member {
        
        	@Id @GeneratedValue
        	private Long id;
        	private String name;
        
        	// 같은 임베디드 타입을 사용한다.
        	@Embededd
        	Address homeAddress;
        	@Embededd
        	Address companyAddress;
        
        	//생성자, getter, setter 생략
        }
        ```
        
        - 발생하는 문제
            - 테이블에 매핑하는 칼럼명이 중복된다.
            - `homeAddress` 에서 사용하는 칼럼명과 `companyAddress` 에서 사용하는 칼럼명이 겹친다.
            - **이때 `@AttributeOverride` 를 통해 매핑정보를 재정의해야 한다.**
    
    - 임베디드 타입을 재정의할 때
        
        ```java
        @Entity
        public class Member {
        
        	@Id @GeneratedValue
        	private Long id;
        	private String name;
        
        	@Embededd
        	Address homeAddress;
        
        	@Embededd
        	@AttributeOverrides({
        		@AttributeOverride(name="city", column=@Column(name = "COMPANY_CITY")),
        		@AttributeOverride(name="street", column=@Column(name = "COMPANY_STREET")),
        		@AttributeOverride(name="zipcode", column=@Column(name = "COMPANY_ZIPCODE"))
        	})
        	Address companyAddress;
        
        	//생성자, getter, setter 생략
        }
        ```
        
        - **`homeAddress` 의 각 필드와 매핑되는 DB 테이블의 칼럼**
            - `city` (필드) ↔ `CITY` (칼럼)
            - `street` (필드) ↔ `STREET` (칼럼)
            - `zipcode` (필드) ↔ `ZIPCODE` (칼럼)
        - **`companyAddress` 와 매핑되는 DB 테이블의 칼럼**
            - `city` (필드) ↔ `COMPANY_CITY` (칼럼)
            - `street` (필드) ↔ `COMPANY_STREET` (칼럼)
            - `zipcode` (필드) ↔ `COMPANY_ZIPCODE` (칼럼)

> 하지만 `@AttributeOverride`를 사용하면 애너테이션을 너무 많이 사용해서, 엔티티 코드가 지저분해진다.  
따라서 이런 상황을 최대한 피하자.

<br/>

### 임베디드 타입과 null

- 임베디드 타입이 `null` 이면 매핑한 칼럼의 값은 모두 `null`이 된다.
- 예시
    
    ```java
    member.setAddress(null); //Address는 임베디드 타입
    em.persist(member);
    ```
    
    - 이때, 회원 테이블의 주소와 관련된 `CITY` , `STREET` , `ZIPCODE` 칼럼의 값은 모두 `null` 이 된다.

<br/><br/>

## 값 타입과 불변 객체

### 값 타입 공유 참조시 문제

- **임베디드 타입과 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.**
- 예시 코드
    
    ```java
    member1.setHomeAddress(new Address("Old City"));
    Address address = member1.getHomeAddress();
    
    address.setCity("New City"); //임베디드 타입의 값 수정
    member2.setHomeAddress(address); //다른 엔티티에 포함시킴
    ```
    
    - 기대: **회원2의 주소만 "New City"로 변경**
    - 실제 동작: **회원1과 회원2의 주소가 모두 "New City"로 변경**
    - 이유: **회원1과 회원2가 같은 `address` 인스턴스를 참조하기 때문**
        - 이 경우, 영속성 컨텍스트는 회원1과 회원2 둘 다 `city` 속성이 변경된 것으로 판단해서, **회원1와 회원2에 대해 각각 UPDATE SQL을 실행**한다.
        - 이것을 **부작용(Side Effect)** 이라고 한다.

<br/>

- 부작용(Side Effect) 해결방법
    - 값을 복사해서 사용해야 한다.

<br/>

### 값 타입 복사

- 값(임베디드 타입의 인스턴스)을 복사해서 사용하면, 부작용(Side Effect)를 방지할 수 있다.

<br/>

- 예시 코드
    
    ```java
    member1.setHomeAddress(new Address("Old City"));
    Address address = member1.getHomeAddress();
    
    //회원1의 address 값을 복사해서, 새로운 newAddress 값 생성
    Address newAddress = address.clone();
    
    newAddress.setCity("New City");
    member2.setHomeAddress(newAddress);
    ```
    
    - `clone()` 메서드가 자신을 복사해서 반환하도록 구현되었다고 하자.
    - 그럼 이때, `address` 인스턴스와 `newAddress` 인스턴스는 서로 다른 객체이다.
    - 따라서 이 경우, 영속성 컨텍스트는 회원2의 주소만 변경된 것으로 판단하여 **회원2에 대해서만 UPDATE SQL을 실행**한다.

> 기본 값 타입(`int` , `Long` 등)은 원래 값을 복사하여 전달하므로 부작용의 위험이 없다.
> 

<br/>

### 값 타입 복사의 문제

- 값 타입을 복사하여 사용하면, '부작용이 있는 공유 값 타입 사용'을 부작용 없이 대체할 수 있다.
- **하지만 이러한 방식을 사용하도록 강제할 수 있는 방법은 없다.**
    - 다른 개발자가 임의로 '공유 값 타입'을 사용할 수 있다.
- 따라서 '값 타입 복사'를 하여 부작용을 막는 것이 아니라, **애초에 '값 타입 객체'의 값을 수정하지 못하게 하면 된다.**
    - 즉 `Address` 객체의 수정자(setter) 메서드를 제거하거나 private으로 선언한다면, 이미 생성된 객체의 값을 수정할 수 없다.
    - 객체의 값을 수정할 수 없다면, 값 타입 객체가 공유되더라도 아무런 문제가 없다.

<br/>

### 불변 객체

- **객체를 불변하게 만들면 값을 수정할 수 없으므로, 부작용을 원천 차단할 수 있다.**
- **따라서 값 타입은 될 수 있으면 불변 객체로 설계해야 한다.**
    - 불변 객체: 한 번 만들면 절대 변경할 수 없는 객체

<br/>

- 예시 코드: 주소 불변 객체
    
    ```java
    @Embeddable
    public class Address {
    
    	private String city;
    
    	//기본 생성자: 임베디드 타입에서 필수적으로 필요하다.
    	public Address() {}
    
    	//생성자로 초기값 설정
    	public Address(String city) {
    		this.city = city;
    	}
    
    	//접근자(getter)는 노출한다.
    	public void getCity() {
    		return city;
    	}
    
    	//수정자(setter)는 숨긴다.
    	private void setCity(String city) {
    		this.city = city;
    	}
    }
    ```
    
    > 교재에서는 아예 setter를 만들지 않았지만, 그럴 경우 IDE 상에서 오류가 발생한다.  
    Hibernate가 setter를 사용하기 때문이다. 따라서, setter를 private으로 숨겨주자.  
    [참고 자료](https://stackoverflow.com/questions/2676689/does-hibernate-always-need-a-setter-when-there-is-a-getter)
    > 

- 위 예시 코드처럼 private으로 setter를 숨기면, 외부에서 값을 수정하기 위해선 new 연산자를 통해 값 타입 객체를 새로 생성해야만 한다.
- 이 방식을 통해, 부작용을 막을 수 있다.

<br/><br/>

## 값 타입의 비교

값 타입간 비교를 하는 상황에서 필요한 것이 바로, `equals` 와 `hashCode` 메서드이다. 자세한 것은 아래 게시글을 참고하자.  

[equals 와 hashCode 메서드에 대하여](https://taegyunwoo.github.io/java/Java_EqualsAndHashcode)

<br/><br/>

## 값 타입 컬렉션

### 값 타입 하나 이상 저장하기

- 값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 아래 애너테이션을 사용하면 된다.
    - `@ElementCollection`
    - `@CollectionTable`

바로 예시 코드를 통해, 알아보자.

<br/>

### 예시 코드

- 회원 엔티티 클래스
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	private Long id;
    
    	@Embededd
    	Address homeAddress;
    
    	@ElementCollection
    	@CollectionTable(
    		name = "FAVORITE_FOODS",
    		joinColumns = @JoinColumn(name = "MEMBER_ID")
    	)
    	@Column(name = "FOOD_NAME")
    	private Set<String> favoriteFoods = new HashSet<String>();
    
    	@ElementCollection
    	@CollectionTable(
    		name = "ADDRESS",
    		joinColumns = @JoinColumn(name = "MEMBER_ID")
    	)
    	private List<Address> addressHistory = new ArrayList<Address>();
    
    	//생성자, getter, setter 생략
    }
    ```
    

- 주소 임베디드 값 타입 클래스
    
    ```java
    @Embeddable
    public class Address {
    
    	@Column
    	String city;
    	String street;
    	String state;
    
    	//기본 생성자 생략
    }
    ```

- 값 타입 컬렉션 ERD
    
    ![Untitled](/assets/img/2021-11-21-JPA_Value_Type/Untitled%202.png)
    
    - **'값 타입 컬렉션을 저장하는 테이블'은 모든 칼럼을 묶어서 기본키로 구성해야 한다.**

<br/>

- 상세 설명
    - `favoriteFoods` 는 기본값 타입인 `String` 을 컬렉션으로 가진다.
        - 관계형 DB 테이블은 칼럼 안에 컬렉션을 포함할 수 없다.
        - 즉 `MEMBER` 테이블에 List형 데이터를 저장할 수 없다.
    - **따라서 위 그림(ERD)처럼 별도의 테이블을 추가하고, `CollectionTable`을 사용해서 추가한 테이블을 매핑해야 한다.**
        - 또한 테이블 `FAVORITE_FOOD` 에서 사용되는 칼럼이 `FOOD_NAME` 하나이므로, 엔티티 클래스의 `Set<FavoriteFood> favoriteFoods` 속성에서 `@Column` 을 통해 칼럼명을 지정할 수 있다.
    - `@CollectionTable` 생략 시, 기본값을 사용해서 매핑한다. 기본값 형태는 아래와 같다.
        - 엔티티이름_컬렉션속성이름
        - 예시
            - `Member` 엔티티의 `addressHistory` 는 아래 테이블과 매핑한다.
            - `Member_addressHistory` 테이블

<br/>

### 값 타입 컬렉션 사용

예시 코드를 통해, 값 타입 컬렉션을 어떻게 사용하는지 알아보자.

```java
Member member = new Member();

//임베디드 값 타입
member.setHomeAddress(new Address("통영", "몽돌해수욕장", "660-123"));

//기본값 타입 컬렉션에 등록
member.getFavoriteFoods().add("짬뽕");
member.getFavoriteFoods().add("짜장");
member.getFavoriteFoods().add("탕수육");

//임베디드 값 타입 컬렉션에 등록
member.getAddressHistory().add(new Address("서울", "강남", "123-123"));
member.getAddressHistory().add(new Address("서울", "강북", "000-000"));

em.persist(member);
```

- 상세 설명
    - **마지막에 `member` 엔티티만 영속화했다.**
        - **JPA는 이때 `member` 엔티티의 값 타입도 함께 저장한다.**
    - 실제 DB에 실행되는 INSERT SQL
        - `member` : INSERT SQL 1번
        - `member.homeAddress` : 컬렉션이 아닌 임베디드 값 타입이므로 회원 테이블을 저장하는 SQL에 포함된다.
        - `member.favoriteFoods` : INSERT SQL 3번
            - `FAVORITE_FOOD` 테이블에 총 3번의 INSERT문이 실행된다.
        - `member.addressHistory` : INSERT SQL 2번
            - `ADDRESS` 테이블에 총 2번의 INSERT문이 실행된다.

> 값 타입 컬렉션은 '영속성 전이(cascade)'와 '고아 객체 제거' 기능을 필수로 가진다고 볼 수 있다.
> 

<br/>

- 값 타입 컬렉션의 페치 전략
    - 기본적으로 `LAZY` 전략(지연 로딩)을 사용한다.
    
    ```java
    @ElementCollection(fetch = FetchType.LAZY)
    ```
    
    - 페치 전략에 대해선 [이전 게시글을 참고](https://taegyunwoo.github.io/jpa/JPA_Proxy)하자.

- 지연 로딩으로 모두 설정했을 때의 값 타입 조회 예시 코드
    
    ```java
    // **상세설명: 1**
    Member member = em.find(Member.class, 1L);
    
    // **상세설명: 2**
    // EAGER
    Address homeAddress = member.getHomeAddress();
    
    // **상세설명: 3**
    //LAZY: favoriteFoods에는 (컬렉션용) 프록시 객체가 들어간다.
    Set<String> favoriteFoods = member.getFavoriteFoods();
    
    //이때 DB에서 실제로 조회된다.
    for (String favoriteFood : favoriteFoods) {
    	System.out.println("Favorite Food: " + favoriteFood);
    }
    
    // **상세설명: 4**
    //LAZY: addressHistory에는 (컬렉션용) 프록시 객체가 들어간다.
    List<Address> addressHistory = member.getAddressHistory();
    
    //이때 DB에서 실제로 조회된다.
    addressHistory.get(0);
    ```
    
    1. `member`
        - 회원만 조회한다.
        - 이때 임베디드 값 타입인 `homeAddress`도 함께 조회한다.
        - SELECT SQL을 1번 호출한다.
    2. `member.homeAddress`
        - 1번에서 회원을 조회할 때, 같이 조회해 둔다.
    3. `member.favoriteFoods`
        - LAZY로 설정했다.
        - 실제 컬렉션을 사용할 때, SELECT SQL을 1번 호출한다.
    4. `memberaddressHistory`
        - LAZY로 설정했다.
        - 실제 컬렉션을 사용할 때, SELECT SQL을 1번 호출한다.

<br/>

- 값 타입 컬렉션 수정 예시 코드
    
    ```java
    Member member = em.find(Member.class, 1L);
    
    // 1. 임베디드 값 타입 수정
    member.setHomeAddress(new Address("새로운도시", "신도시1", "123456"))
    
    // 2. 기본값 타입 컬렉션 수정
    Set<String> favoriteFoods = member.getFavoriteFoods();
    favoriteFoods.remove("탕수육"); //제거후
    favoriteFoods.add("치킨"); //추가
    
    // 3. 임베디드 값 타입 컬렉션 수정
    List<Address> addressHistory = member.getAddressHistory();
    addressHistory.remove(new Address("서울", "기존주소", "123-123")); //제거 후
    addressHistory.add(new Address("새로운도시", "새로운주소", "000-000")); //추가
    ```
    
    1. **임베디드 값 타입 수정**
        - `homeAddress` 임베디드 값 타입은 `MEMBER` 테이블과 매핑했다.
        - 그러므로 `MEMBER` 테이블만 UPDATE 한다.
        - 즉 `Member` 엔티티를 수정하는 것과 같다고 할 수 있다.
    2. **기본값 타입 컬렉션 수정**
        - 탕수육을 치킨으로 변경하려면, 탕수육을 제거하고 치킨을 추가해야 한다.
        - 왜냐하면 자바의 String 타입은 수정할 수 없기 때문이다.
            - `"탕수육".setString("치킨")` 과 같은 코드는 존재하지 않는다.
    3. **임베디드 값 타입 컬렉션 수정**
        - 값 타입은 불변해야 한다.
        - 그러므로 컬렉션에서 기존 주소를 삭제하고 새로운 주소를 등록했다.

<br/>

### 값 타입 컬렉션의 제약사항

- **엔티티는 식별자가 있으므로** 엔티티의 값을 변경해도, 식별자를 통해 DB에 저장된 원본 데이터를 찾아 변경할 수 있다.
- **값 타입은 식별자 개념 자체가 없으므로** 값을 변경해버리면, DB에 저장된 원본 데이터를 찾기 어렵다.
- **값 타입 컬렉션의 제약사항**
    - 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관된다.
    - 따라서 여기에 보관된 값 타입의 값이 변경되면, DB에 있는 원본 데이터를 찾기 어렵다.
    - 이러한 문제를 해결하기 위해, JPA 구현체들은 값 타입 컬렉션에 변경 사항이 발생하면 아래와 같은 방식으로 변경 사항을 반영한다.
        - **값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고, 현재 값 타입 컬렉션 객체에 있는 모든 값을 DB에 다시 저장한다.**

<br/>

- 예시 코드
    - 식별자가 100번인 회원이 관리하는 주소 값 타입 컬렉션을 변경하기
    - 이때 실행되는 SQL
        
        ```sql
        // 1.
        DELETE FROM ADDRESS WHERE MEMBER_ID=100;
        
        // 2.
        INSERT INTO ADDRESS (MEMBER_ID, CITY, STREET, ZIPCODE) VALUES (100, ...);
        INSERT INTO ADDRESS (MEMBER_ID, CITY, STREET, ZIPCODE) VALUES (100, ...);
        ```
        
        1. 100번 회원의 주소 값 타입 컬렉션의 모든 요소 제거
        2. 현재 100번 회원의 주소 값 타입 컬렉션의 모든 요소 추가
    - 총 3개의 SQL이 실행된다.

<br/>

- **'값 타입 컬렉션이 매핑된 테이블'에 데이터가 많다면, 일대다 관계를 고려하자!**
    - 일대다 관계 대신 값 타입 컬렉션을 사용한다면
        - **너무 많은 SQL문이 실행될 수 있다.**
        - 또한, '값 타입 컬렉션을 저장하는 테이블'은 모든 칼럼을 묶어서 기본키로 구성해야 한다. 따라서 **기본키 제약조건으로 인해ㅡ 칼럼에 null을 입력할 수 없다.**
    - 따라서 새로운 엔티티를 만들어서 일대다 관계로 설정하는 것을 추천한다.
        
        > 추가로, 영속성 전이와 고아 객체 제거 기능을 적용하면 값 타입 컬렉션처럼 사용할 수 있다.
        > 
    - 예시 코드
        - 임베디드 값 타입 대체 엔티티 클래스
            
            ```java
            //임베디드 값 타입 Address 대신,
            //엔티티 클래스 AddressEntity 사용
            
            @Entity
            public class AddressEntity {
            
            	@Id @GeneratedValue
            	private Long id;
            
            	@Embedded
            	Address address;
            
            }
            ```
            
        - 엔티티 클래스
            
            ```java
            @Entity
            public class Member {
            
            	@Id @GeneratedValue
            	private Long id;
            
            	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
            	@JoinColumn(name = "MEMBER_ID")
            	private List<AddressEntity> addressHistory = new ArrayList<AddressEntity>();
            
            	//생성자, getter, setter 생략
            }
            ```

<br/><br/>

## 정리

### 엔티티 타입의 특징

- 식별자(`@Id`)가 있다.
- 생명 주기가 있다.
- 공유할 수 있다.

<br/>

### 값 타입의 특징

- 식별자가 없다.
- 생명 주기를 엔티티에 의존한다.
- 공유하지 않는 것이 안전하다.
    - 대신 값을 복사해서 사용해야 한다.
    - 오직 하나의 주인만이 관리해야 한다.
    - 값 타입 객체를 불변 객체로 만드는 것이 안전하다.

<br/>

### 주의사항

- 엔티티와 값 타입을 혼동해서, **엔티티를 값 타입으로 만들면 안된다는 것에 유의**하자!

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>김영한, 『자바 ORM 표준 JPA 프로그래밍』, 에이콘</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>