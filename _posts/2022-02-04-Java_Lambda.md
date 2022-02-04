---
category: Java
tags: [Java]
title: "Java8 람다식 총정리"
date:   2022-02-04 23:15:00 
lastmod : 2022-02-04 23:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 람다식
## 개요
### 람다식이란?
- 람다식이란 간단히 말해서, **메서드를 하나의 식(expression)으로 표현한 것**이다.
- 람다식을 **익명 함수**라고 하기도 한다.
  > 익명 클래스와 익명 함수는 별개의 것이다.

<br/>

### 람다식 예시
```java
int[] arr = new int[5];
Arrays.setAll(arr, (i) -> (int)(Math.random()*5)+1);
```

<br/>

### 람다식의 의의
- **람다식으로 인해 메서드를 변수처럼 다루는 것이 가능해졌다.**
> 자바스크립트에서 함수를 하나의 값으로 취급하는 것(콜백함수)과 유사하다.

<br/>

### 메서드 vs 함수
- 함수
  - 수학에서 따온 명칭이다.
  - 수학의 함수와 개념이 유사하다.
- 메서드
  - 객체지향개념에서 함수 대신 사용하는 용어이다.
  - 객체지향개념에서는 어떤 함수는 특정 클래스에 반드시 속해야 한다는 제약이 있기 때문이다.
- **하지만 람다식의 등장으로, 메서드가 하나의 독립적인 기능을 하기 때문에 람다식에 국한하여 함수라는 용어를 사용한다.**

<br/><br/>

## 람다식의 기초
### 람다식 작성하기
- 기존 메서드 형태
  ```java
  반환타입 메서드이름(매개변수 선언) {
    문장들
    }
  ```
- 람다식으로 변환된 형태
  ```java
  (매개변수 선언) -> {
    문장들
  }
  ```

<br/>

### return 생략
- 반환값이 있는 메서드의 경우, `return`문 대신 `식`을 사용할 수 있다.
- **이때는 문장이 아닌 식이므로 끝에 `;`를 붙이지 않는다.**
- 예시
  - 기존 메서드 형태
    ```java
    int max(int a, int b) {
      return a > b ? a : b;
    }
    ```
  - 람다식 형태
    ```java
    (int a, int b) -> {
      return a > b ? a : b;
    }
    ```
  - `return` 생략
    ```java
    (int a, int b) -> a > b ? a : b
    ```

<br/>

### 매개변수의 타입 생략
- 매개변수의 타입은 추론이 가능한 경우에 생략할 수 있다.
  > 대부분의 경우 생략 가능하다.
- 예시
  - 기존 람다식 형태
    ```java
    (int a, int b) -> {
      return a > b ? a : b;
    }
    ```
  - `return` 및 매개변수 타입 생략
    ```java
    (a, b) -> a > b ? a : b
    ```
- 단 두 개 이상의 매개변수가 존재할 때, 특정 매개변수의 타입만 생략하는 것은 불가능하다.
  ```java
  (int a, b) -> {...} //불가능
  ```

<br/>

### 괄호 생략
- 선언된 매개변수가 하나뿐인 경우에는 괄호`()`를 생략할 수 있다.
- 단, 매개변수의 타입이 있으면 생략할 수 없다.
- 예시
  - 기존 람다식 형태
    ```java
    (a) -> a * a
    ```
  - 괄호 생략
    ```java
    (a) -> a * a
    ```
- 또한 괄호`{}` 안의 문장이 하나일 때는 괄호`{}`를 생략할 수 있다.  
이때 문장의 끝에 `;`를 붙이지 않아야 한다는 것에 주의하자.
- 예시
  - 기존 람다식 형태
    ```java
    (String name, int i) -> {
      System.out.println(name + "=" i);
    }
    ```
  - 괄호 생략
    ```java
    (String name, int i) -> System.out.println(name + "=" i)
    ```
- 단 괄호`{}` 안의 문장이 `return`문일 경우 괄호`{}`를 생략할 수 없다.

<br/><br/>

## 함수형 인터페이스
### 함수형 인터페이스란?
- 자바에선 람다식을 인터페이스를 통해서 다룬다.
- 그리고 람다식을 다루기 위한 인터페이스를 "함수형 인터페이스"라고 부르기로 했다.

<br/>

### 함수형 인터페이스가 필요한 이유
- 사실 람다식은 **익명 클래스의 객체**와 동등하다. 아래 소스코드 A와 B 모두 동등하다.
- 소스코드 A
  ```java
  (int a, int b) -> a > b ? a : b
  ```
- 소스코드 B
  ```java
  new Object() {
    int max(int a, int b) {
      return a > b ? a : b;
    }
  }
  ```
> 메서드 이름 `max`는 임의로 붙인 것일 뿐 의미는 없다.
- 위 소스코드 A와 B는 모두 동등하므로, 람다식은 아래와 같은 형식을 갖을 수 있다.
  ```java
  타입 f = (int a, int b) -> a > b ? a : b;
  ```
  - 람다식을 참조변수(`f`)에 담았다.
  - 그렇다면 이때 참조변수(`f`)에 필요한 타입은 무엇일까?
- **이때 람다식의 참조를 저장하는 참조변수의 타입이 바로 "함수형 인터페이스"이다.**
- 예시
  - 함수형 인터페이스
    ```java
    interface MyFunction {
      public abstract int max(int a, int b);
    }
    ```
  - 익명 클래스 객체
    ```java
    MyFunction f = new MyFunction() {
      @Override
      public int max(int a, int b) {
        return a > b ? a : b;
      }
    }
    ```
  - 람다식 대체
    ```java
    MyFunction f = (int a, int b) -> a > b ? a : b
    ```
    > **위에서 람다식은 익명 클래스의 객체와 동등하다**고 설명했다.  
    따라서 위처럼 변환이 가능하다.

<br/>

### 함수형 인터페이스의 특징
- `@FunctionalInterface` 애너테이션을 붙이는 것을 권장한다.
  - 해당 애너테이션이 붙은 인터페이스가 함수형 인터페이스임을 컴파일러에게 알려줄 수 있으므로, **함수형 인터페이스를 올바르지 않게 선언한 경우 오류를 검출하기 용이**하다.
- 함수형 인터페이스에는 오직 하나의 추상 메서드만 정의되어 있어야 한다.
  - 그래야만, 람다식과 인터페이스의 메서드가 1대1로 연결될 수 있다.
- 함수형 인터페이스 정의 예시
  ```java
  @FunctionalInterface
  interface MyFunction {
    public abstract int max(int a, int b);
  }
  ```

<br/>

### 람다식을 매개변수로 받는 방법
- 람다식을 특정 메서드의 매개변수로 받으려면 "함수형 인터페이스" 타입을 갖는 매개변수를 선언하면 된다.
- 예시
  - 함수형 인터페이스 선언
    ```java
    @FunctionalInterface
    interface MyFunction {
      public abstract void myMethod();
    }
    ```
  - 실행 코드
    ```java
    public class Main {
      public static void main(String[] args) {
        //----람다식을 전달하는 방법 1----
        MyFunction mf = () -> System.out.println("람다식");
        mainMethod(mf);


        //----람다식을 전달하는 방법 2----
        mainMethod( () -> System.out.println("람다식") );
      }

      /*
       * 함수형 인터페이스 MyFunction을 타입으로 갖는 매개변수 선언
      */
      private static void mainMethod(MyFunction mf) {
        mf.myMethod();
        //...
      }
    }
    ```
- **즉 변수처럼 메서드를 주고받는 것이 가능해졌다.**
  - 사실상 **메서드가 아니라 객체(익명 객체)를 주고받는 것**이라 근본적으로 달라진 것은 아무것도 없지만, 람다식 덕분에 예전보다 코드가 더 간결하고 이해하기 쉬어졌다.

<br/>

### 주의사항
- 함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다.
  - **람다식은 익명 객체이고, 익명 객체는 타입이 없다.**

<br/><br/>

## `java.util.function` 패키지
### 권장사항
- 위에서 직접 함수형 인터페이스를 선언하고 사용하는 방법에 대해 알아보았다.
- **하지만 매번 새로운 함수형 인터페이스를 정의하지 말고, 가능하면 `java.util.function` 패키지의 인터페이스를 활용하는 것이 좋다.**

### 자바 라이브러리에서 제공하는 주요 함수형 인터페이스
|함수형 인터페이스|메서드|설명|용도|
|-----------------|------|----|----|
|java.lang.Runnable|`void run()`|매개변수: 없음 <br/> 반환값: 없음|반환값이 없고, 매개변수도 필요없는 람다식|
|Supplier\<T\>|`T get()`|매개변수: 없음 <br/> 반환값: T형 값|반환값이 T형 값이고, 매개변수가 필요없는 람다식|
|Consumer\<T\>|`void accept(T t)`|매개변수: T형 변수 <br/> 반환값: 없음|반환값이 없고, 매개변수가 T형 변수 하나인 람다식|
|Function\<T, R\>|`R apply(T t)`|매개변수: T형 변수 <br/> 반환값: R형 값|반환값이 R형 값이고, 매개변수가 T형 변수 하나인 람다식|
|Predicate\<T\>|`boolean test(T t)`|매개변수: T형 변수 <br/> 반환값: boolean형 값|매개변수가 T형인 변수 하나를 통해 조건식을 표현하는데 사용|
|BiConsumer\<T, U\>|`void accept(T t, U u)`|매개변수: T형 변수, U형 변수 <br/> 반환값: 없음|반환값이 없고, 매개변수가 T형 변수와 U형 변수로 총 2개인 람다식|
|BiFunction\<T, U, R\>|`R apply(T t, U u)`|매개변수: T형 변수, U형 변수 <br/> 반환값: R형 값|반환값이 R형 값이고, 매개변수가 T형 변수와 U형 변수로 총 2개인 람다식|
|BiPredicate\<T, U\>|`boolean test(T t, U u)`|매개변수: T형 변수, U형 변수 <br/> 반환값: boolean형 값|매개변수가 T형이고 U형인 변수 2개를 통해 조건식을 표현하는데 사용|
|UnaryOperator\<T\>|`T apply(T t)`|매개변수: T형 변수 <br/> 반환값: T형 값|반환값과 매개변수가 모두 T형인 람다식|
|BinaryOperator\<T\>|`T apply(T t1, T t2)`|매개변수: T형 변수 2개 <br/> 반환값: T형 값|반환값과 매개변수가 모두 T형이고, 매개변수가 총 2개인 람다식|

<br/>

### 조건식의 표현에 사용되는 `Predicate`
- `Predicate` 함수형 인터페이스는 `Function` 함수형 인터페이스의 변형으로, 반환타입이 boolean이라는 것만 다르다.
- **`Predicate`는 조건식을 람다식으로 표현하는데 사용된다.**
```java
Predicate<Integer> predicate = (a) -> a > 10;

if (predicate.test(15)) {
  System.out.println("10보다 큰 수입니다.");
}
```

> 나머지 상세한 활용법은 추가적인 자료를 찾아보시길 바란다.

<br/><br/>

## 메서드 참조
### 개요
- 람다식을 더욱 간결하게 표현할 수 있는 방법이 존재한다. 이는 아래와 같다.
- **람다식이 하나의 메서드만 호출하는 경우에는 '메서드 참조'라는 방법으로 람다식을 간략히 할 수 있다.**

<br/>

### 메서드 참조 방법
|종류|람다 형식|메서드 참조 형식|설명|
|----|---------|----------------|----|
|static 메서드 참조|`(a) -> ClassName.method(a)`|`ClassName::method`|`ClassName` : 사용할 정적 메서드가 선언된 클래스명 <br/> `method` : 사용할 정적 메서드명|
|인스턴스 메서드 참조|`(obj, a) -> obj.method(a)`|`ClassName::method`|`ClassName` : 사용할 인스턴스 메서드가 선언된 클래스명 <br/> `method` : 사용할 인스턴스 메서드명|
|특정 객체 인스턴스 메서드 참조|`(a) -> obj.method(a)`|`obj::method`|`obj` : 따로 생성한 객체 <br/> `method` : 따로 생성한 객체의 클래스에 선언된 메서드 중, 사용할 인스턴스 메서드명|
|생성자의 메서드 참조|`(a) -> new ClassName(a)`|`ClassName::new`|`ClassName` : 생성할 객체의 클래스명|

<br/>

### 메서드 참조 예시
1. static 메서드 참조
    ```java
    //기존 람다식
    Function<String, Integer> f = (s) -> Integer.parseInt(s);

    //메서드 참조 활용 시
    Function<String, Integer> f = (s) -> Integer::parseInt;
    ```

2. 인스턴스 메서드 참조
    ```java
    //기존 람다식
    BiFunction<String, String, Boolean> f = (s1, s2) -> s1.equals(s2);

    //메서드 참조 활용 시
    BiFunction<String, String, Boolean> f = (s1, s2) -> String::equals;
    ```

3. 특정 객체 인스턴스 메서드 참조
    ```java
    //특정 객체
    MyClass myInstance = new MyClass();

    //기존 람다식
    Function<String, Integer> f = (s) -> myInstance.myMethod(s);

    //메서드 참조 활용 시
    Function<String, Integer> f = myInstance::myMethod;
    ```

4. 생성자 메서드 참조
    ```java
    //기존 람다식
    Function<String, MyClass> f = (s) -> new MyClass(s);

    //메서드 참조 활용 시
    Function<String, MyClass> f = MyClass::new;
    ```

<br/>

### 생략된 부분을 컴파일러가 처리할 수 있는 이유
- 아래 예시를 다시보자. 해당 예시를 통해 설명하도록 하겠다.
  ```java
  //기존 람다식
  Function<String, Integer> f = (s) -> Integer.parseInt(s);

  //메서드 참조 활용 시
  Function<String, Integer> f = (s) -> Integer::parseInt;
  ```
- 컴파일러는 아래 내용을 통해, 생략된 부분의 내용을 알 수 있다.
  - 우변(`Integer::parseInt`)의 `parseInt` 메서드의 선언부
  - 좌변(`Function<String, Integer>`)의 `Function` 인터페이스에 지정된 제네릭 타입

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>남궁성, 『Java의 정석 3rd Edition』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>