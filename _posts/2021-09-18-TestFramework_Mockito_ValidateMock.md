---
category: Test-Framework
tags: [Mockito]
title: "[Mockito] Mock 객체 행동 검증"
date:   2021-09-18 19:00:00 
lastmod : 2021-09-18 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

> 위 프로젝트를 참고하여 기본적인 웹 애플리케이션 코드를 꼭 확인하자!

- 이전 게시글
    - [[Mockito] Mockito 개요](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Summary)
    - [[Mockito] Mock 객체 만들기](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_CreateMock)
    - [[Mockito] Mock 객체 행동 정의](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Stubbing)

<br/><br/>

# Mock 객체의 행동 검증하기

## 개요

### Mock 객체 행동 검증이란?

- Mock 객체가 어떻게 사용이 됐는지 확인하는 것이다.
- 즉, 우리가 정의한 Mock 객체의 행동을 검증하여 어떻게 작동했는지 알 수 있다.

<br/>

### Mock 객체의 행동 검증을 통해 확인할 수 있는 내용

Mock 객체가 어떻게 행동을 했는지 확인할 수 있는 내용은 아래와 같다.

- 특정 메서드가 특정 매개변수로 **몇 번 호출되었는지 확인** 할 수 있다.
- **특정 시점 이후에 아무일도 벌어지지 않았는지 확인** 할 수 있다.

> 이외에도 특정시간에 따라 검증, 순서에 따라 검증할 수 있지만, 본 포스팅에선 다루지 않는다.  (사용하는 경우가 거의 없다고 한다.)

<br/><br/>

## 특정 메서드가 특정 매개변수로 몇번 호출되었는지 검증

- 특정 메서드가 특정 매개변수로 **몇 번 호출되었는지 확인** 할 수 있다.
- `verify()` 메서드를 통해 검증한다.

<br/>

### 형식

```java
verify(확인할_mock객체, times(호출될_횟수)).mock객체의_메서드(매개변수)
```

- **해당 mock 객체의 메서드가 해당 매개변수로 몇번 호출되었는지 확인한다.**
    - 물론 `any()` 를 `mock객체의_메서드` 의 매개변수 전달하여 매개변수에 상관없이 검증할 수 있다.
- `times(횟수)` 대신 다양한 값이 들어갈 수 있다.
    - 한번도 호출되지 않았는가? ⇒ `never()`
    - 정확히 2번 호출되었는가? ⇒ `times(2)`
    - 2번 이상으로 호출되었는가?(2 포함) ⇒ `atLeast(2)`
    - 2번 이하로 호출되었는가?(2 포함) ⇒ `atMost(2)`
- **`호출될_횟수` 가 정확히 맞아떨어져야 테스트에 성공한다.**
- `호출될_횟수` != `실제로_호출된_횟수` 라면, 테스트에 실패한다.

<br/>

### 예시 코드

```java
import static org.mockito.Mockito.*;
//나머지 import 생략

class StudyServiceTest {

  @DisplayName("행동 검증")
  @Test
  void testValidation(@Mock MemberService memberService,
                      @Mock StudyRepository studyRepository) {
    StudyService studyService = new StudyService(memberService, studyRepository);
    Study study = new Study(10, "java");
    Member member = new Member();
    member.setId(1L);
    member.setEmail("test@test.com");

    // ----------- 행동 정의 -----------
    when(memberService.findById(1L)).thenReturn(Optional.of(member));
    when(studyRepository.save(study)).thenReturn(study);

    // ----------- 행동 호출 -----------
    Study newStudy = studyService.createNewStudy(1L, study); //1번만 호출됨

    // ----------- ***행동 검증*** -----------
    verify(memberService, times(1)).findById(1L);

    // ----------- 테스트 검증 -----------
    assertEquals(study, newStudy);
  }

}
```

### 흐름

1. **mock 객체를 주입받은 객체 `StudyService` 의 메서드 호출**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%202.png)

2. **`StudyService` 의 `createNewStudy()` 메서드**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%204.png)

    - `memberService` , `repository` 는 mock객체이다.
    - 따라서, 정의된대로 행동한다.
3. **`memberService` 객체의 `findById()` 메서드가 한번만 호출되었다.**
4. **행동 검증: 1번만 수행되었는가?**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%205.png)

5. **테스트 성공!**

<br/>

### 결과

- 성공시

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%206.png)

- 실패시

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%207.png)

<br/>

### 주의사항

- **mock 객체를 검증하는 코드는 반드시 mock 객체의 행동을 호출한 뒤에 작성해야한다.**
- 예시) `studyService.createNewStudy(1L, study)` 호출 후, `verify()` 를 호출하였다.

<br/><br/>

## 특정 시점 이후에 아무 일이 없는지 검증

- 특정 메서드가 **특정 시점 이후에 호출되지 않았는지 검증**할 수 있다.
- `verify()` 메서드와 `verifyNoMoreInteractions()` 메서드를 통해 검증한다.

### 형식

```java
verify(확인할_mock객체).mock객체의_메서드(매개변수); //기준 시점 설정
verifyNoMoreInteractions(확인할_mock객체); //기준 시점 이후 검증
```

- 해당 mock 객체의 메서드가 기준 시점 이후에도 호출되었는지 확인한다.
- **`verify()` 메서드로 설정한 'mock객체의 메서드'가 호출된 시점을 기준으로, mock객체가 사용되었다면 테스트에 실패한다.**

<br/>

### 예시 코드

- **성공 예시**

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @DisplayName("행동 검증")
      @Test
      void testValidation(@Mock MemberService memberService,
                          @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");

        // ----------- 행동 정의 -----------
        when(memberService.findById(1L)).thenReturn(Optional.of(member));
        when(studyRepository.save(study)).thenReturn(study);

        // ----------- 행동 호출 -----------
        Study newStudy = studyService.createNewStudy(1L, study);

        // ----------- ***행동 검증*** -----------
        verify(memberService).notify(any()); //기준 시점 = notify()가 호출된 시점
        verifyNoMoreInteractions(memberService); //기준 시점 이후에, memberService 객체(mock)가 사용되지 않았나?

        // ----------- 테스트 검증 -----------
        assertEquals(study, newStudy);
      }

    }
    ```

- **실패 예시**

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @DisplayName("행동 검증")
      @Test
      void testValidation(@Mock MemberService memberService,
                          @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");

        // ----------- 행동 정의 -----------
        when(memberService.findById(1L)).thenReturn(Optional.of(member));
        when(studyRepository.save(study)).thenReturn(study);

        // ----------- 행동 호출 -----------
        Study newStudy = studyService.createNewStudy(1L, study);

        // ----------- ***행동 검증*** -----------
        verify(memberService).findById(any()); //기준 시점 = findById()가 호출된 시점
        verifyNoMoreInteractions(memberService); //기준 시점 이후에, memberService 객체(mock)가 사용되지 않았나?

        // ----------- 테스트 검증 -----------
        assertEquals(study, newStudy);
      }

    }
    ```

### 흐름: 성공 예시

1. **mock 객체를 주입받은 객체 `StudyService` 의 메서드 호출**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%208.png)

2. **`StudyService` 의 `createNewStudy()` 메서드**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%209.png)

    - `memberService` , `repository` 는 mock객체이다.
    - 따라서, 정의된대로 행동한다.
3. **`memberService` 객체(mock)의 `notify()` 메서드 이후로, `memberService` 객체에 접근한 적이 없는가?**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2010.png)

4. **기준시점 계산**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2011.png)

5. **해당 기준 시점 이후에, `memberService` 객체에 접근한 적이 없다.**
6. **테스트 성공!**

<br/>

### 흐름: 실패 예시

1. **mock 객체를 주입받은 객체 `StudyService` 의 메서드 호출**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2012.png)

2. **`StudyService` 의 `createNewStudy()` 메서드**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%209.png)

    - `memberService` , `repository` 는 mock객체이다.
    - 따라서, 정의된대로 행동한다.
3. **`memberService` 객체(mock)의 `findById()` 메서드 이후로, `memberService` 객체에 접근한 적이 없는가?**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2013.png)

4. **기준시점 계산**

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2014.png)

5. **해당 기준 시점 이후에, `memberService` 객체에 접근한 적이 있다.**
6. **테스트 실패!**

<br/>

### 결과

- 성공시

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%206.png)

- 실패시

    ![Untitled](/assets/img/2021-09-18-TestFramework_Mockito_ValidateMock/Untitled%2015.png)

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*