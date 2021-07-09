---
category: 데이터구조
tags: [데이터구조, 자료구조, 덱]
title: "데이터구조 [덱]"
date:   2021-07-03 12:00:00 
lastmod : 2021-07-03 12:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 덱

## 개요

### 덱이란?

- **전단(front)과 후단(rear)** 에서 모두 삽입, 삭제가 가능한 큐

![덱 개요](/assets/img/데이터구조_덱/Untitled_33.png)

<br/>

### 덱 연산 예시

![덱 연산 예시](/assets/img/데이터구조_덱/Untitled_34.png)

<br/>

### 특징

- 덱(deque) 구현
  - 배열
- 큐와 유사함
- 입력 방향이 쌍방이다

<br/><br/>

## 덱 ADT

### 데이터

- 전단과 후단을 통한 접근을 허용하는 요소들의 모음

### 연산

- **init()** : 덱 초기화
- **add_front(e)** : 요소 e를 덱 맨 앞에 추가
- **delete_front()** : 전단 요소 삭제 후 반환
- **add_rear(e)** : 요소 e를 덱 맨 뒤에 추가
- **delete_rear()** : 후단 요소 삭제 후 반환
- **get_front()** : 전단 요소를 삭제하지 않고 반환
- **get_rear()** : 후단 요소를 삭제하지 않고 반환
- **is_empty()** : 공백 상태 → true
- **is_full()** : 포화 상태 → true
- **size()** : 덱 내의 모든 요소들의 개수

<br/><br/>

## 원형 덱

### 큐와 동일한 연산

- init()
- delete_front()
- add_rear(e)
- get_front()
- is_empty()
- is_full()
- size()

<br/>

### 덱에서 추가된 연산(큐에 없는 연산)

- **delete_rear()**
- **add_front()**
- **get_rear()**

> - add_rear() == 큐의 enqueue()
> - delete_front() == 큐의 dequeue()

<br/>

### delete_rear(), add_front() 공식

- delete_rear() 실행시 rear값
  - **rear = (rear - 1 + MAX_DEQUE_SIZE) % MAX_DEQUE_SIZE**
- add_front() 실행시 front값
  - **front = (front - 1 + MAX_DEQUE_SIZE) % MAX_DEQUE_SIZE**

> <MAX_DEQUE_SIZE을 더하는 이유>  
rear값, front값이 0일때, 음수가 나오지 않게 하기 위해

<br/>

### 구현 코드

```c
#include <stdio.h>
#define Element int
#define MAX_DEQUE_SIZE 100

Element deque[MAX_DEQUE_SIZE];
int front;
int rear;

void init_deque() {
	front=rear=0;
}

int size_deque() {
	return (rear-front+MAX_DEQUE_SIZE)%MAX_DEQUE_SIZE;
}

int is_empty() {
	return front==rear;
}

int is_full() {
	return front==(rear+1)%MAX_DEQUE_SIZE;
}

void add_rear(Element e) {
	if(is_full()) {
		printf("Deque Overflow");
		return;
	}
	rear = (rear+1)%MAX_DEQUE_SIZE;
	deque[rear] = e;
}

Element delete_rear() {
	if(is_empty()) {
		printf("Deque Underflow");
		return;
	}
	int temp = rear;
	rear = (rear-1+MAX_DEQUE_SIZE)%MAX_DEQUE_SIZE;
	return deque[temp];
}

void add_front(Element e) {
	if(is_full()) {
		printf("Deque Overflow");
		return;
	}
	deque[front] = e;
	front = (front-1+MAX_DEQUE_SIZE)%MAX_DEQUE_SIZE;
}

Element delete_front() {
	if(is_empty()) {
		printf("Deque Underflow");
		return;
	}
	front = (front+1)%MAX_DEQUE_SIZE;
	return deque[front];
}

Element get_rear() {
	if(is_empty()) {
		printf("Deque Underflow");
		return;
	}
	return deque[rear];
}

Element get_front() {
	if(is_empty()) {
		printf("Deque Underflow");
		return;
	}
	return deque[(front-1+MAX_DEQUE_SIZE)%MAX_DEQUE_SIZE];
}

int main() {
	init_deque();
	int i;
	for(i=0; i<10; i++) {
		if(i%2) {
			add_front(i);
		} else {
			add_rear(i);
		}
	}
	
	for(i=0; i<10; i++) {
		printf("%d ", delete_front());
	}
		
}
```