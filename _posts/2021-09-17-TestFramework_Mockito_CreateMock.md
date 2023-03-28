---
category: Test-Framework
tags: [Test-Framework, Mockito]
title: "[Mockito] Mock 객체 만들기"
date:   2021-09-17 17:30:00 
lastmod : 2021-09-17 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

- 이전 게시글
    - [[Mockito] Mockito 개요](https://taegyunwoo.github.io/test-framework/TestFramework_Mockito_Summary)

> 위 프로젝트를 참고하여 기본적인 웹 애플리케이션 코드를 꼭 확인하자!  
(`StudyService` , `MemberService`, `StudyRepository`)

<br/><br/>

# Mock 객체 생성하기

## 개요

### Mock 객체 생성 방법의 종류

Mock 객체를 만드는 방법에는 총 2가지의 방법이 존재한다.

- `Mockito.mock()` 메서드로 만드는 방법
- `@Mock` 애너테이션으로 만드는 방법

<br/>

### Mock 객체가 필요한 경우

- 어떤 인터페이스의 구현체가 필요하지만, 아직 구현체를 작성하지 않은 경우
- 예시 프로젝트에서의 필요성
    - `MemberService` 인터페이스만 존재하는 상태 (구현체 존재X)
    - `StudyRepository` 인터페이스만 존재하는 상태 (구현체 존재X)
    - `StudyService` 클래스의 생성자 형태

        ```java
        public class StudyService {
        	private MemberService memberService;
        	private StudyRepository studyRepository;

        	//기본 생성자 생략

        	public StudyService(MemberService memberService, StudyRepository studyRepository) {
        		this.MemberService memberService = MemberService memberService;
        		this.StudyRepository studyRepository = StudyRepository studyRepository;
        	}

        }
        ```

    - **이때, `StudyService` 클래스를 테스트 하고 싶다면?**
        1. `StudyService` 클래스의 생성자를 통해 객체를 생성해야한다.
        2. 하지만, `MemberService` 와 `StudyRepository` 인터페이스만 존재하고 구현체가 존재하지 않는다.
        3. 이때 Mock 객체를 사용하면 된다! (즉, Mock 객체로 의존성을 주입한다.)

<br/><br/>

## Mock 객체 생성: `Mockito.mock()`

### 형식

```java
Mock객체로_만들_인터페이스 mock = mock(Mock객체로_만들_인터페이스.class);
```

<br/>

### 사용 예시

```java
import static org.mockito.Mockito.*;
//나머지 import 생략

class StudyServiceTest {

  /**
   * Mockito.mock() 으로 mock 객체 만들기
   */
  @Test
  void createStudyServiceWithMethod() {
    /*
     StudyService 생성자의 인수로
     'MemberService'객체와, 'StudyRepository' 객체를 넣어야하지만
     현재 인터페이스만 존재하는 상황이다.
     따라서, mock 객체를 통해 StudyService 객체를 생성할 수 있다.
     */
    MemberService memberService = mock(MemberService.class);
    StudyRepository studyRepository = mock(StudyRepository.class);

		//Mock 객체로 의존성이 주입되었다.
    StudyService studyService = new StudyService(memberService, studyRepository);
    assertNotNull(studyService);
  }
}
```

<br/><br/>

## Mock 객체 생성: `@Mock`

### 형식: 필드

```java
@ExtendWith(MockitoExtension.class)
public class 테스트클래스 {
	//필드
	@Mock Mock객체로_만들_인터페이스 mock;

}
```

- `@ExtendWith(MockitoExtension.class)` 로 Mockito 확장모델을 적용해야 `@Mock` 을 사용할 수 있다.

<br/>

### 형식: 메서드 매개변수

```java
@ExtendWith(MockitoExtension.class)
public class 테스트클래스 {

	//메서드 매개변수
	@Test
	void 테스트메서드(@Mock Mock객체로_만들_인터페이스 mock) {
		//테스트코드
	}

}
```

- 이것도 마찬가지로, `@ExtendWith(MockitoExtension.class)` 로 Mockito 확장모델을 적용해야 `@Mock` 을 사용할 수 있다.

<br/>

### 사용 예시

```java
import static org.mockito.Mockito.*;
//나머지 import 생략

class StudyServiceTest {

  /**
   * @Mock 애너테이션으로 mock 객체 만들기
   */
  @Test
  void createStudyServiceWithAnnotaion(@Mock MemberService memberServiceWithAnno,
					@Mock StudyRepository studyRepositoryWithAnno
									) {

    /*
     StudyService 생성자의 인수로
     'MemberService'객체와, 'StudyRepository' 객체를 넣어야하지만
     현재 인터페이스만 존재하는 상황이다.
     따라서, mock 객체를 통해 StudyService 객체를 생성할 수 있다.
     */
	//Mock 객체로 의존성이 주입되었다.
    StudyService studyService = new StudyService(memberServiceWithAnno, studyRepositoryWithAnno);
    assertNotNull(studyService);
	}

}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*