---
category: Interview
tags: [Interview]
title: "[Interview Study] Array vs ArrayList vs LinkedList"
date:   2023-01-13 23:00:00 
lastmod : 2022-01-13 23:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Array vs ArrayList vs LinkedList

## Array

![Untitled](/assets/img/2023-01-13-Interview_ArrayVsArrayListVsLinkedList/Untitled.png)

### 특징

- 같은 타입의 데이터를 **여러개 나열한 선형 자료구조**이다.
- **연속적인 메모리 공간**에 순차적으로 데이터를 저장한다.
- **크기가 고정적**이다.

### 장점

- index를 사용해 **값을 빠르게 찾을 수 있다.**
- 구조가 간단하다.
- 참조를 위한 **추가적인 메모리(포인터)가 필요하지 않다.**

### 단점

- 배열의 **크기를 변경할 수 없다.**
    - 따라서 최대 사이즈를 알 수 없는 경우, 사용하기 부적절하다.
- 메모리 낭비가 발생할 수 있다.
    - 배열을 선언할 때, 배열의 크기만큼 메모리를 미리 할당해둔다. 따라서 사용하지 않는 공간이라도 메모리를 차지한다.
- 중간에 데이터를 **삽입하거나 삭제하기 어렵다.**
    - 데이터를 삽입하는 경우, 삽입할 위치 이후의 모든 요소들을 뒤로 한칸씩 미뤄야한다. → 따라서 **가장 마지막 요소는 제거**된다.
    - 데이터를 제거하는 경우, 제거할 요소 이후의 모든 요소들을 앞으로 한칸씩 미뤄야 한다. → 따라서 **가장 마지막 요소에 임의의 값**을 넣어야 한다.

<br/><br/>

## ArrayList

![Untitled](/assets/img/2023-01-13-Interview_ArrayVsArrayListVsLinkedList/Untitled%201.png)

### 특징

- **크기를 동적으로 변경**할 수 있다.
    - 미리 충분한 크기의 물리적 메모리(`capacity`)를 할당해두고, 논리적 메모리(`size`) 크기를 관리한다.
    - 물리적 메모리 이상의 크기가 필요하면, 용량을 확장한다.
- Array 와 마찬가지로 **index를 가지고 있다.**
- Array 와 마찬가지로 **연속적인 메모리 공간**에 순차적으로 데이터를 저장한다.

### 장점

- **크기를 동적으로 할당**할 수 있기 때문에 편리하다.
- 데이터를 삽입하거나 삭제할 때, **마지막 요소에 대한 부가적인 처리를 하지 않아도 된다.**
- index를 통해, **데이터에 빠르게 접근**할 수 있다.

### 단점

- 여전히 요소를 중간에 **삽입하거나 삭제할 때 비효율적**이다.
    - 삽입할 위치 이후의 모든 요소를 한칸 뒤로 미뤄야 한다.
    - 삭제할 위치 이후의 모든 요소를 한칸 앞으로 미뤄야 한다.
- 크기를 변경할 수 있지만 **물리적 용량이 변경될 때마다, 오버헤드가 발생**한다.

### ArrayList가 동적으로 크기를 변경하는 원리 (Java)

- ArrayList 는 **Object 배열을 이용**해서 요소를 순차적으로 저장한다.
- 처음에는 10개의 요소를 저장할 수 있는 크기로 Array를 생성한다.
    - 이 Array의 크기가 물리적 메모리 크기 (`capacity`)이다.
    - 요소를 추가·삭제함에 따라, 논리적 메모리 크기 (`size`)를 관리한다.
- 만약 11번째 요소를 저장하려고 하면, **기존 1.5배 크기의 Object 배열을 다시 선언**한다.
- 그리고 **기존 요소들을 새 Object 배열에 모두 복사**하고, 11번째 요소를 저장한다.
    - 이 과정에서 오버헤드가 발생한다.

![Untitled](/assets/img/2023-01-13-Interview_ArrayVsArrayListVsLinkedList/Untitled%202.png)

![Untitled](/assets/img/2023-01-13-Interview_ArrayVsArrayListVsLinkedList/Untitled%203.png)

<br/><br/>

## LinkedList

![Untitled](/assets/img/2023-01-13-Interview_ArrayVsArrayListVsLinkedList/Untitled%204.png)

### 특징

- **한 노드에 연결될 노드의 포인터 위치를 가리키는 방식**으로 구성된다.
    - 즉 연결된 노드의 주소 정보를 가지고 있다.
- 단일 LinkedList 는 다음 요소의 주소정보만 가지고 있다.
- 다중 LinkedList 는 이전·다음 요소의 주소정보를 가지고 있다.
- index가 없다.

### 장점

- 데이터 **삽입·삭제가 용이**하다.
    - 요소들을 밀 필요없이, **가르키는 포인터만 수정**해주면 된다.

### 단점

- **데이터 검색이 느리다.**
    - k번째 요소를 찾을 땐 무조건 **처음부터 k번째 요소까지 순차적으로** 살펴봐야 한다.

## 정리

- 고정된 크기로 데이터 삽입·삭제 작업이 적다면 → Array 사용
- 데이터 삽입·삭제 작업이 드물지만 크기를 알 수 없다면 → ArrayList 사용
- 데이터 삽입·삭제 작업이 많고, 데이터 조회 작업이 드물다면 → LinkedList 사용