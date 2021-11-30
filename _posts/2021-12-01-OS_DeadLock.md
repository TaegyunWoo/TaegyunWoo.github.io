---
category: OS
tags: [OS]
title: "[OS] 데드락"
date:   2021-12-01 00:30:00 
lastmod : 2021-12-01 00:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 데드락

## 개요

### 데드락이란?

- 데드락이란, 둘 이상의 프로세스가 각각 다른 프로세스가 점유한 자원을 요구하면서, 모두 작업 수행을 할 수 없이 대기 상태로 놓이는 상태를 말한다.
- 데드락 == 교착상태

<br/>

### 데드락 예시

1. 시스템은 2개의 Resource를 보유하고 있다.
2. 프로세스 P1과 P2는 각각 한 개의 Resource를 점유한 상태이다.
3. 각각의 프로세스가 데이터 처리를 위해, 또 다른 Resource를 요구하게 된다.
4. 프로세스 P1과 P2는 교착상태에 빠지게 된다.

![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2071.png)

<br/>

### 시스템 모델

- 시스템은 유한한 수의 자원들로 구성된다.
    - 자원 종류: CPU cycles, memory space, I/O devices
- 각 자원 타입(![math](https://latex.codecogs.com/svg.image?R_i))은 ![math](https://latex.codecogs.com/svg.image?W_i) 개의 인스턴스를 갖는다.
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2072.png)
    
- 프로세스는 아래와 같은 순서로만 자원을 사용할 수 있다.
    1. **Request (요청)**
    2. **Use (사용)**
    3. **Release (방출)**

<br/><br/>

## 데드락 특징

### 데드락 발생 조건

아래의 4가지 조건들이 **동시**에 일어나면 데드락(교착상태)이 발생한다.

- **Mutual exclusion (상호배제)**
    - 오직 한개의 프로세스가 자원을 점유할 수 있다.
    - 다른 프로세스가 그 자원을 사용하려 할 경우, 그 프로세스는 자원의 사용이 끝날 때까지 지연되어야 한다.
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2073.png)
    
- **Hold and wait (점유대기)**
    - 프로세스가 자원을 최소 한 개 점유하고, 다른 프로세스가 점유한 다른 자원을 얻기 위해서 기다린다.
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2074.png)
    
- **No preemption (비선점)**
    - 자원을 선점할 수 없다. (아직 사용 중인 자원을 뺏기지 않는다.)
    - 즉, 프로세스가 작업 종료 후에 자발적으로 자원을 방출한다.
- **Circular wait (순환 대기)**
    - 대기하고 있는 프로세스 집합 {![math](https://latex.codecogs.com/svg.image?P_0,P_1,...,P_n)} 이 있다고 할 때
    - ![math](https://latex.codecogs.com/svg.image?P_0) 는 ![math](https://latex.codecogs.com/svg.image?P_1)이 점유하고 있는 자원을 기다린다.
    - ![math](https://latex.codecogs.com/svg.image?P_1) 는 ![math](https://latex.codecogs.com/svg.image?P_2)이 점유하고 있는 자원을 기다린다.
    - ![math](https://latex.codecogs.com/svg.image?P_{n-1}) 는 ![math](https://latex.codecogs.com/svg.image?P_n)이 점유하고 있는 자원을 기다린다.
    - ![math](https://latex.codecogs.com/svg.image?P_n) 는 ![math](https://latex.codecogs.com/svg.image?P_0)이 점유하고 있는 자원을 기다린다.
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2075.png)
    
<br/><br/>

## 자원 할당 그래프 (Resource-Allocation Graph: RAG)

### RAG란?

- 교착 상태를 정확하게 기술하는 그래프

<br/>

### RAG 구성요소

- **Vertex (정점, E)**
    - 프로세스
    - 자원
- **Edge (간선, E)**
    - **요청 간선** : ![math](https://latex.codecogs.com/svg.image?P_i)  → ![math](https://latex.codecogs.com/svg.image?R_j)
    - **할당 간선** : ![math](https://latex.codecogs.com/svg.image?R_j) → ![math](https://latex.codecogs.com/svg.image?P_i)
    - **예약 간선**
        
        > 예약 간선은 포스팅 후반에 다룬다.
        > 

<br/>

### RAG 요소 표현

- **프로세스**
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2076.png)
    
- **자원 타입 (4개 인스턴스를 갖음)**
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2077.png)
    
- **![math](https://latex.codecogs.com/svg.image?P_i) 가 ![math](https://latex.codecogs.com/svg.image?R_j) 의 인스턴스 요청**
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2078.png)
    
- **![math](https://latex.codecogs.com/svg.image?R_j) 의 인스턴스가 ![math](https://latex.codecogs.com/svg.image?P_i) 에 할당**
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2079.png)
    
<br/>

### RAG 예시

- P = {![math](https://latex.codecogs.com/svg.image?P_1,P_2,P_3)}
- R = {![math](https://latex.codecogs.com/svg.image?R_1,R_2,R_3,R_4)}
- E = {![math](https://latex.codecogs.com/svg.image?P_1->R_1,P_2->R_3,R_1->P_2,R_2->P_2,R_2->P_1,R_3->P3)}

![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2080.png)

<br/>

### 데드락과 RAG 예시 : 데드락 발생

- 데드락 찾기
    - 자원의 인스턴스가 하나밖에 없는 경우
        - 사이클 형성 시, 무조건 데드락이 발생한다.
    - 자원의 인스턴스가 여러개 있는 경우
        - 사이클 형성 시, 데드락 발생 가능성이 있다.
    - 즉 인스턴스 여유없이, 사이클 형성시 데드락이 발생한다.

![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2081.png)

<br/>

### 데드락과 RAG 예시 : 데드락 발생 X

- **사이클이 존재한다고 반드시 교착상태가 발생한다는 것은 아니다.**

![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2082.png)

<br/>

### 정리

- 자원 할당 그래프에서 사이클이 없다면, 시스템은 교착 상태가 아니다.
- 만약 사이클이 있는 경우
    - 자원 유현 당 하나의 인스턴스만 있다면 교착상태이다.
    - 자원 유형 당 여러 개의 인스턴스가 잇다면 교착 상태의 가능성이 있다.

<br/>

### 데드락을 관리하는 여러 방법들

- 시스템이 결코 교착상태가 되지 않도록 보장하기 위해, 교착상태를 예방하거나 회피하는 프로토콜을 사용하는 방법
- 시스템이 교착상태가 되도록 허용한다음, 회복시키는 방법
- 문제를 무시하고 교착상태가 시스템에서 결코 발생하지 않는 척 하는 방법
    
    > UNIX를 포함한 대부분의 OS에서 사용하는 방법이다.
    > 

<br/><br/>

## 데드락 핸들링: 데드락 예방 방법 (Deadlock Prevention)

### 데드락 예방이란?

- 자원 요청 방법을 제한하는 방식을 말한다.
- 데드락 발생 조건을 피하여 데드락을 예방하고자 한다.

<br/>

### 데드락 예방을 위한 조건

- **Mutual Exclusion (상호배제)을 부정하기**
    - 애초에 자원에 대해 동시 접근하는 것이 불가능하기 때문에, 상호배제를 부정하는 것은 불가능하다.
- **Hold and Wait (점유 대기)를 부정하기**
    - 점유 대기를 부정하기 위해선, 프로세스가 자원을 요청할 때마다 다른 자원을 보유하지 않도록 보장해야 한다.
    - 또는 자원을 전혀 소유하지 않은 프로세스만 자원을 요청하게 해야한다.
- **No Preemption (비선점)을 부정하기**
    - '어떤 자원을 보유하고 있는 프로세스'가 '즉시 할당할 수 없는 다른 자원'을 요청하면, ' 그 프로세스가 현재 보유하고 있는 모든 자원'을 방출하도록 한다.
    - 프로세스는 '자신이 요청하고 있는 새로운 자원'과 '이전에 점유했던 옛 자원'을 다시 획득할 수 있을 때만 다시 시작된다.
    - 예시
        - '프로세스 A가 요청한 자원'을 프로세스 A가 오래 사용해야한다면, **'프로세스 A가 가지고 있던 자원'을 다른 프로세스가 사용할 수 있도록 모든 자원을 방출한 후**, 요청자원을 할당한다.
- **Circular Wait (순환 대기)를 부정하기**
    - 모든 자원 타입에 전체 순서를 부여하고, 각 프로세스가 열거된 순서대로 오름차순으로 자원을 요청하도록 한다.

<br/>

### Mutex 와 Deadlock 예시

![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2083.png)

- 교착 상태가 발생한다.

<br/><br/>

## 데드락 핸들링: 데드락 회피 방법 (Deadlock Avoidance)

### 데드락 회피란?

- 시스템이 추가적인 사전 정보를 가용하도록(미리 알 수 있도록) 하여 데드락을 피하는 것을 말한다.

<br/>

### 데드락 회피를 위한 조건

- 각 프로세스가 필요한 각 자원의 최대 인스턴스를 선언한다.
- 교착상태회피 알고리즘은 자원할당 상태를 검사하여 순환대기 상태가 없도록 한다.
- 자원 할당 상태는 **'사용가능한 자원 수'** 와 **'할당된 자원 수'** 와 **'프로세스의 최대 요구 자원 수'** 에 의해 정의된다.

<br/>

### Safe State (안전 상태)

- 안전 상태란?
    - '<![math](https://latex.codecogs.com/svg.image?P_1,P_2,...,P_n)>' 가 시스템에 모든 프로세스로 구성되었다고 가정하자.
    - **이때, ![math](https://latex.codecogs.com/svg.image?P_i) 가 요청하는 모든 자원이 '현재 사용 가능한 자원'과 '모든 프로세스가 소유한 자원'으로 충족되는 상태를 말한다.**
    - 예시
        - 전체 자원: 10
        - 사용 중인 자원: 5
        - 이때 각 프로세스가 요청할 수 있는 자원 수는 5를 넘을 수 없다.
- 데드락을 회피하기 위해선, 프로세스가 자원을 요청했을 때 시스템이 그 자원을 할당한 후에도 시스템을 **안전 상태(safe state)** 로 만드는지 확인해야 한다.

- 정리
    - 시스템이 안전 상태인 경우 ⇒ 교착 상태가 발생하지 않는다.
    - 시스템이 안전하지 않은 상태일 경우 ⇒ 교착 상태가 발생할 가능성이 존재한다.
    - 회피란 ⇒ 시스템이 안전하지 않은 상태로 들어가지 않도록 하는 것이다.

<br/>

### 회피 알고리즘

- 단일 인스턴스 자원일 때
    - **RAG 사용** 하여 데드락 회피
- 다중 인스턴스 자원일 때
    - **은행원 알고리즘을 사용** 하여 데드락 회피

<br/>

### 회피 알고리즘: RAG 알고리즘

- RAG의 간선 종류
    - 요청 간선
    - 할당 간선
    - 예약 간선
- **예약 간선**
    - 예약 간선은 프로세스 ![math](https://latex.codecogs.com/svg.image?P_i) 가 미래에 자원 ![math](https://latex.codecogs.com/svg.image?R_j)를 용청하는 의미를 갖는다.
    - 예약 간선은 점선으로 표시한다.
- **프로세스가 자원을 요청할 때, 예약 간선이 요청 간선으로 변환된다.**
- **자원이 프로세스에 할당되면, 요청 간선이 할당 간선으로 변환된다.**
- **자원이 프로세스에 의해 방출되면, 할당 간선은 예약 간선으로 재전환된다.**
    - 시스템에서 자원은 반드시 미리 예약되어야 한다.
- RAG 예시
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2084.png)
    

- 안전하지 않은 상태에서의 RAG 예시
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2085.png)
    

- **RAG 알고리즘**
    - 프로세스 ![math](https://latex.codecogs.com/svg.image?P_i) 가 자원 ![math](https://latex.codecogs.com/svg.image?R_j)를 요청한다고 가정하자.
    - **이때 요청 간선 ![math](https://latex.codecogs.com/svg.image?P_i) → ![math](https://latex.codecogs.com/svg.image?R_j) 를 할당 간선 ![math](https://latex.codecogs.com/svg.image?R_j) → ![math](https://latex.codecogs.com/svg.image?P_i) 로 변환해도, 자원 할당 그래프에 사이클을 형성하지 않을 경우에만 자원 요청을 허용한다.**
        - 이것이 RAG 알고리즘이다.

<br/>

### 회피 알고리즘: 은행원 알고리즘

- 은행원 알고리즘이란?
    - 자원의 다중 인스턴스가 있을 경우에 사용한다.
    - **각 프로세스는 사용하는 모든 자원의 최대 사용할 개수를 미리 선언해야 한다.**
        - 예시
        
        ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2086.png)
        
    - **프로세스가 자원을 요청할 때는 대기해야 할 수도 있다.**
    - **프로세스가 필요한 모든 리소스를 얻으면, 프로세스는 제한된 시간 내에 자원을 모두 반환해야 한다.**

- 관련 데이터
    - **n**
        - 프로세스의 수
    - **m**
        - 자원의 종류 수
    - **Available**
        - 각 종류별로 가용한 자원의 개수를 나타내는 벡터 (m 크기)
        - Available[j] = k 일 때, 현재 자원j를 k개 사용할 수 있다.
    - **Max**
        - 각 프로세스가 최대로 필요로 하는 자원의 개수를 나타내는 행렬 (n*m 크기)
        - Max[i,j] = k 일 때, 프로세스i가 자원j를 최대 k개까지 요청할 수 있다.
    - **Allocation**
        - 각 프로세스에게 현재 할당된 자원의 개수를 나타내는 행렬 (n*m 크기)
        - Allocation[i,j] = k 일 때, 프로세스i가 자원j를 최대 k개 사용 중이라는 것이다.
    - **Need**
        - 각 프로세스가 향후 요청할 수 있는 자원의 개수를 나타내는 행렬 (n*m 크기)
        - Need[i,j] = k 일 때, 프로세스i가 자원j를 k개 더 요청한다는 것이다.
    - 참고
        - **Need[i,j] = Max[i,j] - Allocation[i,j]**

<br/>

- **안전 상태 확인 알고리즘**
    - 안전 상태를 확인하는 알고리즘에 대해 먼저 알아보자.
    1. Work와 Finish는 크기가 m과 n인 vector이다.
        - `Work = Available //초기화`
        - `for (i=0,1,...,n-1) { Finish[i] = false } //초기화`
    2. 아래 두 조건을 만족하는 i 값을 찾는다.
        - `Finish[i] == false`
            - 프로세스i (![math](https://latex.codecogs.com/svg.image?P_i))가 끝나지 않은 상태를 의미한다.
        - `Need_(i) <= Work`
            - 프로세스i가 필요로 하는 자원이 가용 자원보다 작거나 같다는 의미이다.
        
        위 조건에 맞는 i 값을 찾을 수 없다면 step 4로 이동한다.
        
    3. 아래처럼 설정한다.
        - `Finish[i] = true`
            - 프로세스i를 끝날 수 있는 상태로 변경한다.
        - `Work = Work + Allocation_(i)`
            - 프로세스i에 할당된 자원을 가용한 자원으로 만든다.
        
        그리고 다시 step 2로 간다.
        
    4. 모든 i 값에 대해 `Finish[i] == true` 이면, 이 시스템은 안전 상태에 있다.

<br/>

- **은행원 알고리즘**
    - 전제
        - `Request_(i)` 는 프로세스 ![math](https://latex.codecogs.com/svg.image?P_i)의 요청 벡터이다.
        - `Request_(i) = k` 라면, 프로세스 ![math](https://latex.codecogs.com/svg.image?P_i) 가 자원 ![math](https://latex.codecogs.com/svg.image?R_j) 를 k개까지 요청한다는 것이다.
    1. 만일 `Request_(i) <= Need_(j)` 라면, step 2로 이동한다.  
    그렇지 않다면, 프로세스의 최대 요청 개수보다 더 많이 요청을 했으므로 에러로 처리한다.
    2. 만일 `Request_(i) <= Available` 이라면 step 3으로 이동한다.  
    그렇지 않다면, 요청에 대한 자원이 가용하지 않으므로 프로세스i는 대기해야 한다.
    3. 상태를 아래와 같이 바꿈으로써 자원을 프로세스i에게 할당해보려고 시도한다.
        - `Available = Available - Request_(i);`
        - `Allocation_(i) = Allocation_(i) + Request_(i);`
        - `Need_(i) = Need_(i) - Request_(i);`
        
        위 3가지 항목들이 안전 상태인지 판단하는 기준이 된다.
        
    4. 안전하다면, 자원을 프로세스i에게 할당한다.  
    안전하지 않다면, 자원할당은 원상태로 복귀하고 프로세스i는 요청이 만족될때까지 대기한다.
    - 알고리즘 워크플로우
        
        ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2087.png)
        
<br/>

### 은행원 알고리즘 예시

- 전제
    - 프로세스
        - P0 ~ P4까지 총 5개의 프로세스
    - 자원
        - 총 3 종류의 자원 (A, B, C)
        - A=10개_인스턴스
        - B=5개_인스턴스
        - C=7개_인스턴스
- 초기 상황
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2088.png)
    
    - Allocation 구하기
        
        ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2089.png)
        
    - Need 구하기
        - Need = Max - Allocation 이므로 아래가 성립한다.
        
        ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2090.png)
        

- **위 초기상태에서 <P1, P3, P4, P2, P0> 순서로 실행 시, 안전 상태가 될 수 있다.**
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2091.png)
    
    - 증명
        1. P1 실행 시
            
            ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2092.png)
            
        2. P3 실행 시
            
            ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2093.png)
            
        3. P4 실행 시
            
            ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2094.png)
            
        4. P2 실행 시
            
            ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2095.png)
            
        5. P0 실행 시
            
            ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2096.png)
            
        - **결론: 모든 프로세스가 Available 범위내에서 실행되었으므로 시스템은 처음부터 끝까지 안전 상태이다.**

<br/>

- **<P1, P3, P4, P2, P0> 순서 이외로, <P1,P3,P4,P0,P2> 순서로 실행해도 시스템은 계속 안전 상태이다.**
    
    > 이것은 스스로 해보자.
    > 

<br/>

- **Available 범위 내에서 프로세스를 실행할 수 있다고, 무조건 안전 상태인 것은 아니다.**
    - P0 에게 미리 자원을 (0, 2, 0)으로 할당시켜줄수 있을까?
        
        ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2097.png)
        
        - P0에게 (0, 2, 0)을 할당한다면 위와 같이 될 것이다.
        - 표면상, Available이 (2, 3, 0)인 상태에서 (0, 2, 0)를 사용하는 것은 문제가 되지 않는 것 같다.
        - **(0, 2, 0)를 할당하고 난 뒤, Available 값이 (2, 1, 0)으로 변화한다. 이 Available 값으로 실행을 할 수 있는 프로세스가 존재하지 않는다.**
        - **따라서 (0, 2, 0)를 요청하는 것이 Available 범위를 벗어나지 않는다고 해도, 들어줄 수 없다.**

<br/><br/>

## 데드락 핸들링: 데드락 감지 방법 (Deadlock Detection)

### 대기 그래프

- 대기 그래프란
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2098.png)
    

- 대기 그래프 예시
    
    ![Untitled](/assets/img/2021-12-01-OS_DeadLock/Untitled%2099.png)
    
<br/>

### 데드락 탐지 오버헤드

- 교착상태가 자주 일어난다면, 데드락 탐지 알고리즘도 자주 돌려야 한다.
    - **이때 오버헤드가 커진다.**
- 데드락 탐지 오버헤드 완화 대안
    - 한 시간에 한번만 탐지한다.
    - CPU 이용률이 40%일 때만 탐지한다.
    - 등...

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>성결대학교 컴퓨터 공학과 강영명 교수님 (2021)</li>
    <li>Siberschatz et. al., 『Operating System Concepts 10th Ed.』</li>
  </ul>
  본 게시글은 위 강의 및 교재를 기반으로 정리한 글입니다.
</div>