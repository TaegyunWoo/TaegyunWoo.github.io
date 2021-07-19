---
category: DataStructure
tags: [데이터구조, 자료구조, 이중연결리스트]
title: "[데이터구조] 이중연결리스트와 헤드노드"
date:   2021-07-14 13:00:00 
lastmod : 2021-07-14 15:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Double Linked List - with "Head Node"

## 개요

### 특징

- 링크 필드가 양방향(총 2개)를 표현함
- 각 노드의 선행노드를 쉽게 알 수 있음

> 실제로는 이중 연결리스트 + 원형 연결 리스트를 혼합한 형태가 많이 사용됨

<br>

### 노드 구조

![이중연결리스트의 노드구조](/assets/img/2021-07-14-DATASTRUCTURE_DoubleLinkedList/Untitled_8.png)

<br>

```c
typedef int Element;
typedef struct DblLinkedNode {
	Element data;
	struct DblLinkedNode *prev;
	struct DblLinkedNode *next;
} Node;
```

<br>

### 단점

- 공간을 많이 차지
- 복잡한 코드

<br>

### 성립관계

- **p == p->next->prev == p->prev->next**

<br>

**< 공백리스트에서도 성립함 (헤드노드사용시) >**  
![이중연결리스트의 삽입연산1](/assets/img/2021-07-14-DATASTRUCTURE_DoubleLinkedList/Untitled_9.png)

- 헤드노드의 prev가 자기의 next를 가르키고
- 헤드노드의 next가 자기의 prev를 가르키기 때문에

<br>

### 삽입 연산 ( 이중 연결 리스트의 insert_next 연산 )

![이중연결리스트의 삽입연산2](/assets/img/2021-07-14-DATASTRUCTURE_DoubleLinkedList/Untitled_10.png)

<br>

```c
void insert_next(Node *before, Node *n) {
	if( n != NULL ) {
		n->prev = before;       // (1)
		n->next = before->next; // (2) (선행노드가 마지막 노드라면 => NULL 들어감 )
		
		if( before->next != NULL ) { //선행노드가 마지막 노드가 아닐때
			before->next->prev = n; // (3)
		}
		before->next = n;       // (4)
	}
}
```

<br>

### 삭제 연산 ( 이중 연결 리스트의 remove_next 연산 )

삭제 함수 이름이 remove_*next*()일 필요가 없음
 ⇒ 선행노드에 대한 정보가 없어도 삭제할 수 있기 때문에
 ⇒ remove_*next* → remove_*curr*

![이중연결리스트의 삭제연산](/assets/img/2021-07-14-DATASTRUCTURE_DoubleLinkedList/Untitled_8.png)

```c
void remove_curr( Node *n ) {
	if(n->prev != NULL) { //삭제하려는 노드가 첫번째 노드가 아닐때
		n->next->prev = n->prev;
	}
	if(n->next != NULL) { //삭제하려는 노드가 마지막 노드가 아닐때
		n->prev->next = n->next;
	}
}
```

<br><br>

## 이중 연결리스트 + 헤드노드

### 초기변수 및 구조체

```c
typedef int Element;

//노드
typedef struct LinkedNode {
	Element data; //데이터필드
	struct LinkedNode *prev; //선행노드를 가르키는 링크 필드
	struct LinkedNode *next; //후행노드를 가르키는 링크 필드
} Node;

Node org; //헤드노드 (정적으로 생성함)
```

<br>

### error(char *msg)

```c
void error(char *msg) {
	printf("%s", msg);
	exit(1);
}
```

<br>

### init_list()

```c
void init_list() {
	org.next = NULL;
	//원형 리스트가 아니기 때문에, prev 초기화 필요X (헤드노드의 prev는 항상 NULL임)
}
```

<br>

### get_head()

첫 번째 노드 반환

```c
Node* get_head() {
	return org.next;
}
```

<br>

### is_empty()

```c
int is_empty() {
	return get_head()==NULL;
}
```

<br>

### is_full()

- 포화상태는 존재하지 않기 때문에 의미가 없다.

<br>

### get_entry(int index)

```c
Node* get_entry(int index) {
	int i;
	Node *n = &org;
	for(i=-1; i<index; i++, n=n->next) {
		if(n==NULL) { //끝까지 탐색했을때 (찾으려는 index가 존재하지 않을때)
			break;
		}
	}
	return n;
}
```

<br>

### replace(int index, Element e)

```c
void replace(int index, Element e) {
	Node *n = get_entry(index);
	if(n != NULL) {
		n->data = e;
	}
}
```

<br>

### size()

```c
int size() {
	int counter = 0;
	Node *n = NULL;
	for(n=get_head(); n!=NULL; n=n->next) {
		counter++;
	}
	return counter;
}
```

<br>

### clear_list()

```c
void clear_list() {
	while(!is_empty()) {
		delete();
	}
}
```

<br>

### find( Element e )

```c
Element find( Element e ) {
	Node *n;
	for(n=get_head(); n!=NULL; n=n->next) {
		if(n->data == e) {
			return n;
		}
	}
	return NULL; //탐색 실패시
}
```

<br>

### insert_next( Node *before, Node *node ) & insert( int pos, Element e )

```c
void insert_next(Node *before, Node *node) {
	if( node != NULL ) {
		node->prev = before;
		node->next = before->next;
		if( before->next != NULL ) {
			before->next->prev = node;
		}
		before->next = node;
	}
}

void insert( int pos, Element e ) {
	Node *new_node, *before;
	before = get_entry(pos-1);
	if( before != NULL ) { //선행노드가 존재할 때
		new_node = (Node*)malloc(sizeof(Node));
		new_node->data = e;
		new_node->prev = NULL;
		new_node->next = NULL;
		insert_next(before, new_node);
	}
}
```

<br>

### remove_curr(Node *before) & delete(int pos)

```c
void remove_curr( Node *n ) {
	if(n->prev != NULL) { //삭제하려는 노드가 첫번째 노드가 아닐때
		n->next->prev = n->prev;
	}
	if(n->next != NULL) { //삭제하려는 노드가 마지막 노드가 아닐때
		n->prev->next = n->next;
	}
}

void delete(int pos) {
	Node *n = get_entry(pos);
	if(n != NULL) {
		remove_curr(n);
    free(n);
	}
}
```

<br>

### 전체 코드

```c
#include <stdio.h>

typedef int Element;

//노드
typedef struct LinkedNode {
	Element data; //데이터필드
	struct LinkedNode *prev; //선행노드를 가르키는 링크 필드
	struct LinkedNode *next; //후행노드를 가르키는 링크 필드
} Node;

Node org; //헤드노드 (정적으로 생성함)

void error(char *msg) {
	printf("%s", msg);
	exit(1);
}

void init_list() {
	org.next = NULL;
	//원형 리스트가 아니기 때문에, prev 초기화 필요X (헤드노드의 prev는 항상 NULL임)
}

Node* get_head() {
	return org.next;
}

int is_empty() {
	return get_head()==NULL;
}

Node* get_entry(int index) {
	int i;
	Node *n = &org;
	for(i=-1; i<index; i++, n=n->next) {
		if(n==NULL) { //끝까지 탐색했을때 (찾으려는 index가 존재하지 않을때)
			break;
		}
	}
	return n;
}

void replace(int index, Element e) {
	Node *n = get_entry(index);
	if(n != NULL) {
		n->data = e;
	}
}

int size() {
	int counter = 0;
	Node *n = NULL;
	for(n=get_head(); n!=NULL; n=n->next) {
		counter++;
	}
	return counter;
}

void clear_list() {
	while(!is_empty()) {
		delete();
	}
}

Element find( Element e ) {
	Node *n;
	for(n=get_head(); n!=NULL; n=n->next) {
		if(n->data == e) {
			return n;
		}
	}
	return NULL; //탐색 실패시
}

void insert_next(Node *before, Node *node) {
	if( node != NULL ) {
		node->prev = before;
		node->next = before->next;
		if( before->next != NULL ) {
			before->next->prev = node;
		}
		before->next = node;
	}
}

void insert( int pos, Element e ) {
	Node *new_node, *before;
	before = get_entry(pos-1);
	if( before != NULL ) { //선행노드가 존재할 때
		new_node = (Node*)malloc(sizeof(Node));
		new_node->data = e;
		new_node->prev = NULL;
		new_node->next = NULL;
		insert_next(before, new_node);
	}
}

void remove_curr( Node *n ) {
	if(n->prev != NULL) { //삭제하려는 노드가 첫번째 노드가 아닐때
		n->next->prev = n->prev;
	}
	if(n->next != NULL) { //삭제하려는 노드가 마지막 노드가 아닐때
		n->prev->next = n->next;
	}
}

void delete(int pos) {
	Node *n = get_entry(pos);
	if(n != NULL) {
		remove_curr(n);
    	free(n);
	} else {
		printf("delete ERR\n");
	}
}

int main() {
	init_list();
	int i;
	
	for(i=0; i<10; i++) {
		insert(i, i);
	}
	
	delete(4);
	delete(2);
	delete(5);
	
	printf("%d\n", size());
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