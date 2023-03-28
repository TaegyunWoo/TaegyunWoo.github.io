---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 태깅과 필터링"
date:   2021-09-16 16:30:00 
lastmod : 2021-09-16 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**
- 이전 게시글
    - [[JUnit 5] 개요 및 기본 애너테이션](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation)
    - [[JUnit 5] 기본 Assertions](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_BasicAssertions)
    - [[JUnit 5] 시간에 따른 테스트](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_TimeAssertions)
    - [[JUnit 5] 조건에 따른 테스트](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_ConditionalAssertions)

<br/>

# 태깅과 필터링 활용

## 개요

### 태깅이란?

- 테스트 그룹을 만드는 것이다.

<br/>

### 필터링이란?

- 태깅을 통해 만든 테스트 그룹을 특정하여 테스트를 실행하는 것이다.

<br/>

## `@Tag` 애너테이션

### 애너테이션 형식

`@Tag("태그명")`

<br/>

### 기능

- 테스트 메서드에 해당 태그를 설정한다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Tag("myTag")
	@Test
  void create() {
		//테스트 코드
	}
 
}
```

<br/><br/>

## IntelliJ에서 필터링 하기

- Gradle 사용시 JUnit 항목이 나타나지 않는 경우가 있다.
- 따라서 직접 만들어야한다. 만드는 방법은 아래와 같다.

<br/>

### 1. Edit Configurations

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_Tagging/Untitled%201.png)

- 위 그림처럼 Edit Configurations 항목을 클릭한다.

<br/>

### 2. JUnit 항목 추가

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_Tagging/Untitled%202.png)

- 위 사진처럼 JUnit 항목이 존재하지 않는다.
- 따라서, 따로 생성해주어야 한다.

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_Tagging/Untitled%203.png)

- JUnit을 추가한다.

<br/>

### 3. 이름, Module, -cp 설정

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_Tagging/Untitled%204.png)

- 해당 항목들을 참고하여 설정한다.
- 설정 후, OK를 클릭하여 적용한다.

<br/>

### 4. 특정 태그 테스트만 실행하기

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_Tagging/Untitled%205.png)

- 위 그림처럼, 우리가 설정한 이름이 출력된다.
- 해당 항목을 선택한 상태로 Run하게 되면, 해당 태그의 테스트만 실행된다.

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*