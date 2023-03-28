---
category: Test-Framework
tags: [Test-Framework, Mockito]
title: "[Mockito] BDD 스타일 API 작성"
date:   2021-09-18 20:30:00 
lastmod : 2021-09-18 20:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

> 위 프로젝트를 참고하여 기본적인 웹 애플리케이션 코드를 꼭 확인하자!

- 이전 게시글
    - [[Mockito] Mockito 개요](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Summary)
    - [[Mockito] Mock 객체 만들기](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_CreateMock)
    - [[Mockito] Mock 객체 행동 정의](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Stubbing)
    - [[Mockito] Mock 객체 행동 검증](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_ValidateMock)

<br/><br/>

# BDD 스타일로 Mockito 사용하기

## 개요

### BDD란?

- BDD: Behavior Driven Development
- 애플리케이션이 어떻게 **행동**해야 하는지에 대한 공통된 이해를 구성하는 방법이다.
- TDD(Test Driven Development)에서 창안되었다.

<br/>

### Given-When-Then 패턴

- **Given**
    - 어떤 테스트에 대한 **준비단계**
- **When**
    - 어떤 테스트에 대한 **실행단계**
- **Then**
    - 어떤 테스트에 대한 **검증단계**

<br/>

### 주요 변경 사항

Mockito는 BddMockito라는 클래스를 통해 BDD 스타일의 API를 제공한다.

- **when ⇒ given**
- **verify ⇒ then**

<br/><br/>

## when → given

- 기존에 사용하던 `when()` 메서드와 `thenReturn()` , `thenThrow()` 메서드를 아래와 같이 변경하여 사용한다.
- `when()` ⇒ `given()`
- `thenReturn()` ⇒ `willReturn()`
- `thenThrow()` ⇒ `willThrow()`

<br/>

### 요약

```java
when(memberService.findById(1L)).thenReturn(Optional.of(member)); //기존 코드
given(memberService.findById(1L)).willReturn(Optional.of(member)); //BDD 스타일 코드
```

<br/>

### 예시 코드

```java
import static org.mockito.BDDMockito.given;
//나머지 import 생략

class StudyServiceTest {

  @DisplayName("BDD 스타일 API")
  @Test
  void testBdd(@Mock MemberService memberService,
               @Mock StudyRepository studyRepository) {
    //------------- given -------------
    StudyService studyService = new StudyService(memberService, studyRepository);
    Study study = new Study(10, "java");
    Member member = new Member();
    member.setId(1L);
    member.setEmail("test@test.com");

    //행동 정의(given)
    given(memberService.findById(1L)).willReturn(Optional.of(member));
    given(studyRepository.save(study)).willReturn(study);

    //------------- when -------------
    studyService.createNewStudy(1L, study);

    //------------- then -------------
    assertEquals(study, newStudy);

    //행동 검증(차후 설명)
    verify(memberService, times(1)).findById(1L);
  }

}
```

<br/><br/>

## verify → then

- 기존에 사용하던 `verify()` 메서드를 아래와 같이 변경하여 사용한다.
- `verify()` ⇒ `then()`
- `verifyNoMoreInteractions()` ⇒ `shouldHaveNoMoreInteractions()`
- `should()`

<br/>

### 요약

```java
verify(memberService, times(1)).findById(1L); //기존 코드
then(memberService).should(times(1)).findById(1L); //BDD 스타일 코드

verifyNoMoreInteractions(memberService); //기존 코드
then(memberService).shouldHaveNoMoreInteractions(); //BDD 스타일 코드
```

<br/>

### 예시 코드

```java
import static org.mockito.BDDMockito.given;
import static org.mockito.BDDMockito.then;
//나머지 import 생략

class StudyServiceTest {

  @DisplayName("BDD 스타일 API")
  @Test
  void testBdd(@Mock MemberService memberService,
               @Mock StudyRepository studyRepository) {
    //------------- given -------------
    StudyService studyService = new StudyService(memberService, studyRepository);
    Study study = new Study(10, "java");
    Member member = new Member();
    member.setId(1L);
    member.setEmail("test@test.com");

    //행동 정의(given)
    given(memberService.findById(1L)).willReturn(Optional.of(member));
    given(studyRepository.save(study)).willReturn(study);

    //------------- when -------------
    studyService.createNewStudy(1L, study);

    //------------- then -------------
    assertEquals(study, newStudy);

    //행동 검증
    then(memberService).should(times(1)).findById(1L);
  }

}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*