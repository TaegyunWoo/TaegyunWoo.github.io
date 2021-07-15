---
category: DataStructure
tags: [데이터구조, 자료구조, 이진트리, 트리순회]
title: "[데이터구조] 이진트리 순회"
date:   2021-07-15 15:00:00 
lastmod : 2021-07-15 15:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 트리

## 이진 트리의 순회

### 순회란?

- 트리의 노드들을 한 번씩 방문하는 것
- 가장 기본적인 연산
- 순회 방법
  - 전위 순회 (VLR)
  - 중위 순회 (LVR)
  - 후위 순회 (LRV)
  - V, L, R 이란?
    ![V, L, R](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_37.png)

<br>

### 전위 순회

- 루트 → 왼쪽 자식 → 오른쪽 자식
- VLR

![VLR](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_38.png)

- 의사코드

    ![VLR의사코드](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_39.png)

    `preorder(LEFT(x))` : 왼쪽부터 쭉 파고들어간 뒤, 종단노드를 만나면 재귀호출이 종료됨  
    ( 이후엔 `preorder(RIGHT(x))` 호출할 차례 )

- 구현 코드

    ```c
    void preorder(TNode *n) {
    	if(n!=NULL) {
    		printf("[%c] ", n->data);
    		preorder(n->left);
    		preorder(n->right);
    	}
    }
    ```

<br>

### 중위 순회

- 왼쪽 자식 → 루트 → 오른쪽 자식
- LVR

![LVR](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_40.png)

- 의사코드

    ![LVR의사코드](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_41.png)

- 구현 코드

    ```c
    void inorder(TNode *n) {
    	if(n!=NULL) {
    		preorder(n->left);
    		printf("[%c] ", n->data);
    		preorder(n->right);
    	}
    }
    ```

<br>

### 후위 순회

- 왼쪽 자식 → 오른쪽 자식 → 루트
- LRV

![LRV](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_42.png)

- 의사코드

    ![LRV 의사코드](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_43.png)

- 구현 코드

    ```c
    void postorder(TNode *n) {
    	if(n!=NULL) {
    		preorder(n->left);
    		preorder(n->right);
    		printf("[%c] ", n->data);
    	}
    }
    ```

<br>

### 레벨 순회

- 레벨1의 왼쪽노드부터 하나씩 읽으며, 레벨 내려간다.
- 큐를 함께 이용한다.

> 레벨 순회는 전위, 중위, 후위 순회와는 좀 다른 분류이다.

![레벨순회](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_44.png)

<br>

- 동작 원리

  - 트리 구조를 따라, 순회하면서 enqueue와 dequeue를 반복한다.
  ![레벨순회 동작 원리](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_45.png)

<br>

- 의사코드
    ![레벨순회 의사코드](/assets/img/2021-07-14-DATASTRUCTURE_BinaryTree_Traversal/Untitled_46.png)

    > 큐도 구현해야함

<br>

- 구현 코드

    ```c
    //큐 구현부 필요
    //트리 구현부 필요

    void levelorder(TNode *root) {
    	TNode *n;
    	if(root == NULL) { //트리가 공백상태라면 함수 중단
    		return;
    	}
    	
    	init_queue();
    	enqueue(root);
    	while( !isEmpty() ) { //큐가 비어있다면 중단
    		n = dequeue();
    		if( n != NULL ) {
    			printf("[%c] ", n->data); //dequeue된 노드의 데이터값
    			enqueue(n->left);
    			enqueue(n->right);
    		}
    	}
    }
    ```

<br>

### 순회 방법의 선택 요령

- 단순히 모든 노드를 읽는 목적일 때
  - 어떤 순회방법을 사용해도 됨
- 자식 노드를 처리한 다음, 부모 노드를 처리해야 하는 문제
  - 후위 순회 사용
  - ex) 폴더 용량 계산
          ⇒ 하위 폴더의 용량부터 계산해야 함
- 부모 노드를 처리한 다음, 자식 노드를 처리해야 하는 문제
  - 전위 순회 사용
  - ex) 트리 레벨 계산
          ⇒ 루트 노드부터 세야하기 때문에
