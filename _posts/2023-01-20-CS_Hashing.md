---
category: Interview
tags: [Interview]
title: "[CS Study] Java에서의 Vector와 ArrayList"
date:   2023-01-19 20:00:00 
lastmod : 2022-01-19 20:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# Java Vector / ArrayList

## Vector와 ArrayList

### Vector란?

- Vector는 **확장 가능한 배열의 구현체**이다.
    - **List 인터페이스의 구현체**이다.
- Java Collections 프레임워크가 없던 시절, **초기 자바 버전에 포함**이 되어 있었다.
- 현재는 재설계되어, Collections 프레임워크에 포함되었다.

> 참고로 Stack 클래스는 Vector 클래스의 자식 클래스이다.

### Vector의 특징

- **정수 인덱스**를 사용해, 배열에 액세스할 수 있다.
- **동기화(Thread-Safe)**되어있고, **한번에 하나의 쓰레드만 벡터의 메서드를 호출**할 수 있다.

### ArrayList란?

- ArrayList도 **확장 가능한 배열의 구현체**이다.
    - 마찬가지로 **List 인터페이스의 구현체**이다.

### ArrayList의 특징

- **정수 인덱스**를 사용해, 배열에 액세스할 수 있다.
- **동기화되어있지 않다.**

## 주요 차이점

### 동기화 여부

- Vector는 동기화 처리가 이미 되어있어, **Thread-Safe 하게 동작**한다.
- 즉 한번에 여러 쓰레드가 Vector 객체에 동시에 접근할 수 없고, **한번에 하나의 쓰레드가 동작**하게 된다.
- ArrayList는 동기화 처리가 되어있지 않아, **동시성 문제가 발생**할 수 있다.

### 성능

- ArrayList는 동기화되지 않았기 때문에, Vector보다 더 빠르다.

### 배열 확장 크기

- Vector는 모든 공간이 다 차면, capacity를 **기존 크기의 2배로 확장**한다.
- ArrayList는 capacity를 **기존 크기의 1.5배로 확장**한다.

## Vector 와 ArrayList 중 선택

### 멀티쓰레드 환경인 경우

- Vector 사용을 고려해볼 수 있다.
- 혹은 ArrayList 에 접근할 때 동기화 처리를 한다면, ArrayList를 사용해도 좋다.
    - `Collections.synchronizedList` 메서드를 활용해도 좋다.

### 멀티쓰레드 환경이 아닌 경우

- 성능이 더 좋은 ArrayList를 사용하자.