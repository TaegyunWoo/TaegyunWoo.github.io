---
category: Test-Framework
tags: [Test-Framework, Mockito]
title: "[Mockito] Mock 객체 행동 정의"
date:   2021-09-17 19:00:00 
lastmod : 2021-09-17 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

> 위 프로젝트를 참고하여 기본적인 웹 애플리케이션 코드를 꼭 확인하자!

- 이전 게시글
    - [[Mockito] Mockito 개요](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Summary)
    - [[Mockito] Mock 객체 만들기](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_CreateMock)

<br/><br/>

# Mock 객체 행동(stubbing) 제어하기

## Mock 객체의 기본 행동 전략

어떠한 행동도 정의되지 않은 Mock 객체의 기본 행동에 대해서 설명하겠다.

<br/>

### 리턴값이 있는 메서드를 호출할 경우

- `mock.리턴값_있는_메서드()` 을 호출할 경우
- **`null` 을 반환한다.**
- **리턴값이 Optional 인 경우, `Optional.empty` 를 반환한다.**

<br/>

### 리턴값이 없는 메서드를 호출할 경우

- `mock.리턴값_없는_메서드()` 을 호출할 경우
- **예외를 던지지도 않고, 아무런 일도 발생하지 않는다.**

### Mock 객체의 프로퍼티 값에 접근할 경우

- `mock.프로퍼티` 에 접근할 경우
- **mock 객체의 프로퍼티 중 기본 타입은 기본 Primitive 값을 갖는다.**
    - String 형 → 빈문자열
    - int 형 → 0
    - etc ...
- **mock 객체의 프로퍼티 중 컬렉션 타입은 빈 컬렉션을 갖는다.**

<br/><br/>

## 리턴값이 있는 메서드 조작

### 특정 매개변수에 따른 행동 조작: 특정 값 리턴하기

- **특정한 매개변수를 받은 경우 특정한 값을 리턴하도록 mock 객체의 메서드를 조작할 수 있다.**
- 형식

    ```java
    when(mock객체.메서드(원하는_매개변수_값)).thenReturn(원하는_리턴값);
    ```

- 예시 코드

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @Test
      void test(@Mock MemberService memberService,
                @Mock StudyRepository studyRepository
                ) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");

        //행동 정의
        when(memberService.findById(1L)).thenReturn(Optional.of(member)); //findByID 메서드의 인수가 1L일 때만 동작
        
        studyService.createNewStudy(1L, study);
    	}

    }
    ```

    - **`memberService.findById()` 호출시, 매개변수가 `1L` 어어야만 `member` 객체를 반환한다.**
- **흐름**
    1. `studyService.createNewStudy(1L, study)` 를 테스트 메서드에서 호출한다면
    2. 아래 그림의 빨간색 표시된 부분에서 (`StudyService` 클래스의 `createNewStudy` 메서드)

        ![Untitled](/assets/img/2021-09-17-TestFramework_Mockito_Stubbing/Untitled.png)

    3. 테스트 메서드에서 작성한 `member` 객체가 반환된다.

<br/>

### 특정 매개변수에 국한되지 않는 행동 조작: 특정 값 리턴하기

- **모든 매개변수에 대해 특정한 값을 리턴하도록 mock 객체의 메서드를 조작할 수 있다.**
- 형식

    ```java
    when(mock객체.메서드(any())).thenReturn(원하는_리턴값);
    ```

- 예시 코드

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @Test
      void test(@Mock MemberService memberService,
                @Mock StudyRepository studyRepository
                ) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");

        //행동 정의
        when(memberService.findById(any())).thenReturn(Optional.of(member)); //findByID 메서드의 인수에 관계없이 동작
        
        studyService.createNewStudy(1L, study);
        }

    }
    ```

    - **`memberService.findById()` 호출시, 매개변수에 관계없이 `member` 객체를 반환한다.**
- **흐름**
    1. `studyService.createNewStudy(1L, study)` 를 테스트 메서드에서 호출한다면
    2. 아래 그림의 빨간색 표시된 부분에서 (`StudyService` 클래스의 `createNewStudy` 메서드)

        ![Untitled](/assets/img/2021-09-17-TestFramework_Mockito_Stubbing/Untitled.png)

    3. 테스트 메서드에서 작성한 `member` 객체가 반환된다.
        - **`any()` 를 이용하여, 행동을 정의하면 모든 매개변수에 대해 작동한다.**

<br/>

### 특정 매개변수에 따른 행동 조작: 특정 예외 던지기

- **특정한 매개변수를 받은 경우 특정 예외를 던지도록 mock 객체의 메서드를 조작할 수 있다.**
- 형식

    ```java
    when(mock객체.메서드(원하는_매개변수_값)).thenThrow(new 원하는_예외());
    ```

- 예시 코드

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @Test
      void test(@Mock MemberService memberService,
                @Mock StudyRepository studyRepository
                ) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");

        //행동 정의
        when(memberService.findById(1L)).thenThrow(new RuntimeException()); //findByID 메서드의 인수가 1L일 때만 RuntimeException 던지기
        
        studyService.createNewStudy(1L, study); //이 코드는 예외가 발생한다.
    	}

    }
    ```

    - **`memberService.findById()` 호출시, 매개변수가 `1L` 어어야만 `RuntimeException`를 반환한다.**
- **흐름**
    1. `studyService.createNewStudy(1L, study)` 를 테스트 메서드에서 호출한다면
    2. 아래 그림의 빨간색 표시된 부분에서 (`StudyService` 클래스의 `createNewStudy` 메서드)

        ![Untitled](/assets/img/2021-09-17-TestFramework_Mockito_Stubbing/Untitled.png)

    3. 테스트 메서드에서 작성한 `RuntimeException` 이 발생한다.

<br/><br/>

## 리턴값이 없는 메서드 조작

### 특정 예외 던지기

- **리턴값이 없는 메서드가 호출되었을 때, 특정 예외를 던지도록 조작할 수 있다.**
- 형식

    ```java
    doThrow(new 원하는_예외()).when(mock객체).mock객체의_메서드(원하는_매개변수);
    ```

    > 물론 `any()` 를 통해, 매개변수 상관없이 작동시킬 수도 있다.

- 예시 코드

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @Test
      void test(@Mock MemberService memberService,
                @Mock StudyRepository studyRepository
                ) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member = new Member();
        member.setId(1L);
        member.setEmail("test@test.com");
        when(memberService.findById(1L)).thenReturn(Optional.of(member)); //createNewStudy() 를 수행하기 위해 필요

        //행동 정의
        doThrow(new RuntimeException()).when(memberService).notify(any()); //notify 메서드의 인수에 상관없이 동작
        
        studyService.createNewStudy(1L, study); //이 코드는 예외가 발생한다.
    	}

    }
    ```

    - **`notify()` 메서드는 void형 메서드이다.**
    - **`memberService.notify()` 호출시, 매개변수에 상관없이 `RuntimeException`를 반환한다.**
- **흐름**
    1. `studyService.createNewStudy(1L, study)` 를 테스트 메서드에서 호출한다면
    2. 아래 그림의 빨간색 표시된 부분에서 (`StudyService` 클래스의 `createNewStudy` 메서드)

        ![Untitled](/assets/img/2021-09-17-TestFramework_Mockito_Stubbing/Untitled%201.png)

    3. 테스트 메서드에서 작성한 `RuntimeException` 이 발생한다.

<br/><br/>

## 여러번 호출시 각각 다른 행동으로 조작

### 여러번 호출시 다르게 행동하도록 조작

- **동일한 매개변수로 여러번 호출될 때, 각기 다르게 행동하도록 조작할 수 있다.**
- 형식

    ```java
    when(mock객체.메서드(원하는_매개변수_값)).thenReturn(원하는_리턴값_1) //첫번째 호출시 동작
    					.thenThrow(new 원하는_예외()) //두번째 호출시 동작
    					.thenReturn(원하는_리턴값_2) //세번째 호출시 동작
    					...
    ```

- 예시 코드

    ```java
    import static org.mockito.Mockito.*;
    //나머지 import 생략

    class StudyServiceTest {

      @Test
      void test(@Mock MemberService memberService,
                @Mock StudyRepository studyRepository
                ) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        Study study = new Study(10, "java");
        Member member1 = new Member();
        member.setId(1L);
        member.setEmail("test1@test.com");
        Member member2 = new Member();
        member.setId(2L);
        member.setEmail("test2@test.com");

        //행동 정의
        when(memberService.findById(1L)).thenReturn(Optional.of(member1)) //첫번째 호출시 동작
                    .thenReturn(Optional.of(member2)) //두번째 호출시 동작
                    .thenThrow(new RuntimeException()); //세번째 호출시 동작

        studyService.createNewStudy(1L, study); //1번째 동작
        studyService.createNewStudy(1L, study); //2번째 동작
        studyService.createNewStudy(1L, study); //3번째 동작 => 예외발생
    	}

    }
    ```

    - 첫번째 동작: `thenReturn(Optional.of(member1))`
    - 두번째 동작: `thenReturn(Optional.of(member2))`
    - 세번째 동작: `thenThrow(new RuntimeException())`

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*