---
category: Test-Framework
tags: [JUnit5]
title: "[JUnit 5] 시간에 따른 테스트"
date:   2021-09-15 17:00:00 
lastmod : 2021-09-15 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**
- 이전 게시글
    - [[JUnit 5] 개요 및 기본 애너테이션](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation)
    - [[JUnit 5] 기본 Assertions](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_BasicAssertions)

<br/><br/>

# 시간 측정 테스트하기

## `assertTimeout` 메서드

### 메서드 형식

`assertTimeout( Duration.ofMillis(밀리초), () - {측정할 코드})`

<br/>

### 기능

- 해당 람다식이 모두 끝난 후, 수행시간이 `Duration.ofMillis(밀리초)` 보다 길다면 테스트에 실패한다.
- **설정한 시간이 지나도 람다식에 작성한 코드가 모두 완료될 때까지 기다린다.**

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);

		assertTimeout(Duration.ofMillis(1000), () -> {
			//시간을 측정할 코드
		});

	}
 
}
```

<br/><br/>

## `assertTimeoutPreemptively` 메서드

### 메서드 형식

`assertTimeoutPreemptively( Duration.ofMillis(밀리초), () - {측정할 코드})`

<br/>

### 기능

- 해당 람다식이 모두 끝난 후, 수행시간이 `Duration.ofMillis(밀리초)` 보다 길다면 테스트에 실패한다.
- **설정한 시간이 지나면, 람다식에 작성한 코드가 완료되지 않아도 종료된 후 테스트가 중단된다.**

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);

		assertTimeoutPreemptively(Duration.ofMillis(1000), () -> {
			//시간을 측정할 코드
		});

	}
 
}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*