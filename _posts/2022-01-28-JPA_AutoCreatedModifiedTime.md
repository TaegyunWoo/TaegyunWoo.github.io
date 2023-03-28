---
category: JPA
tags: [JPA]
title: "[JPA] JPA Auditing으로 생성시간/수정시간 자동화하기"
date:   2022-01-28 14:19:00 
lastmod : 2022-01-28 14:19:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# JPA Auditing을 활용한 엔티티 생성·수정시간 자동화

## 개요

### 문제상황

- 기존에는 자바에서 날짜나 시간을 표현하기 위해, `java.util.Date` 나 `java.util.Calendar` 등을 사용했다.
- 하지만 자바 8부터 추가된 `LocalDate` 와 `LocalDateTime` 을 사용해야 한다. 그 이유는 대표적으로 아래와 같다.
    - `java.util.Date` 와 `java.util.Calendar` 는 불변객체가 아니다.
        - 즉 set 메서드가 존재한다.
        - 따라서 멀티쓰레드 환경에서 문제가 발생할 수 있다.
    - `java.util.Calendar` 는 월(month) 값 설계가 잘못되었다.
        - 10월을 나타내는 `Calendar.OCTOBER` 의 숫자 값은 ‘9’이다.
    - `java.util.Calendar` 에 int 상수 필드가 남용되어있다.
        - 2초를 더하는 코드를 예시로 보자. 이는 아래와 같다.
            
            ```java
            calendar.add(Calendar.SECOND, 2);
            ```
            
        - 만약 이때 실수로 아래와 같이 작성했다고 해보자.
            
            ```java
            calendar.add(Calendar.OCTOBER, 2);
            ```
            
        - 위 코드처럼 작성하여도 컴파일 시점에서 해당 오류를 찾아낼 수 없다.
    
    > 이외에도 많은 문제가 있다.  
    자세한 것은 [https://d2.naver.com/helloworld/645609](https://d2.naver.com/helloworld/645609) 을 참고하자.

<br/>

### 문제 해결 방안

- 이러한 문제를 해결하고자 Java8부터 추가된 것이 바로 `LocalDate` 와 `LocalDateTime` 이다.
    - 또한 Hibernate 5.2.10 버전부터 `LocalDate` 와 `LocalDateTime` 을 본격적으로 지원하기 시작했다.
    - 만약 본인이 `JPA 2.1`을 구현하는 `Hibernate 4.3.x` 를 사용한다면, [http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html](http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html) 를 참고하자.

<br/><br/>

## JPA Auditing

### JPA Auditing이란?

<br/>

> Audit: 감시하다

<br/>

- JPA Auditing 이란, Spring Data JPA에서 시간에 대해, 자동으로 값을 넣어주는 기능이다.
    - 도메인을 영속성 컨텍스트에 저장하거나 조회를 수행한 후에 UPDATE를 하는 경우, 매번 시간 데이터를 입력해 주어야 하는 번거로움을 `JPA Auditing` 기능으로 해결할 수 있다.

<br/>

### JPA Auditing 사용

- Java에서 JPA를 사용하여 도메인을 관계형 데이터베이스 테이블에 매핑할 때, 각 도메인들이 공통적으로 가지고 있는 필드·칼럼이 존재한다.
    - 대표적으로 생성일자, 수정일자 등과 같은 필드·칼럼이 존재한다.
- 따라서 `@MappedSuperclass` 를 통해, 생성일자나 수정일자와 같은 공통 필드를 묶어서 관리해야 한다.
- **즉, `JPA Auditing` 기능과 `@MappedSuperclass` 를 조합하여 사용해야 한다.**

<br/><br/>

## 엔티티 생성·수정시간 자동화 예시 코드

바로 예시 코드를 통해 알아보자.

### `BaseTimeEntity` 클래스

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {
	@CreatedDate
	private LocalDateTime createdDate;

	@LastModifiedDate
	private LocalDateTime modifiedDate;
}
```

- `@MappedSuperclass` 애너테이션
    - 따라서 `BaseTimeEntity` 클래스를 상속받는 다른 엔티티 클래스들이 해당 필드들(`createdDate` , `modifiedDate`)를 엔티티 필드로서 사용할 수 있다.
- `@EntityListeners(AuditingEntityListener.class)`
    - 영속성 메커니즘상에서 이벤트(우리의 경우, 엔티티 저장·수정)가 발생할 때 호출해줄 Listener 클래스를 지정한다.
    - 즉, `BaseTimeEntity` 클래스에 `Auditing` 기능을 포함시킨다.
- `@CreatedDate`
    - Entity가 생성되어 저장될 때 시간이 자동 저장된다.
- `@LastModifiedDate`
    - 조회한 Entity의 값을 변경할 때 시간이 자동 저장된다.

<br/>

### `Post` 엔티티 클래스

```java
@Entity
public class Post extends BaseTimeEntity {
	//...
}
```

- `BaseTimeEntity` 를 상속받아 `createdTime` 필드와 `modifiedTime` 필드를 사용한다.

<br/>

### `Application` 클래스

> 애플리케이션 구동 클래스 (`main` 메서드 존재 클래스)

```java
@EnableJpaAuditing //JPA Auditing 활성화
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
} 
```

<br/><br/>

## REFERENCE
- [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#auditing](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#auditing)
- [https://it-mesung.tistory.com/195](https://it-mesung.tistory.com/195)
- [http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html](http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html)
- [https://webcoding-start.tistory.com/53](https://webcoding-start.tistory.com/53)

<br><br>