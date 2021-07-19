---
category: DataStructure
tags: [데이터구조, 자료구조, 연결리스트, 스택]
title: "[데이터구조] 연결리스트-스택"
date:   2021-07-09 12:00:00 
lastmod : 2021-07-09 12:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 연결리스트

## 개요

### 특징

- **논리적순서 ≠ 물리적순서**
> 연결리스트에 'a, b, c' 을 저장이 했을 때, 메모리 주소는 '0x1, 0x53, 0x31'처럼 비순차적으로 저장된다.

### 장점

- 연속 메모리 공간 필요 X
- 삽입, 삭제 용이
- 크기 제한X

### 단점

- 오류 발생 쉬움
- 구현 어려움

### 구조

- 노드 구성요소
  - 데이터 필드
  - 링크 필드
- 헤드 포인터
  - 첫 번째 노드의 주소를 저장하는 포인터 변수
  - 헤드포인터 == top

![연결리스트 구조](/assets/img/2021-07-09-DATASTRUCTURE_LinkedList_Stack/Untitled_47.png)

<br/><br/><br/>

## 연결리스트로 구현한 스택

### 구성요소

![연결리스트 구성요소](/assets/img/2021-07-09-DATASTRUCTURE_LinkedList_Stack/Untitled_48.png)

### 노드 구조

```c
typedef Element int; // 데이터 타입 정의

typedef struct LinkedNode {
	Element data; //저장할 데이터
	struct LinkedNode *link; //다음 노드의 주소값
} Node;

Node *top = NULL; //최상단 노드의 주소값
```

<br>

> `typedef` 로 선언한 구조체의 정확한 이름으로 자체 참조 구조체 만들어야함

```c
typedef struct AA {
    struct a *link; // ⇒ 불가
} a;
```

<br><br>

### 공백상태

- 헤드 포인터 top이 NULL값을 갖는 경우

```c
int is_empty() {
	return top==NULL;
}
```

<br>

### 포화상태

- 계속해서 추가할 수 있기 때문에 의미없다.

<br>

### 스택 초기화

`top == NULL`

<br>

### push 연산

1. '삽입할 노드의 링크 필드'가 후행노드를 가리키도록 함
2. '헤더 포인터 top'이 '삽입할 노드'를 가리키도록 함

```c
void push(Element inputData) {
	//추가할 노드 생성
	Node *inputNodeAddr = (Node*)malloc(sizeof(Node));

	//추가할 데이터를 새 노드의 data에 저장
	inputNodeAddr->data = inputData;

	//기존에 top이 가르키던 노드의 주소값을 새 노드의 link에 저장
	inputNodeAddr->link = top;

	//top이 가르키는 주소값을 새 노드의 주소값으로 변경
	top = inputNodeAddr;
}
```

- 가장 먼저, 추가할 데이터를 가지는 노드 객체를 생성해야함 (동적 할당으로)

- 가장 마지막에 있는 노드의 link는 NULL값을 갖는다.

<br>

### pop 연산

1. 공백검사
2. '새로운 포인터 변수'가 '삭제할 노드'를 가르킴 (top의 값 가져옴)
be) return할 노드 기억
3. '헤더 포인터 top'이 '삭제할 노드가 가르키는 노드'를 가리키도록 함
4. '기억해둔 노드'의 data를 저장
5. '기억해둔 노드'를 메모리에서 해제
6. '기억해둔 data'를 반환

```c
Element pop(Element inputData) {
	//return할 노드의 주소값을 담을 포인터변수
	Node *returnNode = NULL;
	//반환할 데이터
	Element returnData

	if(is_empty()) {
		error("스택 공백 에러");
	}

	//기존에 top이 가르키던 노드 주소값 저장
	returnNode = top;

	//top이 가르킬 '다음 노드'의 주소값
	top = returnNode->link;

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
2. top이 가르키는 노드의 data멤버 반환

```c
Element peek() {
	//공백검사
	if(is_empty()) {
		error('공백오류');
	}

	return top->data;
}
```

<br>

### destroy_stack 연산

- 스택 사용이 끝났다면, 스택을 메모리에서 해제해야함

1. 공백상태가 아닐 때까지 pop 반복

```c
void destroy_stack() {
	while(is_empty==0) {
		pop();
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
	for(node=top; node!=NULL; node=node->link) {
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

Node *top = NULL;

void init_stack() {
	top = NULL;
}

void error(char *msg) {
	printf("%s", msg);
	exit(1);
}

int is_empty() {
	return top==NULL;
}

void push(Element e) {
	Node *node = (Node*)malloc(sizeof(Node));
	node->data = e;
	node->link = top;
	top = node;
}

Element pop() {
	Node *node;
	Element data;
	if(is_empty()) {
		error("스택공백에러");
	}
	
	node = top;
	data = node->data;
	top = node->link;
	
	free(node);
	return data;
}

Element peek() {
	if(is_empty) {
		error("공백 스택 에러");
	}
	return top->data;
}

int size() {
	int count=0;
	Node *node;
	for(node=top; node->link!=NULL; node=node->link) {
		count++;
	}
	return count;
}

void destroy_stack() {
	Node *node = top;
	while(top->link != NULL) {
		pop();
	}
}

int main() {
	init_stack();

	int i;
	for(i=0; i<10; i++){
		push(i);
	}
	
	printf("%d\n", pop());
	printf("%d\n", pop());
	printf("%d\n", pop());
	printf("%d\n", pop());
	printf("%d\n", pop());
	printf("size=%d", size());
	
	destroy_stack();
	
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