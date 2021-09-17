---
category: Test-Framework
tags: [JUnit5]
title: "[JUnit 5] JUnit 설정 파일"
date:   2021-09-17 15:30:00 
lastmod : 2021-09-17 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

Title: JUnit 설정 파일

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
    - [[JUnit 5] 테스트 순서](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_TestSequence)

<br/><br/>

# junit-platform.properties

## 개요

- junit-platform.properties 파일은 JUnit 설정 파일이다.
- JUnit 관련 설정은 해당 파일에 작성하면 된다.
- **파일경로**
    - src/test/resources/junit-platform.properties

<br/>

## 주요 설정

### 테스트 인스턴스 라이프사이클 설정

`junit.jupiter.testinstance.lifecycle.default = per_class`

<br/>

### 확장팩 자동 감지 기능

`junit.jupiter.extensions.autodetection.enabled = true`

> 확장팩에 대한 자세한 내용은 다음 게시글에서 다룬다.

<br/>

### @Disabled 무시하고 실행하기

`junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition`

<br/>

### 테스트 이름 표기 전략 설정

`junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores`

- 위와 같이 설정시, 테스트메서드의 이름 중 언더스코어(_)를 공백문자로 변경해준다.

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*