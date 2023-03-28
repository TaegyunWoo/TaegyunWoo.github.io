---
category: DataStructure
tags: [DataStructure]
title: "[데이터구조] 큐"
date:   2021-07-03 12:00:00 
lastmod : 2021-07-03 12:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 큐

## 개요

### 특징

- **선입선출**

<br/><br/>

## 큐 ADT

### 특징

- **삽입** : 큐의 후단(rear)
- **삭제** : 큐의 전단(front)

<br/>

### 데이터

선입선출의 접근 방법을 유지하는 요소들의 모음

<br/>

### 연산

- **init()** : 큐를 초기화한다.
- **enqueue(e)** : 주어진 요소 e를 큐의 맨 뒤에 추가
- **dequeue()** : 큐가 비어있지 않으면 맨 앞 요소를 삭제하고 반환한다.
- **is_empty()** : 큐가 비어있으면 true
                    큐가 비어있지 않다면 false
- **peek()** : 큐가 비어있지 않으면 맨 앞 요소를 삭제하지 않고 반환한다.
- **is_full()** : 큐가 가득 차 있으면 true
               큐가 가득 차 있지 않다면 false
- **size()** : 큐의 모든 요소들의 개수 반환

![Enqueue, Dequeue 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_22.png)

<br/><br/>

## 선형큐

### 특징

- **선형으로 큐를 구현**

<br/>

### 원리

![선형큐 Enqueue 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_23.png)

<br/>

### 선형큐의 문제점

- 삽입을 계속하기 위해서는 **요소들을 이동**시켜야 함 (dequeue 할 때 마다)

![선형큐의 문제점](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_24.png)

> 해결법 : 원형큐

<br/><br/>

## 원형 큐

### 구조

- **front** : 첫 번째 요소 하나 앞의 인덱스
- **rear** : 마지막 요소의 인덱스

<br/>

### 원형 큐의 삽입 삭제 연산

![원형 큐 Enqueue, Dequeue 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_25.png)

<br/>

### 상태

- 공백상태
- front == rear
- 포화상태
- M : 원형큐의 최대크기
- front%M == (rear+1)%M

> <문제점><br/>
원형큐의 모든 공간에 요소가 있다면, "front == rear == 0"이 된다.
이것은 공백상태와 동일한 상태로, 포화상태와 공백상태의 구분이 불가능하다.  
⇒ **해결책 : 하나의 공간은 남겨둔다.**

![원형큐 마지막 공간 남기기](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_26.png)

<br/><br/>

## 알고리즘

### init() 연산

- 공백상태로 만드는 것
- front와 rear 동일한 값으로 설정
- 모두 0으로 초기화

![초기화 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_27.png)

<br/>

### size() 연산

![size 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_28.png)

> <MAX_QUEUE_SIZE를 더한 뒤, 다시 MAX_QUEUE_SIZE로 나누는 이유>  
**rear<front 일때, 음수가 나오지 않게 하기 위해**

<br/>

### is_empty() 연산

![is_empty 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_29.png)

<br/>

### is_full() 연산

![is_full 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_30.png)

<br/>

### 삽입(enqueue) 연산

- 나머지 연산을 사용하여 인덱스를 원형으로 회전시킨다.
(index가 큐의 최대 크기를 넘지 못하도록)

![삽입 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_31.png)

<br/>

### 삭제(dequeue) 연산

- 나머지 연산을 사용하여 인덱스를 원형으로 회전시킨다.
(index가 큐의 최대 크기를 넘지 못하도록)

![삭제 연산](/assets/img/2021-07-03-DATASTRUCTURE_Queue/Untitled_32.png)

<br/><br/>

## 구현

### 초기변수설정

```c
#define Element int
#define MAX_QUEUE_SIZE 100

Element data[MAX_QUEUE_SIZE];
int front;
int rear;
```

### init()

```c
void init() {
	front = rear = 0;
}
```

### is_empty()

```c
int is_empty() {
	return front==rear;
}
```

### is_full()

```c
int is_full() {
	return front == (rear+1)%MAX_QUEUE_SIZE;
}
```

### size()

```c
int size() {
	return (rear - front + MAX_QUEUE_SIZE) % MAX_QUEUE_SIZE;
}
```

### print_queue()

```c
void print_queue(char msg[]) {
	int i, maxi = rear;
	if(front >= rear) {
		maxi += MAX_QUEUE_SIZE;
	}
	printf("%s[%2d] = ", msg, size());
	for(i=front+1; i<=maxi; i++) {
		printf("%2d ", queue[i%MAX_QUEUE_SIZE]);
	}
	printf("\n");
}
```

### 삽입(enqueue)

```c
void enqueue(Element val) {
	if( is_full() ) {
		error("큐 포화 에러");
	}
	rear = (rear+1) % MAX_QUEUE_SIZE;
	queue[rear] = val;
}
```

### 삭제(dequeue)

```c
Element dequeue() {
	if( is_empty() ) {
		error("큐 공백 에러");
	}
	front = (front+1) % MAX_QUEUE_SIZE;
	return queue[front];
}
```

### 살펴보기(peek)

```c
Element peek() {
	if(is_empty()) {
		error("큐 공백상태");
	}
	return queue[(front+1)%MAX_QUEUE_SIZE];
}
```

### 종합예제

```c
#include <stdio.h>
#define Element int
#define MAX_QUEUE_SIZE 100

Element queue[MAX_QUEUE_SIZE];
int front;
int rear;

void init() {
	front = rear = 0;	
}

int size() {
	return (rear-front+MAX_QUEUE_SIZE) % MAX_QUEUE_SIZE;
}

int is_full() {
	return front == (rear+1)%MAX_QUEUE_SIZE;
}

int is_empty() {
	return front == rear;
}

void enqueue(Element e) {
	if(is_full()) {
		error("큐 포화상태");
	}
	rear = (rear+1) % MAX_QUEUE_SIZE;
	queue[rear] = e;
}

Element dequeue() {
	if(is_empty()) {
		error("큐 공백상태");
	}
	front = (front+1)%MAX_QUEUE_SIZE;
	return queue[front];
}

Element peek() {
	if(is_empty()) {
		error("큐 공백상태");
	}
	return queue[(front+1)%MAX_QUEUE_SIZE];
}

int main() {
	init();
	int i;
	for(i=0; i<10; i++) {
		enqueue(i);
	}
	
	for(i=0; i<112; i++) {
		printf("dequeue : %d\n", dequeue());
	}
}
```

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>성결대학교 컴퓨터 공학과 박미옥 교수님 (2021)</li>
    <li>최영규, 『두근두근 자료구조』</li>
  </ul>
  본 게시글은 위 강의 및 교재를 기반으로 정리한 글입니다.
</div>