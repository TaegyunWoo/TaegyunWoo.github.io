---
category: Test-Code
tags: [JUnit5]
title: "[JUnit 5] 기본 테스트"
date:   2021-09-15 16:30:00 
lastmod : 2021-09-15 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**
- 이전 게시글
    - [[JUnit 5] 개요 및 기본 애너테이션](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation)

<br/><br/>

# 기본 Assertions

## 개요

### 주요 메서드 종류

- `assertEquals(expected, actual)`
- `assertNotNull(actual)`
- `assertTrue(boolean)`
- `assertAll(executables...)`
- `assertThrows(expectedType, executable)`
- `assertTimeout(duration, executable)`

> `executable` 은 람다식으로 생각할 수 있다.

<br/>

### 테스트 실패시 출력할 Message

- **각 메서드의 마지막 매개변수에 테스트 실패시 출력할 message를 전달할 수 있다.**
- 복잡한 메시지를 생성해야 하는 경우, **`Supplier<String>`** 을 사용하면 효율적이다.
    - **문자열 연산자 `+` 를 사용하여 msg 만들기**
        - 테스트 실패 여부와 상관없이 무조건 msg에 대해 연산을 수행한다.
    - **`Supplier<String>` 을 사용하여 msg 만들기**
        - 테스트 실패시에만 연산된다. (람다식을 이용하므로)
        - 따라서, 실패 msg가 필요할 때만 msg에 대해 연산을 수행한다.
- `Supplier<String>` 사용 예시

    ```java
    assertEquals("A", 객체.getName(), Supplier<String>() {
    	@Override
    	public String get() {
    		//복잡한 문자열 계산 로직
    		return "복잡한 문자열";
    	}
    })
    ```

<br/>

### 예시 코드 도메인

- 이제부터 설명할 예제 코드에 대한 전제조건은 이전 게시글을 참고하면 된다.
    - [전제조건](https://taegyunwoo.github.io/test-framework/TestFramework_JUnit5_SummaryAndBasicAnnotation#7)

<br/><br/>

## `assertEquals` 메서드

### 메서드 형식

`assertEquals(기대하는값, 실제값)`

### 기능

- '기대하는값' 과 '실제값' 이 동일한 값을 갖는지 확인한다.
- 서로 다른 값이라면, 테스트에 실패한다.
- **완벽히 동일한 객체, 즉 주소값이 같은 객체인지 확인하려면 `assertSame()` 을 사용해야 한다.**

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);
		assertEquals(StudyStatus.DRAFT, study.getStudyStatus(), new Supplier<String>() {
				@Override
				public String get() {
					return "study 객체 생성시, 상태는 DRAFT이어야 한다.";
				}
    );
	}
 
}
```

<br/><br/>

## `assertNotNull` 메서드

### 메서드 형식

`assertNotNull(실제값)`

<br/>

### 기능

- '실제값' 이 null 이 아닌지 확인한다.
- null이라면, 테스트에 실패한다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);
		assertNotNull(study);
	}
 
}
```

<br/><br/>

## `assertThrows` 메서드

### 메서드 형식

`assertThrows(예외클래스.class, () -> {예외가 발생하는 코드})`

<br/>

### 기능

- 람다식이 해당 예외클래스를 던지면 테스트가 성공한다.
- 람다식이 해당 예외클래스를 던지지 않으면, 테스트가 실패한다.

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
			assertThrows(RuntimeException.class, () -> {
				//RuntimeException이 발생하는 코드
			});
	}
 
}
```

<br/><br/>

## `assertAll` 메서드

### 메서드 형식

```java
assertAll( () -> assert문_1,
			() -> assert문_2,
			() -> assert문_3, ...)
```

<br/>

### 기능

- 매개변수로 전달한 assert문들을 모두 수행한다.
- assertAll을 사용하지 않고 여러 assert 문 수행시
    - **어떤 assert 문이 실패하면 나머지 assert 문이 수행되지 않는다.**
- assertAll을 사용하여 여러 assert 문 수행시
    - **어떤 assert 문이 실패해도 나머지 assert 문이 수행된다.**

<br/>

### 예시 코드

```java
//import 생략

class StudyTest {
	
	@Test
  void create() {
    Study study = new Study(10);

		assertAll(
            () -> assertNotNull(study),
            () -> assertEquals(StudyStatus.DRAFT, study.getStudyStatus(), new Supplier<String>() {
                @Override
                public String get() {
                    return "study 객체 생성시, 상태는 DRAFT이어야 한다.";
                }
            }),
            () -> assertTrue(study.getLimit() > 0, "study 객체의 limit 값은 0보다 커야한다.")
		);
	}
 
}
```

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*