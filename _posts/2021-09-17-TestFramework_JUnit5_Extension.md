---
category: Test-Framework
tags: [JUnit5, Extension]
title: "[JUnit 5] 확장 모델"
date:   2021-09-17 16:30:00 
lastmod : 2021-09-17 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

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
    - [[JUnit 5] JUnit 설정 파일](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_Configuration)

<br/><br/>

# 확장 모델 사용하기

## 개요

### 확장 모델이란?

- 테스트 클래스에 대한 부가 기능을 따로 분리해낸 것
- 즉, 테스트 클래스의 기능을 확장한 모델이다.
- 예시)
    - 각 테스트 메서드 중 1초 이상 걸리는 테스트 찾기

<br/>

### 확장 모델 등록: JUnit 4

- JUnit 4에서 확장 모델을 적용하기 위해 아래 애너테이션들을 사용한다.
    - `@RunWith(Runnner)`
    - `@Rule`
    - etc...

<br/>

### 확장 모델 등록: JUnit 5

- JUnit 5에서 확장 모델을 적용하기 위해선, `@ExtendWith` 이나 `@RegisterExtension`만을 이용한다.
- JUnit 5에서의 확장 모델은 오직 `Extension` 만 존재한다.

<br/>

### 확장모델을 만드는 방법

1. 원하는 기능의 인터페이스를 implements 한다.
2. implements 한 인터페이스의 메서드를 구현한다.
> 본 게시글에선 BeforeTestExecutionCallback 과 AfterTestExecutionCallback 인터페이스에 대해서만 다룬다.

<br/>

### 확장모델 등록(적용) 방법

확장모델을 등록하는 방법에는 크게 3가지가 존재한다.

- 선언적 등록
    - `@ExtendWith(확장모델.class)` 애너테이션을 활용한다.
    - **해당 애너테이션을 테스트 클래스 레벨에 적용하여, 확장모델을 적용한다.**
- 프로그래밍 등록
    - `@RegisterExtension` 애너테이션을 활용한다.
    - **해당 애너테이션을 테스트 클래스 내부의 static 필드에 적용하여, 확장모델 객체를 생성 및 등록한다.**
- 자동 등록
    - **[이전 게시글](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_Configuration#4)** 에서 다룬 설정파일을 이용한다.

<br/><br/>

## 주요 확장 모델 인터페이스

### BeforeTestExecutionCallback 인터페이스

해당 인터페이스를 구현한 확장 모델을 테스트 클래스에 등록하면, 각 테스트 메서드가 실행되기 전에 아래 메서드가 실행된다.

- 메서드

    ```java
    public void beforeTestExecution(ExtensionContext context) throws Exception {
    	
    }
    ```

- `ExtendsionContext` 매개변수
    - 아래에서 설명한다.

<br/>

### AfterTestExecutionCallback 인터페이스

해당 인터페이스를 구현한 확장 모델을 테스트 클래스에 등록하면, 각 테스트 메서드가 실행되고 난 후에 아래 메서드가 실행된다.

- 메서드

    ```java
    public void afterTestExecution(ExtensionContext context) throws Exception {
    	
    }
    ```

- `ExtendsionContext` 매개변수
    - 이 역시, 아래에서 설명한다.

<br/>

### `ExtendsionContext` 객체

- 해당 객체는 `afterTestExecution` 메서드, `beforeTestExecution` 메서드 등에서 사용되는 매개변수이다. JUnit 프레임워크가 해당 매개변수에 적절한 객체를 전달해준다.
- 해당 객체는 테스트 메서드에 대한 문맥(context)(정보)를 가지고 있다.

<br/>

- **적용된 테스트 클래스 이름 가져오기**
    - `getRequiredTestClass().getName()`
- **현재 테스트할 메서드의 이름 가져오기**
    - `getRequiredTestMethod().getName()`
- **확장 모델 내부에서 공통으로 사용할 수 있는 저장소(Store) 객체 가져오기**
    - `getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));`
- **확장 모델 내부에서 공통으로 사용할 수 있는 저장소(Store) 객체에 값 저장하기**
    - `저장소_객체.put("이름", 값);`
- **확장 모델 내부에서 공유으로 사용할 수 있는 저장소(Store) 객체에 저장된 값을 삭제하고 가져오기**
    - `저장소_객체.remove("START_TIME", long.class);`

<br/><br/>

## 확장 모델 사용 예시

### 확장 모델 만들기

- 해당 확장 모델은 수행시간이 1초 이상인 테스트 메서드를 알려주는 확장모델이다.

```java
package practice.testCode;

import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.BeforeTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.springframework.test.context.event.annotation.AfterTestExecution;
import org.springframework.test.context.event.annotation.BeforeTestExecution;

//수행시간이 1초 이상인 테스트 메서드를 알려주는 확장모델
public class FindSlowExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

	private static final long THRESHOLD = 1000L;

  //테스트 메서드 호출 전, 호출되는 메서드
  @Override
  public void beforeTestExecution(ExtensionContext context) throws Exception {
    String testClassName = context.getRequiredTestClass().getName();
    String testMethodName = context.getRequiredTestMethod().getName();

    //확장 모델 내부에서 공유할 수 있는 저장소(Store) 가져오기
    ExtensionContext.Store store = context.getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));

    //확장 모델 내부에서 공유할 수 있는 저장소(Store)에 값 저장하기
    store.put("START_TIME", System.currentTimeMillis());
  }

  //테스트 메서드 호출 후, 호출되는 메서드
  @Override
  public void afterTestExecution(ExtensionContext context) throws Exception {
    String testClassName = context.getRequiredTestClass().getName();
    String testMethodName = context.getRequiredTestMethod().getName();

    //확장 모델 내부에서 공유할 수 있는 저장소(Store) 가져오기
    ExtensionContext.Store store = context.getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));

    //확장 모델 내부에서 공유할 수 있는 저장소(Store)에 저장된 값 삭제 및 가져오기
    Long start_time = store.remove("START_TIME", long.class);
    long duration = System.currentTimeMillis() - start_time;

    if (duration > THRESHOLD) {
      System.out.printf("[%s] 메서드는 느린 테스트입니다.\n", testMethodName);
    }
  }

}
```

<br/>

### 확장모델 등록하기: `@ExtendWith` 사용

```java
// import 생략

@ExtendWith(FindSlowExtension.class)
class StudyTest {

  @DisplayName("실행시간이 1초 이하인 테스트")
  @Test
  void fastTest() {
    System.out.println("StudyTest.fastTest 실행됨.");
  }

  @DisplayName("실행시간이 1초 이상인 테스트")
  @Test
  void slowTest() {
    System.out.println("StudyTest.slowTest 실행됨.");
    try {
      Thread.sleep(2000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

}
```

<br/>

### 결과: `@ExtendWith` 사용

![Untitled](/assets/img/2021-09-17-TestFramework_JUnit5_Extension/Untitled%2015.png)

<br/>

### 확장모델 등록하기: `@RegisterExtension` 사용

```java
// import 생략

//@ExtendWith(FindSlowExtension.class)
class StudyTest {

	@RegisterExtension
	static FindSlowExtension extension = new FindSlowExtension();

  @DisplayName("실행시간이 1초 이하인 테스트")
  @Test
  void fastTest() {
    System.out.println("StudyTest.fastTest 실행됨.");
  }

  @DisplayName("실행시간이 1초 이상인 테스트")
  @Test
  void slowTest() {
    System.out.println("StudyTest.slowTest 실행됨.");
    try {
      Thread.sleep(2000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

}
```

<br/>

### 결과: `@ExtendWith` 사용
- 위 결과와 동일하다.  

![Untitled](/assets/img/2021-09-17-TestFramework_JUnit5_Extension/Untitled%2015.png)

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*