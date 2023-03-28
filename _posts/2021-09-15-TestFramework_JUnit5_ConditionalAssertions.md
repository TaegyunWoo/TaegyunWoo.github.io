---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 조건에 따른 테스트"
date:   2021-09-15 17:30:00 
lastmod : 2021-09-15 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**
- 이전 게시글
    - [[JUnit 5] 개요 및 기본 애너테이션](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation)
    - [[JUnit 5] 기본 Assertions](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_BasicAssertions)
    - [[JUnit 5] 시간에 따른 테스트](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_TimeAssertions)

<br/><br/>

# 조건에 따라 테스트 실행하기

## `assertTrue` 메서드

### 메서드 형식

`assertTrue(조건식)`

<br/>

### 기능

- 조건이 true면, 테스트에 성공한다.
- 조건이 false라면, 테스트에 실패한다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);
		assertTrue(study.getLimit() > 0);
		// 나머지 테스트 코드
	}
 
}
```

<br/><br/>

## `assumingThat` 메서드

### 메서드 형식

`assumingThat(조건식, 테스트람다식)`

<br/>

### 기능

- 조건이 true면, 람다식(테스트)를 수행한다.
- 조건이 false라면, 람다식(테스트)를 수행하지 않는다. **(테스트에 실패하지는 않는다.)**

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);
		assumingThat(study.getLimit()>0, () -> {
			//조건 만족시 수행할 테스트 코드
		});
	}
 
}
```

<br/><br/>

## `@EnabledOnOS` 애너테이션

### 애너테이션 형식

`@EnabledOnOs(OS.WINDOWS)`

<br/>

### 기능

- 속성으로 전달한 해당 OS에서 테스트를 실행할때, 테스트 메서드를 수행한다.
- 해당 OS가 아닌 경우, 테스트 메서드를 수행하지 않는다.

<br/>

### `@DisabledOnOS` 애너테이션

- 말 그대로, `@EnabledOnOS` 의 반대 개념이다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@EnabledOnOS(OS.WINDOWS)
	@Test
  void create() {
		//테스트 코드
	}
 
}
```

<br/>

## `@EnabledOnJre` 애너테이션

### 애너테이션 형식

`@EnabledOnJre({JRE.JAVA_8, JRE.JAVA_11, ... })`

<br/>

### 기능

- 속성으로 전달한 해당 자바 버전에서 테스트를 실행할때, 테스트 메서드를 수행한다.
- 해당 자바 버전이 아닌 경우, 테스트 메서드를 수행하지 않는다.

<br/>

### `@DisabledOnJre` 애너테이션

- 말 그대로, `@EnabledOnJre` 의 반대 개념이다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@EnabledOnJre({JRE.JAVA_8, JRE.JAVA_11})
	@Test
  void create() {
		//테스트 코드
	}
 
}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*