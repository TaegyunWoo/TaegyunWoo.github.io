---
category: Interview
tags: [Interview]
title: "[Interview Study] Hash"
date:   2023-01-19 19:00:00 
lastmod : 2022-01-19 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 해시(Hash)

![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled.png)

## 해시란?

- 데이터를 효율적으로 관리하기 위해, **임의의 길이를 갖는 데이터를 고정된 길이의 데이터로 매핑하는 것** 을 말한다.
- **해시 함수**를 통해, 데이터 값을 해시 값으로 매핑한다.
    
    > 해시 함수는 보통 O(1) 시간이 걸리도록 구현한다.
    > 

## 해시 충돌 문제

- **서로 다른 데이터가 같은 해시 값으로 충돌** 하게 되는 문제가 발생할 수  있다.
- 이것을 **‘collision’ 현상** 이라고 한다.

## 해시 테이블

### 특징

- `(Key, Value)` 구조로 데이터를 저장하는 자료구조 중 하나이다.
- 해시값을 index로 사용해, 데이터를 빠르게 찾아낼 수 있다.
- 해시테이블의 평균 시간복잡도는 O(1)이다.

### 해시 충돌 문제 해결

1. **체이닝**
    - **연결리스트** 로 노드를 계속 추가해나가는 방식
    - 제한 없이 계속 연결할 수 있다.
    
    ![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled%201.png)
    
2. **Open Addressing**
    - 해시 함수로 얻은 주소가 아닌 **다른 주소에 데이터를 저장할 수 있도록 허용** 하는 방식
    - 즉 **이미 해당 주소에 값이 저장되어 있으면 다음 주소에 저장** 한다.
    
    ![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled%202.png)
    
    - Open Addressing 방식에는 **‘선형 탐사’, ‘제곱 탐사’, ‘이중 해싱’ 방법이 존재** 한다.
3. **선형 탐사**
    - Open Addressing 기법 중 하나로, **정해진 고정 폭으로 옮겨 해시값의 중복을 피하는 기법** 이다.
    - 예를 들어, 고정폭이 2이고 52번 인덱스에서 충돌이 발생하면 54번 인덱스에 저장한다.
    - **데이터들이 메모리의 특정 구간에만 몰려있는 현상**이 발생하게 된다. 그리고  특정 구간에 몰려 있을수록 충돌 가능성이 커지고, 규모가 커진다. (**Primary Clustering**) → 이 현상 때문에, 성능이 감소하게 된다.
    
    ![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled%203.png)
    
4. **제곱 탐사**
    - Open Addressing 기법 중 하나로, **정해진 고정 폭을 제곱수로 옮겨 해시값의 중복을 피하는 기법** 이다.
    - 예를 들어, `3` 번 인덱스에서 충돌이 발생하면, `3 + 1^2` 번 인덱스를 확인한다. 그리고 해당 인덱스에도 값이 이미 저장되어 있다면, `3 + 2^2` 번 인덱스를 확인하며 빈공간을 찾을때 까지 반복한다.
    
    ![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled%204.png)
    
    - **Primary Clustering 문제를 최소화** 할 수 있는 방법 중 하나이다.
    - 하지만 **여러 개의 원소가 동일한 초기 해시 함수값을 갖는 현상** 이 발생할 가능성이 크다. (**Secondary Clustering**)
        
        > Primary Clustering 문제보다는 덜 심각하다.

위와 같은 방식들로 해시 충돌 문제를 해결한다고 해도, 해시테이블의 데이터 조회 시간복잡도는 증가할 수 밖에 없다.

### 선형 탐사에서 원소 삭제 시 주의사항

만약 선형 탐사 기법을 사용해 해시 테이블을 구현한다면, 원소 삭제에 주의해야 한다.

![Untitled](/assets/img/2023-01-20-Interview_Hashing/Untitled%205.png)