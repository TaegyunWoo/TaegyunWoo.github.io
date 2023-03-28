---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 테스트 순서"
date:   2021-09-16 22:40:00 
lastmod : 2021-09-16 22:40:00
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
    - [[JUnit 5] 태깅과 필터링](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_Tagging)
    - [[JUnit 5] 커스텀 태그](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_CustomTag)
    - [[JUnit 5] 테스트 반복](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_RepeatTest)
    - [[JUnit 5] 테스트 인스턴스](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_TestInstance)

<br/><br/>

# 테스트 순서 설정하기

## 개요

### 기본 전략

- 기본적으로 실행할 테스트 메서드는 특정한 순서에 의해 실행되지만, 어떻게 그 순서를 정하는지는 의도적으로 분명히 하지 않는다.
- **즉 테스트 메서드의 순서는 알 수 없다.**
- 왜냐하면, 테스트 메서드는 서로 독립적으로 실행되며 의존되지 않기 때문이다.
- 하지만, 경우에 따라 특정 순서대로 테스트를 실행하고 싶을 때도 있다.

    > ex) 회원가입 → 로그인 → 글쓰기

- 이런 경우, 아래 방법을 사용하면 된다.

<br/><br/>

## 테스트 순서 설정

### `@TestMethodOrder` 애너테이션

- 테스트 메서드의 실행 순서가 보장되어야하는 경우, `@TestMethodOrder` 애너테이션을 클래스 레벨에 적용하여 실행 순서를 보장할 수 있다.
- 보통 이런 경우, 테스트 메서드당 인스턴스를 생성하면 안된다.
- 따라서, `@TestInstance` 애너테이션도 함께 사용한다.

    > [관련 포스팅](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_TestInstance)

<br/>

### 예시

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class StudyTest {
	
	@Test
	@Order(1)
	void test1() {
		System.out.println("test1");
	}

	@Test
	@Order(2) // 두번째로 실행될 테스트
	void test2() {
		System.out.println("test2");
	}

	@Test
	@Order(3) // 세번째로 실행될 테스트
	void test3() {
		System.out.println("test3");
	}
}
```

<br/>

### 결과

![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_TestSequence/Untitled%2014.png)

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*