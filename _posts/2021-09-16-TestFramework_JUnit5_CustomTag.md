---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 커스텀 태그"
date:   2021-09-16 16:40:00 
lastmod : 2021-09-16 16:40:00
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

<br/><br/>

# 커스텀 태그 활용

## 개요

### 커스텀 태그란?

- JUnit 5 애너테이션을 조합하여 본인만의 태그를 만드는 것을 뜻한다.

<br/><br/>

## 커스텀 태그 만들기

### 생성 예시

```java
@Target(ElementType.METHOD) // 메서드 레벨에 적용한다.
@Retention(RetentionPolicy.RUNTIME) // 해당 애너테이션 정보를 Runtime까지 유지한다.
@Tag("MyTag")
@Test
public @interface MyTagTest {
}
```

- 해당 애너테이션은 테스트 패키지 내부에 작성해야한다.
- `@Target(애너테이션_적용_레벨)`
    - 어떤 레벨에 애너테이션을 적용할지 설정한다.
    - 매개변수, 메서드, 클래스 등을 설정할 수 있다.
- `@Retention(RetentionPolicy.RUNTIME)`
    - 런타임까지 해당 애너테이션 정보를 유지한다고 명시한다.

<br/>

### 활용 예시

```java
//import 생략

class StudyTest {
	
	@MyTagTest
  void create() {
		//테스트 코드
	}
 
}
```

- `@MyTagTest` 에 `@Tag("MyTag")` 와 `@Test` 가 포함되어있다.
- 즉, 위 코드는 아래와 같다.

    ```java
    //import 생략

    class StudyTest {
    	
    	@Tag("MyTag")
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