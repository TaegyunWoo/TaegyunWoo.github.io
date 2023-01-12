---
category: Interview
tags: [Interview]
title: "[Interview 대비] 연결 리스트"
date:   2023-01-12 13:00:00 
lastmod : 2022-01-12 13:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 연결리스트 (Linked Lists)

## 연결 리스트의 특징

![Untitled](/assets/img/2023-01-12-Interview_Linked_Lists/Untitled.png)

- **불연속적인 메모리 위치에 저장**되는 **선형** 데이터 구조
    - **포인터**를 사용해서 연결된다.
    
    > 배열 = 연속적인 메모리 위치에 저장된다.
    > 
- 각 노드마다 **`다음 노드의 주소를 갖는 포인터`** 가 존재한다.
    - 양방향 리스트라면 **`다음 노드의 주소를 갖는 포인터`** 와 **`이전 노드의 주소를 갖는 포인터`** 를 모두 가지고 있다.

## 연결 리스트를 사용하는 이유

### 배열의 한계

- **배열의 크기가 고정**되어 있어, 미리 요소의 수에 대해 할당을 받아야 한다.
- **새로운 요소를 삽입**하는 것은 **비용이 많이 든다.**
    - 삽입할 위치 이후의 모든 요소를 한칸씩 뒤로 밀고 삽입해야한다.
    
    ![Untitled](/assets/img/2023-01-12-Interview_Linked_Lists/Untitled%201.png)
    
    - 시간 복잡도 : **O(n)**
- **중간 요소를 제거**하는 것 역시 **비용이 많이 든다.**
    
    ![Untitled](/assets/img/2023-01-12-Interview_Linked_Lists/Untitled%202.png)
    
    - 시간 복잡도 : **O(n)**

### 연결 리스트의 장점

- **동적으로 크기를 조절**할 수 있다.
- **삽입이 용이**하다.
    - 새로운 노드를 만들고, 삽입할 위치에 따라 포인터를 설정하기만 하면 된다.
    
    ![Untitled](/assets/img/2023-01-12-Interview_Linked_Lists/Untitled%203.png)
    
    - 시간 복잡도 : O(1)
- **삭제가 용이**하다.
    - 삭제할 위치에 따라 포인터를 설정하기만 하면 된다.
    
    ![Untitled](/assets/img/2023-01-12-Interview_Linked_Lists/Untitled%204.png)
    
    - 시간 복잡도 : O(1)

### 연결 리스트의 단점

- 임의의 노드에 **바로 접근할 수 없다.**
    - 즉 첫번째 노드부터 **순차적으로 요소에 접근**해야 한다.
- 포인터가 필요하므로, **추가적인 메모리 공간이 필요**하다.

## 관련 포스팅

[https://taegyunwoo.github.io/datastructure/DATASTRUCTURE_List](https://taegyunwoo.github.io/datastructure/DATASTRUCTURE_List)