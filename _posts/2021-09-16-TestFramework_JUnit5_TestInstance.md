---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 테스트 인스턴스"
date:   2021-09-16 22:30:00 
lastmod : 2021-09-16 22:30:00
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

<br/><br/>

# 테스트 인스턴스 조작

## 개요

### JUnit 기본 동작 전략

- JUnit은 **테스트 메서드마다 테스트 인스턴스를 새로 만든다.**
- 왜냐하면, 테스트 메서드를 독립적으로 실행하여 예상치 못한 부작용을 방지하기 위함이다.
- 이러한 전략 방식을 JUnit 5에서 변경할 수 있다.

<br/>

### `@TestInstance(Lifecyle.PER_CLASS)`

- `@TestInstance(Lifecyle.PER_CLASS)` 애너테이션을 클래스 레벨에 설정할 수 있다.
- 해당 애너테이션을 적용하면, **테스트 클래스당 인스턴스를 하나만 만들어서 사용** 한다.

<br/><br/>

## 적용 예시

### 테스트 클래스

- `test1` , `test2` , `test3` 모두 하나의 인스턴스의 메서드로 실행된다.

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class StudyTest {
	
	@Test
	void test1() { ... }

	@Test
	void test2() { ... }

	@Test
	void test3() { ... }
}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*