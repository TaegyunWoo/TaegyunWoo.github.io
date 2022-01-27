---
category: Java
tags: [Java]
title: "Comparable 과 Comparator 인터페이스"
date:   2022-01-27 19:15:00 
lastmod : 2022-01-27 19:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Comparable과 Comparator
## 개요
### Comparable이란?
- Comparable은 `java.lang`에 정의된 인터페이스이다.
- Comparable은 `compareTo(Object o)` 메서드를 정의하고 있다.
- 컬렉션을 정렬하는데 필요한 메서드를 정의하고 있다.
- Comparable 인터페이스를 구현하고 있는 클래스들은 같은 타입의 인스턴스끼리 서로 비교할 수 있는 클래스들, 주로 `Integer`와 같은 wrapper 클래스와 `String`, `Data`, `File` 과 같은 클래스들이 있다.
- 기본적으로 오름차순, 즉 작은 값에서부터 큰 값의 순으로 정렬되도록 구현되어 있다.
- **따라서 Comparable 인터페이스를 구현한 클래스는 정렬이 가능하다는 것을 의미한다.**
```java
public interface Comparable {
  public int compareTo(Object o);
}
```

<br/>

### Comparator이란?
- Comparator는 `java.util`에 정의된 인터페이스이다.
- Comparator는 `compare(Object o1, Object o2)` 메서드를 정의하고 있다.
- 컬렉션을 정렬하는데 필요한 메서드를 정의하고 있다.
- Comparable 인터페이스와 동일한 목적(객체 비교)를 갖는다.
```java
public interface Comparator {
  public int compare(Object o1, Object o2);
}
```

<br/>

### Comparable VS Comparator
- Comparable 인터페이스
  - 기본 정렬기준을 구현하는데 사용한다.
  - 비교할 객체에 기준을 **내장(정의)** 시키기 위한 인터페이스이다.
  - 따라서 비교할 객체(클래스) 자체에 implement 해야한다.

<br/>

- Comparator 인터페이스
  - 기본 정렬기준 외에 다른 기준으로 정렬하고자할 때 사용한다.
  - 새로운 '비교 기준 객체'를 위한 인터페이스이다.
  - 다른 정렬 기준이 필요한 곳에 직접 전달하여, 해당 기준(Comparator 객체)을 적용시킬 수 있다.

<br/><br/>

## 구현 방법
### 예시 목표
- 예시를 통해, Comparator와 Comparable을 구현하는 방법을 알아보자.  
예시 목표는 아래와 같다.
  - Person 클래스의 인스턴스를 나이 오름차순으로 정렬하고자 한다.
- 위 목표를 위해, Comparable 인터페이스나 Comparator 인터페이스를 사용할 수 있다.
- 계속해서 코드를 통해 알아보자.

<br/>

### Comparable 인터페이스의 `compareTo(Object o)` 메서드 구현
- Person 클래스의 인스턴스를 나이 오름차순으로 정렬하기 위해, Comparable 인터페이스를 사용하여 아래처럼 구현할 수 있다.

- `Person` 클래스
  ```java
  public class Person implements Comparable<Person> {
    private String name;
    private int age;

    //생성자, getter, setter 생략

    @Override
    public int compareTo(Person other) {
      if (this.age > other.age) {
        return 1;
      } else if (this.age == other.age) {
        return 0;
      } else if (this.age < other.age) {
        return -1;
      }
    }

  }
  ```
  - '매개변수로 전달받은 객체(`Person other`)'보다 '현재 객체(`this`)'의 age 필드값이 더 크다면 양수를 반환해야 한다.
  - '매개변수로 전달받은 객체(`Person other`)'와 '현재 객체(`this`)'의 age 필드값이 같다면 0을 반환해야 한다.
  - '매개변수로 전달받은 객체(`Person other`)'보다 '현재 객체(`this`)'의 age 필드값이 더 작다면 음수를 반환해야 한다.

- 결과 출력
    ```java
    public class Main {
      public static void main(String[] args) {

        Person personA = new Person("홍길동", 15);
        Person personB = new Person("이순신", 29);
        Person personC = new Person("소년", 10);
        Person[] ary = {personA, personB, personC};
        
        Arrays.sort(ary); //자체 정렬 기준(compareTo 메서드) 사용

        for (Person p : ary) {
          System.out.println(p.getName());
        }
      }
    }
    ```
  - 결과
    ```text
    소년
    홍길동
    이순신
    ```

<br/>

### Comparator 인터페이스의 `compare(Object o1, Object o2)` 메서드 구현
- Person 클래스의 인스턴스를 나이 오름차순으로 정렬하기 위해, Comparator 인터페이스를 사용하여 아래처럼 구현할 수 있다.

- `Person` 클래스
  ```java
  public class Person {
    private String name;
    private int age;

    //생성자, getter, setter 생략
  }
  ```
  - 따로 `정렬 기준 객체(Comparator 구현체)`를 만들어 사용하기 때문에, `Person` 클래스는 단순하다.

- `PersonOrder` 클래스
  ```java
  public class PersonOrder implements Comparator<Person> {
    @Override
    public int compare(Person me, Person other) {
      if (this.age > other.age) {
        return 1;
      } else if (this.age == other.age) {
        return 0;
      } else if (this.age < other.age) {
        return -1;
      }
    }
  }
  ```
  - 구현 내용은 Comparable 인터페이스의 `compareTo` 메서드와 같다.

- 결과 출력
    ```java
    public class Main {
      public static void main(String[] args) {

        Person personA = new Person("홍길동", 15);
        Person personB = new Person("이순신", 29);
        Person personC = new Person("소년", 10);
        Person[] ary = {personA, personB, personC};
        
        // Arrays.sort(ary);
        /* 위 코드는 오류가 발생한다.
        * 왜냐하면, Person 클래스에 정렬 기준이 없기 때문이다.
        */


        Arrays.sort(ary, new PersonOrder()); //새로운 정렬 기준(PersonOrder 클래스의 compare 메서드) 사용

        for (Person p : ary) {
          System.out.println(p.getName());
        }
      }
    }
    ```
  - 결과
    ```text
    소년
    홍길동
    이순신
    ```

<br/>

### 권장 사항
- 위에서 `Comparable` 과 `Comparator` 를 각각 사용하여, `Person` 객체를 정렬하였다.
- `Comparable` 과 `Comparator` 중 어떤 것을 사용해서 정렬하여도 상관없지만, 위 경우 `Comparable` 인터페이스를 구현하여 사용하는 것이 더욱 자연스럽다.
- **왜냐하면 `Comparable` 인터페이스는 기본 정렬기준을 구현하는데 사용되기 때문이다.**
- 반면에, 만약 아래와 같이 `Comparable` 인터페이스를 구현한 상태에서 나이 내림차순으로 정렬을 하고 싶다면 어떻게 할까?
  ```java
  public class Person implements Comparable<Person> {
    private String name;
    private int age;

    //생성자, getter, setter 생략

    /**
    * 기본적으로 나이 오름차순 정렬을 하도록 정의함.
    */
    @Override
    public int compareTo(Person other) {
      if (this.age > other.age) {
        return 1;
      } else if (this.age == other.age) {
        return 0;
      } else if (this.age < other.age) {
        return -1;
      }
    }

  }
  ```
- 바로 이런 경우, `Comparator` 인터페이스 구현체를 사용하면 된다.
  ```java
  public class PersonOrder implements Comparator<Person> {
    /**
    * 나이 내림차순 정렬을 하도록 새로운 기준을 정의함.
    */
    @Override
    public int compare(Person me, Person other) {
      if (this.age > other.age) {
        return -1;
      } else if (this.age == other.age) {
        return 0;
      } else if (this.age < other.age) {
        return 1;
      }
    }
  }
  ```
- 실행 코드
  ```java
  Person personA = new Person("홍길동", 15);
  Person personB = new Person("이순신", 29);
  Person personC = new Person("소년", 10);
  Person[] ary = {personA, personB, personC};

  Arrays.sort(ary); //나이 오름차순 정렬 (기본 정렬)
  /* 결과
  * personC, personA, personB
  */


  Arrays.sort(ary, new PersonOrder()); //나이 내림차순 정렬
  /* 결과
  * personB, personA, personC
  */
  ```