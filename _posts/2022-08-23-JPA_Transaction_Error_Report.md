---
category: JPA
tags: [JPA, Hibernate]
title: "[JPA] 엔티티 연관관계·트랜잭션 관련 오류와 해결방법"
date:   2022-08-23 12:10:00 
lastmod : 2022-08-23 12:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# JPA 엔티티 연관관계·트랜잭션 관련 오류와 해결방법

## 개요

- 최근 SWMaestro 과정에서 개발을 하던 중 문제가 발생하여, 정리하고자 글을 작성한다.
- 사용자 인가 처리를 위해 토큰 저장 관련 부분에서 문제가 발생하였다.
    - 참고로 토큰을 일단 관계형 DB에 다루기로 했다.

### 문제 요약

나는 아래와 같이 동작하는 코드를 작성하려 했다.

1. 로그인 시, Access·Refresh 토큰을 발급하고 DB에 저장
2. 재 로그인 시, 기존의 Access·Refresh 토큰을 제거하고 DB에 새로운 토큰 저장

하지만 아래와 같이 동작했다.

1. 로그인 시, Access·Refresh 토큰을 발급하고 DB에 저장
2. 재 로그인 시, 기존의 Access·Refresh 토큰이 유지되고 DB에 새로운 토큰도 저장

<br/><br/>

## 엔티티 관계

문제가 생긴 엔티티 관계는 아래와 같다.

### `User` 엔티티 클래스

```java
public class User {
  @Id @GeneratedValue
  private long id;
  
  //양방향 설정 및 Cascade 설정
  @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
  private AccessToken accessToken;

  //양방향 설정 및 Cascade 설정
  @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
  private RefreshToken refreshToken;

  //편의 메서드
  public void registerRefreshToken(RefreshToken refreshToken) {
    this.refreshToken = refreshToken;
    if (refreshToken.getUser() != this) {
      refreshToken.associateUser(this);
    }
  }

  //편의 메서드
  public void registerAccessToken(AccessToken accessToken) {
    this.accessToken = accessToken;
    if (accessToken.getUser() != this) {
      accessToken.associateUser(this);
    }
  }
  
}
```

### `AccessToken` 엔티티 클래스

```java
@Builder
public class RefreshToken {
  @Id @GeneratedValue
  private long id;
  
  @Column
  private String tokenValue;
  
  @OneToOne(fetch = FetchType.LAZY)
  private User user;

  //양방향 설정 및 Cascade 설정
  @OneToOne(mappedBy = "refreshToken", cascade = CascadeType.ALL)
  private AccessToken accessToken;

  //편의 메서드
  public void associateAccessToken(AccessToken accessToken) {
    this.accessToken = accessToken;
    if (accessToken.getRefreshToken() != this) {
      accessToken.associateRefreshToken(this);
    }
  }

  //편의 메서드
  public void associateUser(User user) {
    this.user = user;
    if (user.getRefreshToken() != this) {
      user.registerRefreshToken(this);
    }
  }
  
}
```

### `RefreshToken` 엔티티 클래스

```java
@Builder
public class RefreshToken {
  @Id @GeneratedValue
  private long id;
  
  @Column
  private String tokenValue;
  
  @OneToOne(fetch = FetchType.LAZY)
  private User user;

  *//양방향 설정 및 Cascade 설정*
  @OneToOne(mappedBy = "refreshToken", cascade = CascadeType.ALL)
  private AccessToken accessToken;

  *//편의 메서드*
  public void associateAccessToken(AccessToken accessToken) {
    this.accessToken = accessToken;
    if (accessToken.getRefreshToken() != this) {
      accessToken.associateRefreshToken(this);
    }
  }

  *//편의 메서드*
  public void associateUser(User user) {
    this.user = user;
    if (user.getRefreshToken() != this) {
      user.registerRefreshToken(this);
    }
  }

}
```

### 엔티티 관계

세가지 엔티티 `User` , `AccessToken` , `RefreshToken` 간의 관계를 그림으로 표현하면 아래와 같다.

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled.png)

<br/><br/>

## 서비스 메서드

### `saveTokenPairToDb(...)`

서비스 계층에 작성된 트랜잭션 메서드이다. 이 메서드를 호출할 때, 문제가 발생한다.

```java
@Transactional
public void saveTokenPairToDb(long userId, String accessTokenValue, String refreshTokenValue) {
  tokenRepository.removeTokenPairByUserId(userId); *// 기존 토큰 제거 (동작 안함)*
  tokenRepository.saveTokenPair(userId, accessTokenValue, accessExpiration, refreshTokenValue, refreshExpiration); *//새 토큰 저장*
}
```

<br/><br/>

## 레포지토리 메서드

서비스 메서드 `saveTokenPairToDb(...)` 가 호출하는 레포지토리는 아래와 같다.

### `removeTokenPairByUserId(...)`

```java
public void removeTokenPairByUserId(long userId) throws NoResultException {
  User user = em.find(User.class, userId);
  if (!Objects.isNull(user.getRefreshToken())) {
	  em.remove(user.getRefreshToken());
  }
  return user;
}
```

### `saveTokenPair(...)`

```java
public RefreshToken saveTokenPair(long userId, String accessValue, String refreshValue) {
  RefreshToken refreshToken;
  AccessToken accessToken;
  User user;

  user = em.find(User.class, userId);
  refreshToken = RefreshToken.builder()
      .tokenValue(refreshValue)
      .user(user)
      .build();
  accessToken = AccessToken.builder()
      .tokenValue(accessValue)
      .user(user)
      .build();
  refreshToken.associateAccessToken(accessToken);
  */* 
   * 주석 처리
   * user.registerRefreshToken(refreshToken);
   * user.registerAccessToken(accessToken);
   */*

  *//Cascade.ALL 옵션 때문에, 부모 엔티티(RefreshToken)이 영속화될 때, 자식 엔티티(Access Token)도 영속화된다.*
  em.persist(refreshToken);

  return refreshToken;
}
```

<br/><br/>

## 문제 원인

위와 같은 상태에서, 하나의 트랜잭션에서 `removeTokenPairByUserId(...)` 메서드를 호출하고 `saveTokenPair(...)` 메서드를 호출하면 어떻게 될까?

### **`removeTokenPairByUserId(...)` 호출 시**

먼저 `removeTokenPairByUserId(...)` 를 호출하면 아래와 같이 동작한다.

```java
*//1. userId 파라미터에 '1'을 전달한다.*
public void removeTokenPairByUserId(long userId) throws NoResultException {
  *//2. 영속 상태의 엔티티 'User#1'이 조회된다.*
  User user = em.find(User.class, userId);
  
  if (!Objects.isNull(user.getRefreshToken())) {
    */*
     * 3. Cascade 설정으로 인해, 엔티티 'AccessToken#1' 이 먼저 준영속 상태로 변경되고,
     * 엔티티 `RefreshToken#1`이 준영속 상태로 변경된다.
     */*
    em.remove(user.getRefreshToken());
  }
  
  return user;
}
```

이것을 그림으로 표현하면 아래와 같다.

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%201.png)

- `User#1` 을 조회하면, 연관관계가 있는 `AccessToken#4` 와 `RefreshToken#3` 이 함께 조회된다.
    - `User.accessToken` 과 `User.refreshToken` 이 모두 `fetch = FetchType.EAGER` 로 설정(default)되어 있기 때문이다.
- 또한 각 엔티티들이 서로를 참조하고 있다.

그 다음으로 아래 그림과 같이 수행된다.

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%202.png)

- `AccessToken#4` 와 `RefreshToken#3` 엔티티가 준영속상태로 변경된다.
- `User#1` 은 전혀 변경되지 않는다.
    - `User#1` 은 전혀 건드리지 않는다.
    - `User#1` **에 변화가 있을 때,** `AccessToken` 과 `RefreshToken` 을 변경시키도록 `Cascade` 설정을 했을 뿐이다.
    - 하지만 `User#1` 을 조작하는 작업을 하지 않았다.
- `AccessToken#4` **와** `RefreshToken#3` **엔티티가 준영속상태로 변경되더라도,** `User#1` **은 그대로 참조를 하고 있다.**
    - `em.remove()` 를 호출하여도 단순히 영속 상태만 변화하게 된다.
- 결과적으로 `removeTokenPairByUserId(...)` 는 전혀 문제없이 정상적으로 동작했다.
    - 실제로 영속 컨텍스트에서 기존 토큰들을 지웠다. 따라서 아래와 같은 로그가 출력되었다.

실제로 `em.remove()` 가 정상적으로 동작했는지는 아래 그림을 통해 확인해볼 수 있다.

> application.properties 에 logging.level.org.hibernate=trace 설정 시 확인할 수 있음
> 

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%203.png)

- 컨텍스트에 있는 `User` 를 로딩해서, 참조 중인 `RefreshToken` 과 `AccessToken` 을 `remove()` 한다.
- `RefreshToken.accessToken` 이 `cascade = CascadeType.ALL` 로 설정되어 있기 때문에, `AccessToken#4` 를 제거하고 `RefreshToken#3` 을 제거한다.
- 즉, 토큰 엔티티들을 모두 **준영속 상태**로 만든다.
- 문제는 `saveTokenPair(...)` 를 호출할 때 발생한다.

### `saveTokenPair(...)` 호출 시

다음으로 `saveTokenPair(...)` 를 호출하면 아래와 같이 동작한다.

```java
*/*
 * 1. 파라미터에 값 전달
 *  - userId 파라미터 = 1
 *  - accessValue 파라미터 = 새 Access Token 값
 *  - refreshValue 파라미터 = 새 Refresh Token 값
 */*
public RefreshToken saveTokenPair(long userId, String accessValue, String refreshValue) {
  RefreshToken refreshToken;
  AccessToken accessToken;
  User user;

  *//2. 기존 Persistence Context 에 영속되어 있던 User#1 을 다시 꺼내온다. (Cache Hit)*
  user = em.find(User.class, userId);
  
  *//3. 새 토큰 엔티티 생성*
  refreshToken = RefreshToken.builder()
      .tokenValue(refreshValue)
      .user(user)
      .build();
  accessToken = AccessToken.builder()
      .tokenValue(accessValue)
      .user(user)
      .build();
  refreshToken.associateAccessToken(accessToken);
  */* 
   * 주석 처리
   * user.registerRefreshToken(refreshToken);
   * user.registerAccessToken(accessToken);
   */*

  *//4. 새로운 토큰 엔티티들을 저장*
  *//Cascade.ALL 옵션 때문에, 부모 엔티티(RefreshToken)이 영속화될 때, 자식 엔티티(Access Token)도 영속화된다.*
  em.persist(refreshToken);

  return refreshToken;
}
```

바로 그림을 통해 알아보자.

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%204.png)

- 이미 Persistence Context 에 `User#1` 이 존재하므로, `em.find()` 를 호출하면 Persistence Context 에 있는 `User#1` 을 로드한다.
- **즉 DB 조회를 하지 않는다.**

이때 `em.persist(새로운 RefreshToken 엔티티)` 를 호출하면 아래 그림과 같이 변경된다.

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%205.png)

- 새로운 `RefreshToken#5` 와 `AccessToken#6` 이 Persistence Context 에 추가되었다.
    - **하지만 기존의** `User#1` **엔티티의** `refreshToken` **과** `accessToken` **필드를 전혀 수정하지 않았기 때문에, 이전 토큰 엔티티(삭제되어야 하는 엔티티)을 그대로 참조하고 있다.**
    - 새로운 `RefreshToken#5` 와 `AccessToken#6` 은 `User#1` 를 참조한다.

**따라서 서비스 메서드** `saveTokenPairToDb(...)` **의 실행이 종료되어, 트랜잭션이 commit 되면 최종적으로 아래와 같은 상태가 된다.**

![Untitled](/assets/img/2022-08-23-JPA_Transaction_Error_Report/Untitled%206.png)

- `RefreshToken#3` **과** `AccessToken#4` **가 전혀 삭제되지 않는다.**
- **왜냐하면 영속상태인** `User#1` **이 계속 참조하고 있는 엔티티이기 때문이다.**

**즉,** `em.remove()` **를 호출하여 제거를 하려면, 그와 관련있는 연관 엔티티 관계를 잘 정리해야만 한다.**

<br/><br/>

## 문제 해결

문제를 해결하려면, `User#1` 의 토큰 필드들을 정리해주면 된다.

레포지토리 메서드 `saveTokenPair(...)` 를 아래와 같이 수정하자.

### 수정된 `saveTokenPair(...)`

```java
public RefreshToken saveTokenPair(long userId, String accessValue, String refreshValue) {
  RefreshToken refreshToken;
  AccessToken accessToken;
  User user;

  user = em.find(User.class, userId);
  refreshToken = RefreshToken.builder()
      .tokenValue(refreshValue)
      .user(user)
      .build();
  accessToken = AccessToken.builder()
      .tokenValue(accessValue)
      .user(user)
      .build();
  refreshToken.associateAccessToken(accessToken);
  
  *//----추가 코드----*
  user.registerRefreshToken(refreshToken);
  user.registerAccessToken(accessToken);
  *//----추가 코드 끝----*

  *//Cascade.ALL 옵션 때문에, 부모 엔티티(RefreshToken)이 영속화될 때, 자식 엔티티(Access Token)도 영속화된다.*
  em.persist(refreshToken);

  return refreshToken;
}
```

**위처럼 편의 메서드를 사용하여, 관계를 정리하는 코드를 작성하면 문제를 해결할 수 있다.**