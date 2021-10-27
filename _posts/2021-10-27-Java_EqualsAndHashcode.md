---
category: Java
tags: [Java, Hash]
title: "equals 와 hashCode 메서드에 대하여"
date:   2021-10-27 17:15:00 
lastmod : 2021-10-27 17:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 개요

이번 포스팅은 Java가 제공하는 메서드 `equals` 와 `hashCode`에 대해 알아볼 것이다. 먼저 이 메서드들이 어떤 기능을 하는지 알아보고, 왜 이런 기능들이 필요한지, 어디에서 사용되는지 알아볼 것이다.  
> 참고로, HashTable이나 HashMap과 같은 자료구조가 어떻게 동작하는지 알고 있어야 본 글을 이해할 수 있다. 해당 자료구조와 알고리즘에 대한 내용은 [[알고리즘] 해시알고리즘](https://taegyunwoo.github.io/algorithm/ALGORITHM_HashAlgorithm)을 참고하자.

<br/><br/>

# `equals`와 `hashCode` 메서드의 기능
## `equals` 메서드의 기능
### 동일성과 동등성

먼저 `equals` 메서드가 어떤 기능을 하는지 알아보기 전에, 동일성 비교와 동등성 비교에 대해 알아보자. 동일성과 동등성에 대한 내용은 아래와 같다.

- 동일성
  - 객체의 참조(주소값)를 비교한다.
  ```java
  Member memberA = new Member(); //참조 = 0x10
  Member memberB = new Member(); //참조 = 0x20
  System.out.println(memberA == memberB); // 결과: false
  ```
  - 위 코드에서 인스턴스 `memberA`와 `memberB`는 서로 다른 메모리 주소에 위치한다. 왜냐하면 `new` 연산자를 통해, **서로 다른 메모리를 할당받아 인스턴스를 생성했기 때문**이다.
  - 따라서, `==` 연산자를 통해 동일성을 비교하면 결과가 false이다.

<br/>

- 동등성
  - 객체의 내용(필드값)을 비교한다.
  ```java
  // [Member 클래스의 equals 메서드가 동등성 비교를 위해,
  // 적절하게 오버라이딩되었다고 가정하자.]
  Member memberA = new Member(); //참조 = 0x10
  Member memberB = new Member(); //참조 = 0x20

  // [Member 클래스에는 id이라는 필드가 하나 존재한다고 가정하자.]
  //서로 같은 필드값으로 설정되었다.
  memberA.setId(1);
  memberB.setId(1);

  System.out.println(memberA.equals(memberB)); // 결과: true
  ```
  - 위 코드에서, 인스턴스 `memberA`와 `memberB`는 서로 다른 주소값을 갖는다.
  - 이때 두 인스턴스에 대해, `equals` 메서드로 동등성 비교를 하면 결과가 true이다.
  - **`Member` 클래스의 유일한 필드인 `id`의 값이 서로 같기 때문**이다.

<br/>

### `equals` 메서드의 기능
위에서 살펴보았듯이, `equals` 메서드는 두 객체 간의 동등성을 비교하기 위한 메서드이다. 해당 메서드는 `Object`에 선언되어있으며, Java의 모든 객체는 Object를 상속받았으므로 **모든 객체 간의 동등성을 비교할 수 있는 것**이다.  

하지만, `Object`에 선언된 `equals` 메서드를 그저 사용만 하면 되는 것은 아니다. 해당 메서드는 기본적으로 아래와 같이 선언되어있다.

> 실제로 아래와 완전히 동일하다는 것은 아니다. 이해를 위해, 아래와 같이 설명한다.

```java
public class Object {

  //...

  public boolean equals(Object obj) {
      return this == obj; //동일성을 비교한다.
  }

}
```

즉, 기본적인 `equals` 메서드는 동일성을 비교하는 내용을 작성되어있다. **따라서, 해당 메서드를 오버라이드하여 동등성을 비교할 수 있도록 수정해야 한다.** 이에 대한, 예시 코드는 아래와 같다.

```java
public class Member {
  private int id;

  @Override
  public boolean equals(Object obj) {
    return this.id == obj.id; //내용만을 비교한다.
  }

}
```

<br/>

## `hashCode` 메서드의 기능

위에서 `equals` 메서드에 대해 알아보았다. 이번에는 `hashCode` 메서드에 대해 알아보자. 이 메서드는 `equals`와 같이 `Object` 클래스에 선언되어 있다. 따라서, 모든 객체에서 사용할 수 있다.  

이 메서드는 현재 인스턴스의 해시값을 반환하는 메서드이다. 즉, 해시함수의 역할을 수행하는 메서드라고 생각하면 된다. 이때, 각 객체의 해시코드는 무조건 다르게 나오는 것이 좋은 것일까? 정답은 아니다. 물론 해시코드 값이 동일하게 나오지 않는다면, 해시 관련 자료구조에서 좋은 성능을 보여줄 수 있다. **하지만, 같은 내용을 가지고 있는 객체들이라면 모두 같은 해시코드를 반환해주어야 한다.** 만약 서로 같은 내용을 가지는 객체들의 해시코드가 다르다면, HashTable이나 HashMap에서 이들을 찾을 방도가 없다.  

> 다시한번 말하지만, 해시알고리즘에 대해 잘 모른다면 [이전 게시글](https://taegyunwoo.github.io/algorithm/ALGORITHM_HashAlgorithm)을 참고하자.  

<br/>

`hashCode` 메서드는 `equals` 메서드와 마찬가지로 **오버라이딩을 통해 직접 구현**해야 한다. 왜냐하면 이것 역시, `Object` 클래스에 아래 내용과 같이 선언되어 있기 때문이다.  

> 이것 역시, 실제로 아래와 완전히 동일하다는 것은 아니다. 이해를 위해, 아래와 같이 설명한다.  

```java
public class Object {

  //...

  public int hashCode() {
      // 객체의 메모리주소를 기반으로 값을 반환한다.
  }

}
```

<br/>

즉, 기존에 작성된 `hashCode` 메서드는 **객체의 주소를 기반으로 해시코드 값을 반환**한다. 따라서 서로 같은 내용을 갖는 객체 간의 해시코드 값이 다르기 출력되기 때문에, **서로 같은 내용을 갖는다면 같은 해시코드를 반환하도록 적절히 오버라이딩**해주어야 한다. 이에 대한 예시코드는 아래와 같다.  

```java
public class Member {
  private int id;

  @Override
  public int hashCode() {
    return id; // 주소가 아닌, 내용을 기준으로 해시코드를 반환한다.
  }

}
```

<br/><br/>

# `equals`와 `hashCode`를 같이 오버라이딩해야 하는 이유
지금까지 `equals`와 `hashCode` 메서드가 어떤 기능을 하고, 왜 오버라이딩을 하여 사용해야 하는지 알아보았다. 하지만 여기서 의문이 생긴다. 보통 사람들은 이 두 가지를 모두 동시에 오버라이딩을 해야 한다고 한다. 그 이유는 무엇일까?  

그에 대한, 답을 듣기 위해선 먼저 HashTable, HashMap 등의 자료구조가 어떻게 동작하는지 알아야 한다. 참고로 HashTable과 HashMap 등은 큰 작동원리가 모두 같으므로, HashTable을 통해 설명하겠다.  

<br/>

## HashTable의 동작원리
HashTable은 아래 그림과 같이 **저장**한다.  

![untitled](/assets/img/2021-10-27-Java_EqualsAndHashcode/Untitled.png)

1. HashTable에 넣고자 하는 (Key, Value) 쌍에서 Key만을 Hash 함수에 넣는다.
2. Hash 함수가 Key 값을 index 값으로 변환해준다.
3. 해당 index에 해당하는 버킷에 (Key, Value)를 하나의 Entry로 저장한다.
    - 이때, 다른 Entry가 이미 존재하면 연결리스트 형태로 저장된다.

이러한, 과정에서 `equals`와 `hashCode` 메서드가 사용된다.

<br/>

## HashTable과 `hashCode` 메서드

HashTable에 특정 데이터를 put하거나 get하려면, **Key 값을 hash화 해야 한다.**  이때, `hashCode` 메서드가 사용된다.  

![untitled](/assets/img/2021-10-27-Java_EqualsAndHashcode/Untitled%201.png)

- 이때 만약, `hashCode` 메서드가 적절히 오버라이딩되지 않았다면?
  - `hashTable.get(key객체)`를 통해, 원하는 Entry에 접근할 수 없다.
  - 왜냐하면, **key 객체의 모든 필드 값이 같더라도 다른 해시코드를 반환하기 때문**이다.
  - **따라서, `hashCode` 메서드가 반드시 오버라이딩되어야 한다.**

<br/>

## HashTable과 `equals` 메서드
### HashTable에 특정 데이터를 put할 때

- 연결리스트에 존재하는 여러 Entry 중 **Key 값을 대상으로 `equals` 메서드를 적용하여 비교**한다.
- 만약 '연결리스트에 존재하는 모든 Entry의 Key'가 'put하려는 Entry의 Key'와 다를 경우, 연결리스트에 'put하려는 Entry'를 추가한다.
- 만약 '연결리스트에 존재하는 Entry의 Key'가 'put하려는 Entry의 Key'와 같을 경우, 해당 Entry를 'put하려는 Entry'로 덮어쓴다.  

![untitled](/assets/img/2021-10-27-Java_EqualsAndHashcode/Untitled%202.png)

<br/>

### HashTable에 특정 데이터를 get할 때

- 연결리스트에 존재하는 여러 Entry 중 **Key 값을 대상으로 `equals` 메서드를 적용하여 비교**한다.
- 만약 '연결리스트에 존재하는 모든 Entry의 Key'가 'get하려는 Key'와 다를 경우, null을 반환한다.
- 만약 'get하려는 Key'와 같은 Entry를 연결리스트에서 찾은 경우, 해당 Entry를 반환한다.  

![untitled](/assets/img/2021-10-27-Java_EqualsAndHashcode/Untitled%203.png)

## 정리
지금까지 `equals`와 `hashCode` 메서드의 목적과 사용방법 그리고 HashTable에서의 사용까지 알아보았다. 지금까지 설명한 내용을 정리하자면 다음과 같다.

- `equals` 메서드
  - `equals` 메서드는 동등성 비교를 위해 사용된다.
  - 동등성 비교를 하기 위해선, `equals` 메서드를 적절히 오버라이딩해야 한다.
  - `equals` 메서드는 HashTable이나 HashMap의 연결리스트에 존재하는 Entry에 대하여, Key 객체의 동등성 비교를 통해 Entry를 덮어씌우거나 가져올 수 있다.

<br/>

- `hashCode` 메서드
  - `hashCode`는 해시코드 값을 반환시킨다.
  - 같은 내용을 갖는 인스턴스에 대해 같은 해시코드 값을 반환받기 위해선, `hashCode` 메서드를 적절히 오버라이딩해야 한다.
  - `hashCode` 메서드는 HashTable이나 HashMap에서 버킷에 대한 index를 찾기 위해 사용된다.

<br/>

- **따라서, 반드시 `equals`와 `hashCode` 메서드를 같이 오버라이딩해야 한다. 그렇지 않으면, Hash 관련 자료구조를 사용하기 어렵다.**

![untitled](/assets/img/2021-10-27-Java_EqualsAndHashcode/Untitled%204.png)