---
category: Test-Framework
tags: [Test-Framework, JUnit5]
title: "[JUnit 5] 테스트 반복"
date:   2021-09-16 22:00:00 
lastmod : 2021-09-16 22:00:00
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

<br/><br/>

# 테스트 반복하기

## 개요

- JUnit 5에서 특정 테스트 메서드를 반복하여 실행할 수 있다.
- 반복 횟수 등을 지정할 수 있다.

<br/><br/>

## `@RepeatedTest` 애너테이션

### 애너테이션 형식

`@RepeatedTest(name="반복테스트명", value=반복횟수)`

<br/>

### 속성

- `name`
    - **{displayName}**
        - `@DisplayName` 으로 설정한 이름을 출력한다.
    - **{currentRepetition}**
        - 현재 반복횟수를 출력한다.
    - **{totalRepetitions}**
        - 전체 반복횟수를 출력한다.
- `value`
    - 반복 횟수를 설정한다.

<br/>

### 기능

- 반복 횟수와 반복 테스트 이름을 설정한다.
- **`RepetitionInfo` 타입의 객체을 테스트 메서드의 매개변수로 받을 수 있다.**
    - `RepetitionInfo` 객체: 반복관련 정보가 담긴 객체

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@DisplayName("테스트 반복")
  @RepeatedTest(name = "{displayName}: 전체 {totalRepetitions} 중 {currentRepetition}번", value = 5)
  void repeatTest(RepetitionInfo info) {
    System.out.println("StudyTest.repeatTest" + info.getCurrentRepetition());
  }
 
}
```

<br/>

- 출력결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%206.png)

<br/><br/>

## `@ParameterizedTest` + `@ValueSource` 애너테이션

### 애너테이션 형식

- `@ParameterizedTest(name="반복테스트명")`
- `@ValueSource(데이터타입s={값1, 값2, ...})`

<br/>

### 속성

- `name`
    - **{displayName}**
        - `@DisplayName` 으로 설정한 이름을 출력한다.
    - **{index}**
        - 현재 반복횟수를 출력한다.
    - **{0}, {1}, ...**
        - 매개변수로 전달받은 것들 중 해당되는 값
        - 여러 종류의 매개변수를 전달받는다면 {n}으로 처리한다.

<br/>

### 기능

- `@ValueSource` 로 설정한 값의 개수만큼 반복한다.
- `@ValueSource` 로 설정한 값을 테스트 메서드의 매개변수로 전달받는다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@DisplayName("커스텀 타입 컨버터 + 매개변수 반복 2")
  @ParameterizedTest(name = "{displayName} 현재반복횟수={index}, 전달받은 매개변수값={0}")
  @ValueSource(strings = {"A", "B", "C"})
  void paramRepeat(String s) {
    System.out.println("s = " + s);
  }

}
```

<br/>

- 출력결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%207.png)

<br/><br/>

## 인자로 전달할 수 있는 소스 애너테이션

- 위 경우처럼, `@ValueSource` 애너테이션을 통해 매개변수에 값을 전달하는 방법 뿐만 아니라, 다른 애너테이션을 사용할 수 있다.
- 소스 애너테이션 종류
    - `@ValueSource`
        - 오직 한 종류의 인자만 전달할 수 있다.
    - `@NullSource`
        - 매개변수에 **null** 을 넣은 테스트를 수행한다.
        - 다른 소스 애너테이션과 조합하여 사용할 수 있다.
    - `@EmptySource`
        - 매개변수에 **빈 문자열** 을 넣은 테스트를 수행한다.
        - 다른 소스 애너테이션과 조합하여 사용할 수 있다.
    - `@NullAndEmptySource`
        - 매개변수에 **null과 빈 문자열** 을 넣은 테스트를 수행한다.
        - 다른 소스 애너테이션과 조합하여 사용할 수 있다.
    - `@CsvSource`
        - `@CsvSource` 는 배열 형태의 문자열을 배열 형태로 받아서 여러 종류의 매개변수를 처리할 수 있다.
        - `@CsvSource` + `ArgumentsAccessor 객체` 으로 사용하거나
        - `@CsvSource` + `@AggregateWith` 으로 사용할 수 있다.

<br/>

### `@ValueSource` 애너테이션

- 오직 한 종류의 인자만 전달할 수 있다.
- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@DisplayName("커스텀 타입 컨버터 + 매개변수 반복 2")
      @ParameterizedTest(name = "{displayName} 현재반복횟수={index}, 전달받은 매개변수값={0}")
      @ValueSource(strings = {"A", "B", "C"})
      void paramRepeat(String s) {
        System.out.println("s = " + s);
      }

    }
    ```

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%207.png)

<br/>

### `@NullSource` 애너테이션

- 매개변수에 **null** 을 넣은 테스트를 수행한다.
- 다른 소스 애너테이션과 조합하여 사용할 수 있다.
- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@Test
      @ParameterizedTest(name = "{index}번째")
      @ValueSource(strings = {"A", "B", "C"})
      @NullSource
      void test(String s) {
        if (s == null) {
          System.out.println("s = null");
        } else {
          System.out.println("s = " + s);
        }
      }

    }
    ```

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%208.png)

<br/>

### `@EmptySource` 애너테이션

- 매개변수에 **빈 문자열** 을 넣은 테스트를 수행한다.
- 다른 소스 애너테이션과 조합하여 사용할 수 있다.
- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@Test
      @ParameterizedTest(name = "{index}번째")
      @ValueSource(strings = {"A", "B", "C"})
      @EmptySource
      void test(String s) {
        System.out.println("s = " + s);
      }

    }
    ```

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%209.png)

<br/>

### `@NullAndEmptySource` 애너테이션

- 매개변수에 **null과 빈 문자열** 을 넣은 테스트를 수행한다.
- 다른 소스 애너테이션과 조합하여 사용할 수 있다.
- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@Test
      @ParameterizedTest(name = "{index}번째")
      @ValueSource(strings = {"A", "B", "C"})
      @NullAndEmptySource
      void test(String s) {
        if (s == null) {
          System.out.println("s = null");
        } else {
          System.out.println("s = " + s);
        }
      }

    }
    ```

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%2010.png)

<br/>

### `@NullAndEmptySource` 애너테이션

- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@Test
      @ParameterizedTest(name = "{index}번째")
      @ValueSource(strings = {"A", "B", "C"})
      @NullAndEmptySource
      void test(String s) {
        if (s == null) {
          System.out.println("s = null");
        } else {
          System.out.println("s = " + s);
        }
      }

    }
    ```

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%2010.png)

<br/>

### `@CsvSource` 애너테이션, `ArgumentsAccessor객체`

- `@CsvSource` 는 배열 형태의 문자열을 배열 형태로 받아서 여러 종류의 매개변수를 처리할 수 있다. 즉, 아래와 같다.
    - `@CsvSource({"1, '첫번째 반복'", "2, '두번째 반복'", "3, '세번째 반복'"})`
- **`ArgumentsAccessor` 객체를 매개변수로 전달받아서, `@CsvSource` 로 전달한 값에 접근할 수 있다.**

<br/>

- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@DisplayName("커스텀 타입 컨버터 + 매개변수 반복 4")
      @ParameterizedTest
      @CsvSource({"10, '자바'", "20, '공부'"})
      void paramRepeatWithCsv2(ArgumentsAccessor argumentsAccessor) {
        Study study = new Study(argumentsAccessor.getInteger(0), argumentsAccessor.getString(1));
        System.out.println("study = " + study);
      }

    }
    ```

    > `Study` 클래스는 [이전 게시글](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation#8)이나 예시 프로젝트 참고

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%2011.png)

<br/><br/>

## 인자 값 타입 변환

### 암묵적인 타입 변환

- JUnit 5가 기본적으로 제공하는 타입 컨버터가 제공하는 기능이다.
- 어떤 타입을 어떻게 변환해줄까? [(레퍼런스 참고)](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit)

<br/>

### 명시적인 타입 변환

- `SimpleArgumentConverter` 를 상속 받은 구현체를 만들어서 커스텀 타입 컨버트가 가능하다.
- `@ConvertWith` 애너테이션을 활용한다.

<br/>

- 예시) int → Study객체

    ```java
    //import 생략

    class StudyTest {
    	
    	@DisplayName("커스텀 타입 컨버터 + 매개변수 반복 1")
      @ParameterizedTest(name = "{displayName}: 현재반복횟수={index}, 전달받은 매개변수값={0}")
      @ValueSource(ints = {10, 20, 40})
      void paramRepeatWithCustomTypeConvert(@ConvertWith(StudyConverter.class) Study study) {
        System.out.println("study.getLimit() = " + study.getLimit());
      }

    	static class StudyConverter extends SimpleArgumentConverter {
        @Override
        protected Object convert(Object source, Class<?> targetType) throws ArgumentConversionException {
          Study study = new Study(Integer.parseInt(source.toString()));
          return study;
        }
      }

    }
    ```

    > `Study` 클래스는 [이전 게시글](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation#8)이나 예시 프로젝트 참고

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%2012.png)

<br/><br/>

## 인자 값 조합

### 기존 방식

- 기존에는 `ArgumentsAccessor객체` 를 매개변수로 전달받아서 인자값에 접근하였다.

<br/>

### 인자 값 조합 분리하기

- `@AggregateWith` 애너테이션과 `ArgumentsAggregator` 인터페이스를 사용하면 따로 조합할 수 있다.
- **즉, `@AggregateWith` 를 사용하면 직접 ArgumentsAccessor 객체를 사용하지 않고, 원하는 타입으로 변환할 수 있다.**
    - 어떤 객체로 변환할 것인지는 static class에 따로 정의해야한다.
    - 해당 static class는 `ArgumentsAggregator` 인터페이스를 implements 한다.

<br/>

- 사용 예시

    ```java
    //import 생략

    class StudyTest {
    	
    	@DisplayName("커스텀 타입 컨버터 + 매개변수 반복 5")
      @ParameterizedTest
      @CsvSource({"10, '자바'", "20, '공부'"})
      void paramRepeatWithCsv3(@AggregateWith(StudyAggregator.class) Study study) {
        System.out.println("study = " + study);
      }

    	static class StudyAggregator implements ArgumentsAggregator {
        @Override
        public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context) throws ArgumentsAggregationException {
          Study study = new Study(accessor.getInteger(0), accessor.getString(1));
          return study;
        }
    	}

    }
    ```

    > `Study` 클래스는 [이전 게시글](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation#8)이나 예시 프로젝트 참고

- 결과

    ![Untitled](/assets/img/2021-09-16-TestFramework_JUnit5_RepeatTest/Untitled%2013.png)

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*