---
category: Test-Framework
tags: [JUnit5, 개요]
title: "[JUnit 5] 개요 및 기본 애너테이션"
date:   2021-09-15 15:30:00 
lastmod : 2021-09-15 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

# JUnit 5 개요

## JUnit 5 모듈

![Untitled](/assets/img/2021-09-15-TestFramework_JUnit5_SummaryAndBasicAnnotation/Untitled.png)

<br/>

### JUnit Platform

- 테스트를 실행해주는 런처를 제공한다.
- TestEngine API를 제공한다.

<br/>

### Jupiter

- TestEngine API 구현체이다. (JUnit Platform 구현체이다.)
- **JUnit 5를 제공한다.**
- **스프링부트 2.2 이상의 버전으로 프로젝트 생성시, 자동으로 의존성 추가된다.**
    - `spring-boot-starter` 에 포함되어 있다.

<br/>

### Vintage

- TestEngine API 구현체이다. (JUnit Platform 구현체이다.)
- **JUnit 4와 3를 지원한다.**

<br/><br/><br/>

# 기본 애너테이션

## 기본 애너테이션 종류

JUnit 5가 제공하는 기본 애너테이션은 다음과 같다.

- `@Test`
- `@BeforeAll`
- `@AfterAll`
- `@BeforeEach`
- `@AfterEach`
- `@Disabled`
- `@DisplayName`

<br/><br/>

## 전체 예시 도메인

- 애너테이션 설명을 하기 전, 예시 코드에 대한 기본 전제 코드를 설명하겠다.

> 자세한 것은 Github 참고 (최상단 내용 링크 참고)

<br/>

### `Study` 클래스

```java
public class Study {

    private StudyStatus studyStatus = StudyStatus.DRAFT;
    private int limit;
    private String name;

    public Study() {
    }

    public Study(int limit) {
        if (limit < 0) {
            throw new IllegalArgumentException("limit은 0보다 작을 수 없다.");
        }
        this.limit = limit;
    }

    public Study(int limit, String name) {
        this.limit = limit;
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public StudyStatus getStudyStatus() {
        return this.studyStatus;
    }

    public int getLimit() {
        return limit;
    }

    @Override
    public String toString() {
        return "Study{" +
                "studyStatus=" + studyStatus +
                ", limit=" + limit +
                ", name='" + name + '\'' +
                '}';
    }
}
```

<br/>

### `StudyStatus` ENUM 클래스

```java
public enum StudyStatus {
    DRAFT, STARTED, ENDED, OPENED
}
```

<br/><br/>

## `@BeforeAll`

### 기능

- 전체 테스트 메서드가 실행되기 전에, **단 한번만 실행**된다.
- 적용할 메서드는 **static 으로 선언**해야 한다.
- 적용할 메서드의 **반환 값은 void**이어야 한다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {
	@BeforeAll
  static void beforeAll() {
    System.out.println("StudyTest.beforeAll");
  }

	@Test
  void create() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }
}
```

<br/><br/>

## `@AfterAll`

### 기능

- 전체 테스트 메서드가 실행되고 난 후에, **단 한번만 실행**된다.
- 적용할 메서드는 **static 으로 선언**해야 한다.
- 적용할 메서드의 **반환 값은 void**이어야 한다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {
	@AfterAll
  static void afterAll() {
    System.out.println("StudyTest.afterAll");
  }

	@Test
  void create() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }
}
```

<br/><br/>

## `@BeforeEach`

### 기능

- 각 테스트 메서드가 실행되기 전, 각 테스트 메서드마다 실행된다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {

	@BeforeEach
  void beforeEach() {
    System.out.println("StudyTest.beforeEach");
  }

	@Test
  void create() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }
}
```

<br/>

## `@AfterEach`

### 기능

- 각 테스트 메서드가 실행되고 난 후, 각 테스트 메서드마다 실행된다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {

	@AfterEach
  void afterEach() {
    System.out.println("StudyTest.afterEach");
  }

	@Test
  void create() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }
}
```

<br/><br/>

## `@Disabled`

### 기능

- 해당 테스트 메서드를 비활성화한다.
- 전체 테스트 수행시 제외된다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {

	@Test
  void create1() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }

	@Disabled
	@Test
  void create2() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }
}
```

<br/><br/>

## `@DisplayName`

### 기능

- 어떤 테스트인지 테스트 이름을 보다 쉽게 표현할 수 있는 방법을 제공하는 애너테이션이다.
- `@DisplayNameGeneration` 대신 권장되는 애너테이션이다.

<br/>

### 예시 코드

```java
//import 생략

public class StudyTest {

	@DisplayName("Study 객체 생성")
	@Test
  void create1() {
    System.out.println("StudyTest.create1\n");
    Study study = new Study();
    assertNotNull(study);
  }

}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*