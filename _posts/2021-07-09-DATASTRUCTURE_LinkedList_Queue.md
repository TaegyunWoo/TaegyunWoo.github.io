---
category: DataStructure
tags: [DataStructure]
title: "[데이터구조] 연결리스트-큐"
date:   2021-07-09 12:00:00 
lastmod : 2021-07-09 12:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 연결리스트

## 연결리스트 - 큐

### 구성요소

![연결리스트 큐 구성요소](/assets/img/2021-07-09-DATASTRUCTURE_LinkedList_Queue/Untitled_49.png)

<br>

### 노드 구조

```c
typedef Element int; //저장할 데이터의 타입 정의

typedef struct LinkedNode {
	Element data; //저장할 데이터
	struct LinkedNode *link; //다음 노드의 주소
} Node;

Node *front = NULL; //front가 가르킬 노드의 주소
Node *rear = NULL; //rear가 가르킬 노드의 주소
```

<br>

### 공백상태

- front == rear == NULL

```c
int is_empty() {
	return top==NULL&&rear==NULL;
}
```

<br>

### 포화상태

- 계속해서 추가할 수 있으므로 의미가 없다.

<br>

### 큐 초기화

- front == NULL
- rear == NULL

<br>

### enqueue 연산

- **큐가 비어있는 상태에서의 enqueue** 와  
**큐가 비어있지 않는 상태에서의 enqueue** 의  
작동 방식이 다르다.

- <큐가 비어있는 상태에서 enqueue할 때>
  - front와 rear 모두 새로운 노드를 가르킨다.

  ![큐가 비어있는 상태에서 enqueue](/assets/img/2021-07-09-DATASTRUCTURE_LinkedList_Queue/Untitled_50.png)

<br>

- <큐가 비어있지 않은 상태에서 enqueue할 때>
  1. **'새로운 노드의 link'** 는 'rear가 가르키는 노드'의 주소값을 가르킨다.
  2. **'rear가 가리키는 노드의 link'** 는 새로운 노드의 주소값을 가르킨다.

  ![큐가 비어있는 상태에서 enqueue](/assets/img/2021-07-09-DATASTRUCTURE_LinkedList_Queue/Untitled_51.png)

```c
void enqueue(Element e) {
	Node *newNode = (Node*)malloc(sizeof(Node));
	p->data = e;
	p->link = NULL;

	if(is_empty()) {
		//빈 큐일때
		front = rear = newNode;
	} else {
		//비어있지 않은 큐일때
		rear->link = newNode; //(1)
		rear = newNode; //(2)
	}
}
```

> 가장 먼저, 추가할 데이터를 가지는 노드 객체를 생성해야함 (동적 할당으로)

> 가장 마지막에 있는 노드의 link는 NULL

<br>

### dequeue 연산

1. 공백검사
2. **'새로운 포인터 변수'** 가 '삭제할 노드'를 가르킴 (front의 값 가져옴)
be) return할 노드 기억
3. **'front'** 가 'front가 가르키는 노드'를 가리키도록 함
4. 만약 하나 남은 노드를 제거했을 때, rear도 NULL로 설정함
be) 공백상태 만들기
5. **'기억해둔 노드'** 의 data를 저장
6. **'기억해둔 노드'** 를 메모리에서 해제
7. **'기억해둔 data'** 를 반환

```c
Element pop(Element inputData) {
	//return할 노드의 주소값을 담을 포인터변수
	Node *returnNode = NULL;
	//반환할 데이터
	Element returnData

	if(is_empty()) {
		error("스택 공백 에러");
	}

	//기존에 front가 가르키던 노드 주소값 저장
	returnNode = front;

	//front가 가르킬 '다음 노드'의 주소값
	front = front->link;

	//return 노드의 data 저장
	Element returnData = returnNode->data;

	//삭제할 노드를 메모리에서 해제
	free(returnNode);

	//pop한 노드의 데이터 반환
	return returnData;
}
```

<br>

### peek 연산

1. 공백상태 검사
2. front가 가르키는 노드의 data멤버 반환

```c
Element peek() {
	//공백검사
	if(is_empty()) {
		error('공백오류');
	}

	return front->data;
}
```

<br>

### destroy_queue 연산

- 큐 사용이 끝났다면, 큐를 메모리에서 해제해야함

- 공백상태가 아닐 때까지 dequeue 반복

```c
void destroy_stack() {
	while(is_empty==0) {
		dequeue();
	}
}
```

<br>

### size 연산

- 모든 노드들을 따라가면서 연결된 횟수 검사

```c
int size() {
	int size = 0;
	Node *node;

	//마지막 노드의 link값이 NULL이므로, NULL까지 찾는다.
	for(node=front; node!=NULL; node=node->link) {
		size++;
	}
	return size;
}
```

<br>

### 종합예시

```c
#include <stdio.h>
#include <stdlib.h>

typedef int Element;
typedef struct LinkedNode {
	Element data;
	struct LinkedNode *link;
} Node;
Node *front;
Node *rear;

void init_queue() {
	front = NULL;
	rear = NULL;
}

void error(char *msg) {
	printf("%s", msg);
	exit(1);
}

int is_empty() {
	return front==NULL&&rear==NULL;
}

void enqueue(Element e) {
	Node *newNode = (Node*)malloc(sizeof(Node));
	newNode->data = e;
	newNode->link = NULL;
	if(is_empty()) {
		front = newNode;
		rear = newNode;
	} else {
		rear->link = newNode;
		rear = newNode;
	}
}

Element dequeue() {
	Node *node;
	Element returnData;
	if(is_empty()) {
		error("공백상태 에러");
	}
	
	node = front;
	returnData = node->data;
	front = front->link;
	if(front==NULL) {
		rear = NULL;
	}
	free(node);
	return returnData;
}

int size() {
	Node *node;
	int count = 0;
	for(node=front; node!=NULL; node=node->link) {
		count++;
	}
	return count;
}

Element peek() {
	Node *node;
	if(is_empty()) {
		error("큐 공백에러");
	}
	node = front;
	return front->data;
}

void main() {
	init_queue();
	int i;
	for(i=0; i<10; i++) {
		enqueue(i);
	}
	
	printf("%d", dequeue());
	printf("%d", dequeue());
	printf("%d", dequeue());
	printf("%d", dequeue());
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