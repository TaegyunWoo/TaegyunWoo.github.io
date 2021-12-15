---
category: OS
tags: [OS]
title: "[OS] 프로세스 동기화를 위한 여러 방법"
date:   2021-11-25 19:30:00 
lastmod : 2021-11-25 19:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 프로세스 동기화 방법

## 배경

### 개요

- 프로세스들은 기본적으로 병렬 수행된다.
    - 하지만 언제든 인터럽트가 걸릴 수 있다.
- 병렬수행하다 보면 **공용데이터 값에 불일치 문제**가 발생할 수 있다.
    - 따라서 불일치 문제를 해결할 방법들이 필요하다.

<br/>

### 버퍼 문제

- 생산자·소비자 문제에서 모든 버퍼를 다 채우면 발생하는 문제가 존재한다.
    - 솔루션 1
        - (BufferSize-1)개의 버퍼만을 사용하여 문제를 해결할 수 있다.
        - 자세한 것은 아래 포스팅 글을 참고하자.
        - [[데이터구조] 큐](https://taegyunwoo.github.io/datastructure/DATASTRUCTURE_Queue#11), [[OS] 프로세스 - 2](https://taegyunwoo.github.io/os/OS_Process2#10)
    - 솔루션 2
        - 정수 카운터를 통해서 버퍼가 다 찼는지 살펴보면 문제를 해결할 수 있다.
        - 계속해서 좀 더 자세히 알아보자.

<br/><br/>

## 버퍼 문제: 정수 카운터(counter)를 통한 버퍼 문제 해결

### 동작 방식

- **counter**
    - 버퍼를 얼마나 사용 중인지 개수를 센다.
- 시각화
    
    ![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2063.png)
    
- Producer (생산자) 코드
    
    ```c
    while (true) {
    
    	//버퍼가 Full 상태일 때
    	while (counter == BUFFER_SIZE) {
    		/* 아무것도 수행하지 않는다.*/
    	}
    
    	//버퍼에 데이터를 넣을 수 있을 때
    	buffer[in] = next_produced;
    	in = (in + 1) % BUFFER_SIZE;
    	counter++;
    }
    ```
    
- Consumer (소비자) 코드
    
    ```c
    while (true) {
    
    	//버퍼가 Empty 상태일 때
    	while (counter == 0) {
    		/* 아무것도 수행하지 않는다.*/
    	}
    
    	//버퍼에서 데이터를 가져올 수 있을 때
    	next_consumed = buffer[out];
    	out = (out + 1) % BUFFER_SIZE;
    	counter--;
    }
    ```
    
<br/>

### Counter로 버퍼 사용시 발생하는 문제: 경쟁 상황

- `counter++` 를 기계명령어로 변환시
    - `register1 = counter`
    - `register1 = register1 + 1`
    - `counter = register1`
- `counter--` 를 기계명령어로 변환시
    - `register2 = counter`
    - `register2 = register2 - 1`
    - `counter = register2`

<br/>

- **문제 상황: 프로세스 병렬처리로 인해, 명령어 실행 순서가 섞였을 때**
    - 가정: counter=5인 상태
    - Step 0 - producer
        - `register1 = counter` 실행
        - 결과: 'register1 = 5'
    - Step 1 - producer
        - `register1 = register1 + 1` 실행
        - 결과: 'register1 = 6'
            - 아직 register1의 값(6)을 counter 변수에 저장하지는 않았다.
    - Step 2 - consumer
        - `register2 = counter` 실행
        - 결과: 'register2 = 5'
            - 아직 counter가 5이므로, register2에 5가 저장된다.
    - Step 3 - consumer
        - `register2 = register2 - 1` 실행
        - 결과: 'register2 = 4'
            - 아직 register2의 값(4)을 counter 변수에 저장하지는 않았다.
    - Step 4 - producer
        - `counter = register1` 실행
        - 결과: 'counter = 6'
            - 이제서야 counter에 register1의 값(6)을 저장하므로, 6이 저장된다.
    - Step 5 - consumer
        - `counter = register2` 실행
        - 결과: 'counter = 4'
            - 이제서야 counter에 register2의 값(4)을 저장하므로, 4이 저장된다.
    - **결론**
        - 프로세스가 병렬처리되지 않았다면(정상적으로 수행되었다면) 최종적으로 counter의 값이 5이어야 한다.
        - **하지만 공유데이터(counter)를 병렬적으로 사용하여, 잘못된 결과가 나왔다.**
        - **즉, 기계명령어 실행 순서에 의해 값이 달라졌다.**

<br/>

- 위와 같은 문제상황을 **경쟁상황(Race Condition)** 이라고 한다.
    - 경쟁상황이란?
        - 동시에 여러 프로세스가 동일한 자료를 접근하는 상황이다.
        - 실행결과가 접근한 순서에 의존한다.

<br/><br/>

## 임계 구역 문제

### 임계 구역이란?

- 서로 다른 프로세스가 공유하는 변수·테이블·파일 등을 변경하는 작업을 포함하는 코드이다.
- 즉, 공유영역을 말한다.
- 임계 구역 == Critical Section (CS)

> 위에서 살펴본 '버퍼의 counter 문제' 역시 임계구역에서 발생한 문제이다.
> 

<br/>

### 임계 구역 문제

- 위에서 살펴본 경쟁상황(Race Condition) 역시, 임계 구역 문제 중 하나이다.
- 임계 구역 문제 해결방법
    - **하나의 프로세스가 CS(임계 구역) 내에 있으면, 다른 프로세스들은 CS에 들어오지 못한다.**

<br/>

### 임계 구역 구조

![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2064.png)

- **Entry Section (CS 진입구역)**
    - CS(임계영역) 진입 허가 요청을 하는 부분이다.
- **Exit Section (CS 퇴장구역)**
    - CS(임계영역) 퇴장 시, CS를 나왔다는 Signal을 보내는 부분이다.
- **Remainder Section (나머지 구역)**
    - CS와 관계없는 나머지 영역이다.

<br/>

### 임계 구역 문제 해결을 위한 조건들

CS에서 발생할 수 있는 문제를 해결하기 위해선, 아래 조건들을 반드시 모두 만족해야 한다.

- **Mutual Exclusion (상호 배제)**
    - 프로세스 ![math](https://latex.codecogs.com/svg.image?P_i) 가 자신의 임계구역에서 실행된다면, 다른 프로세스들은 임계구역 내 실행이 불가능하다.
- **Progress (진행)**
    - 조건
        - 임계구역에서 실행하는 프로세스가 없다.
        - 임계구역으로 진입하려는 프로세스가 존재한다.
    - 위 두가지 조건이 만족할 경우, 임계구역으로 진입하는 프로세스의 선택이 무한대로 연기되면 안된다.
    - **즉 CS에 진입한 프로세스가 계속 없으면 안된다.**
- **Bounded Waiting (한정된 대기)**
    - 프로세스가 임계구역 진입을 요청하고 그 요청이 허용될 때까지, 다른 프로세스들이 임계구역에 진입하는 횟수에 제한이 있다.
    - **즉 무한대기 방지를 위해, 대기 가능한 프로세스 개수를 제한할 수 있다.**

<br/>

### 운영체제에서의 CS 영역 관리

- 시스템 종류
    - Preemptive (선점형 시스템)
    - Non-preemptive (비선점형 시스템)
- 선점형 시스템에서 CS를 다루는 것이 비선점형 시스템보다 어렵다.
    - 하지만 현대의 멀티 코어 멀티 프로그래밍 시스템에서는 Preemptive가 필수이다.
    - 따라서 잘 알아두어야 한다!

<br/><br/>

## S/W적 CS 문제 해결: Peterson's Algorithm

### Peterson's Algorithm 이란?

- **S/W적 해결 방법이다.**
- 프로세스 2개가 경쟁하는 상황에서 사용되는 솔루션이다.
- 동작 개요
    - 가정: `load` 와 `store` 기계 명령어를 수행할 땐, 인터럽트가 걸리지 않는다.
    - 사용되는 변수
        - `int turn`
            - 차례를 나타내는 변수
            - 0 또는 1 이 저장된다.
            - 임계 구역에 들어갈 프로세서가 P0인지, P1인지 결정한다.
        - `Boolean flag[2]`
            - 프로세스가 임계구역에 들어갈 준비가 되었는지 나타내는 변수
            - `flag[0]=true` : P0(프로세스0)이 임계 구역에 들어갈 준비가 되었음을 의미
            - `flag[1]=true` : P1(프로세스1)이 임계 구역에 들어갈 준비가 되었음을 의미
        
<br/>

### Peterson's Algorithm 동작 예시

- 아래 코드는 프로세스i(![math](https://latex.codecogs.com/svg.image?P_i))에 대한 동작 코드이다.
    - 프로세스 종류: ![math](https://latex.codecogs.com/svg.image?P_i,P_j)

```c
do {

	flag[i] = true;
	//프로세스i가 CS에 진입할 준비됨
	
	turn = j;
	//위에서 프로세스i가 준비되었음을 알리고, 프로세스j 차례로 돌려준다.
	//이때, flag[j]가 true라면 CS영역를 프로세스j가 먼저 사용하게 된다.
	//즉, 프로세스i가 CS영역을 쓰기 전에 프로세스j가 쓰고 싶어하는지 확인하고 양보한다.

	while (flag[j] && turn==j);
	//flag[j]: 프로세스j가 진입될 준비가 되었는가?
	//turn==j: 프로세스j가 실행될 순서인가?
	// 위 두가지가 모두 참이라면, 프로세스j가 CS에서 실행 중이라는 것이다.
	// 그러므로 프로세스i가 CS에 진입하지 못하고 대기한다. (무한루프)

	// [CS 영역]

	flag[i] = false;
	//CS 영역을 사용하고 난 뒤, CS영역을 사용완료했다고 알린다.

	// [나머지 영역]

} while (true);
```

<br/>

- 피터슨 알고리즘과 CS문제 해결 조건
    - **Mutual Exclusion (상호배제)**
        - 상대방이 실행 중(`flag[j] && turn==j`)일 땐, 진입하지 못한다.
        - 따라서 위 코드는 상호배제 조건에 만족한다.
    - **Progress (진행)**
        - 상대방이 준비되지 않으면 언제든지 진입할 수 있다.
        - 따라서 위 코드는 진행 조건에 만족한다.
    - **Bounded-waiting (한정된 대기)**
        - 둘이 준비되어도 한 프로세스가 끝나야 진입할 수 있다.
        - 따라서 위 코드는 한정된 대기 조건에 만족한다.

<br/>

## H/W적 CS 문제 해결: test_and_set Insturction

### H/W적 해결

- 위에서 살펴본 피터슨 알고리즘과 같은 **소프트웨어적 해결방법에는 한계**가 있다.
    - 단일 프로세서 시스템에서의 S/W적 해결방법
        - 임계영역 실행 시, 인터럽트 금지를 통해 CS 문제를 해결한다.
        - 따라서 굳이 H/W적 해결방법을 사용하지 않아도 된다.
    - 멀티 프로세서 시스템에서의 S/W적 해결방법
        - 인터럽트 금지를 사용할 수 없다. 왜냐하면, 모든 처리기에 인터럽트 불능 메시지를 전달해야하기 때문이다.
        - 즉 임계영역 실행 시, 단순히 인터럽트를 금지하는 방식으로 문제를 해결하는 것은 비효율적이다.
        - **따라서, Locking 기법으로 CS문제를 해결해야 한다. ⇒ H/W적 해결을 해야한다.**
- 따라서 H/W적 해결 방법을 사용해야 한다.

<br/>

- H/W적 해결의 핵심
    - **원자적 기계 명령어 사용**
        - 인터럽트가 되지 않는 특수한 명령어를 사용한다.
    - **Locking**
        - 프로세스가 CS를 사용하면, 자물쇠를 걸어 다른 프로세스가 진입하지 못하게 막는다.

<br/>

- Locking 기법의 기본 방식
    
    ![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2065.png)
    
    - `acquire lock`
        - lock을 얻는 부분
        - lock을 얻어야만 CS영역에 진입할 수 있다.
    - `release lock`
        - lock을 반환하는 부분
        - CS 실행 완료 시, lock을 돌려준다.

<br/>

### test_and_set Instruction 이란?

- 정의
    
    ```c
    boolean test_and_set (boolean *target) {
    
    	boolean rv = *target;
    	*target = true;
    	return rv;
    
    }
    ```
    
- 상세설명
    - 위 코드는 원자적으로 실행된다. 즉, 인터럽트가 없다.
    - 현재 `*target` 이 false일 때
        1. `rv`를 false로 설정한다.
        2. `*target` 를 true로 설정한다.
        3. `rv` 값을 반환한다.
    - 현재 `*target` 이 true일 때
        1. `rv`를 true로 설정한다.
        2. `*target` 를 true로 설정한다.
        3. `rv` 값을 반환한다.

<br/>

### test_and_set Instruction 사용 예시

- 아래는 test_and_set Instruction을 사용하는 예시 코드이다.
    
    ```c
    do {
    
    	while (test_and_set(&lock));
    	//만약 &lock이 true라면, 무한루프에 빠진다.
    	//만약 &lock이 false라면, 무한루프를 탈출한다.
    	//이렇게 무한루프로 CS진입을 막는 것을 Busy Waiting이라고 한다.
    
    	// [CS영역]
    
    	lock = false;
    	//cs영역 실행 완료 시, lock을 반환한다.
    
    	// [나머지 영역]
    
    } while (true);
    ```
    
<br/>

- 상세 설명
    - x가 false이면 (CS 미사용이면) x가 true로 set되고, while루프를 벗어난다.
    - x가 true면 (CS 사용이면) x가 true로 set되고, while루프에 머문다.
    - **결국 lock이라는 변수 하나로 CS 접근 허용을 제어한다.**

<br/><br/>

## H/W적 CS 문제 해결: compare_and_swap Instruction

### compare_and_swap Instruction 이란?

> 위에서 살펴본 test_and_set Instruction 와 전체 메커니즘은 유사하다.  
단순히 반환값으로 boolean형에서 int형을 사용하는 것 뿐이다.  
그러므로 간단히 살펴만 보자.
> 
- 정의
    
    ```c
    int compare_and_swap(int *value, int expected, int new_value) {
    
    	int temp = *value;
    
    	if (*value == expected) {
    		*value = new_value;
    	}
    
    	return temp;
    
    }
    ```
    
- 상세 설명
    - 위 코드는 원자적으로 실행된다. 즉, 인터럽트가 없다.
    - 전달받은 `*value` 의 값을 그대로 반환한다.

<br/>

### compare_and_swap Instruction 사용 예시

- 아래는 compare_and_swap Instruction을 사용하는 예시 코드이다.
    
    ```c
    do {
    
    	while (compare_and_swap(&lock, 0, 1) != 0);
    	//Busy Waiting
    
    	// [CS영역]
    
    	lock = 0;
    
    	// [나머지 영역]
    
    } while (true);
    ```
    
<br/><br/>

## H/W적 CS 문제 해결: Mutex Locks

### Mutex Locks의 배경

- 기존 Lock을 사용하는 방식들은 좀 복잡하다.
- 비교적 Mutex는 가장 단순하다.

### Mutex Locks 이란?

- 사용하는 주요 함수
    - `acquire()`
        - lock 획득
    - `release()`
        - lock 반환
- Mutex Locks의 문제점
    - 다른 Lock 방식에 비해, 더 간단하기는 하다.
    - 하지만 다른 Lock 방식들과 마찬가지로, **Busy Waiting 방식**을 사용한다.
        - **Busy Waiting: while문을 계속 돌며, CS 진입을 대기하는 것**
        - Busy Waiting == spinlock

<br/>

- 정의
    - `acquire()`
        
        ```c
        acquire() {
        
        	while (!available); //Busy Waiting
        
        	available = false;
        
        }
        ```
        
    - `release()`
        
        ```c
        release() {
        
        	available = true;
        
        }
        ```
        
<br/>

### Mutex Locks 사용 예시

![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2066.png)

<br/><br/>

## H/W적 CS 문제 해결: Semaphore (세마포어)

### Semaphore (세마포어)란?

- Busy Waiting이 필요없는 동기화 기법이다.
- 사용하는 변수
    - `int S`
- 사용하는 함수
    - `wait()`
        
        ```c
        wait(S) {
        	while (S <= 0); //Busy Waiting
        	S--;
        }
        ```
        
        - CS 진입 시도 함수
    - `signal()`
        
        ```c
        Signal(S) {
        	S++;
        }
        ```
        
        - CS 퇴장 함수

<br/>

### Semaphore 종류

- **Counting Semaphore (카운팅 세마포어)**
    
    > 이진 세마포어 이후에 설명하는 세마포어는 모두 이것을 의미한다.
    > 
- **Binary Semaphore (이진 세마포어)**
    - 이것은 Mutex Locks와 같다.

<br/>

### 이진 세마포어 예시

- 프로세스
    - P1
    - P2
- 가정
    - ![math](https://latex.codecogs.com/svg.image?S_1) 이 ![math](https://latex.codecogs.com/svg.image?S_2) 보다 먼저 수행되어야 한다.
    - `synch` 변수 초기값 = 0

![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2067.png)

- 상세 설명
    - Busy Waiting의 필요성을 극복하기 위해, `waiting()` , `signal()` 세마포어 연산 정의를 변경했다.
        - 즉 세마포어로 Busy Waiting을 해결했다.
    - Busy Waiting 해결 방법
        - 프로세스가 `wait()` 연산을 실행하고, 세마포어 값이 양수가 아니면, 프로세스를 Busy Waiting 대신 블록(중지) 시킬 수 있다.
        
        > 이 후에 좀 더 자세히 알아보자.
        > 
    - 자원이 방출되면 블록된 프로세스를 실행한다.

<br/>

### 세마포어를 통해, Busy Waiting하지 않는 방법

- 세마포어 큐를 이용한 연산을 통해, 프로세스를 블로킹(중지)할 수 있다.
    - 세마포어 큐란?
        
        ![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2068.png)
        
        ![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2069.png)
        
- 세마포어 큐를 이용한 연산
    - **block**
        - 호출한 프로세스를 중지(block)시키고 대기 큐에 넣는다.
    - **wakeup**
        - 대기 큐에 있는 프로세스를 꺼내어 준비완료 큐에 넣는다.

<br/>

- 동작 예시
    
    ![Untitled](/assets/img/2021-11-25-OS_SynchronizationTools/Untitled%2070.png)
    
<br/>

### Deadlock

- Deadlock이란?
    - 교착 상태
    - 둘 이상의 프로세스가 '다른 프로세스가 점유하고 있는 자원'을 서로 기다릴 때, 무한 대기 상태에 빠지는 것을 의미한다.

<br/>

### Starvation

- Starvation이란?
    - 기아 상태
    - 세마포어 큐에서 LIFO 순서로 제거할 경우에 발생한다.
        - LIFO 구조인 경우, 가장 먼저 들어온 프로세스가 영원히 기다릴 수 있다.

<br/>

### 우선순위 역전

- '우선 순위가 높은 프로세스'가 CS 영역을 사용하고자 할 때, '우선 순위가 낮은 프로세스'가 CS 영역을 이미 사용 중이라면, '우선 순위가 높은 프로세스'는 우선순위가 높더라도 기다려야 한다.

<br/>

- 예시
    - 가정
        - L : 우선순위 가장 낮은 프로세스
        - M : 중간 우선순위 프로세스
        - H : 우선순위 가장 높은 프로세스
    1. L 이 CS 영역을 사용하고 있다.
        - CPU 점유 = L
    2. **이때 H가 CS 영역을 사용하고자 하지만, CS영역이 사용 중이라 블록된다.**
        - CPU 점유 = L
    3. M이 CPU를 사용하고자 하여, 할당받는다. (CS 영역 사용 X)
        - CPU 점유 = M
        - **이때 CS영역을 사용 중이던 L은 M에게 CPU제어권을 뺏기고 기다린다.**
        - **이것 때문에 H는 M까지 기다려야 한다.**
    4. M의 작업이 끝나고, CPU는 L에게 할당된다.
        - CPU 점유 = L
        - L의 작업이 끝나야, H에게 CS 영역을 줄 수 있으므로
    5. L의 작업이 끝나고나서야, H가 CS 영역에 진입한다.
        - CPU 점유 = H

<br/>

- 문제 해결 방법: 우선순위 상속
    - '낮은 우선순위 프로세스'가 '높은 우선순위 프로세스'의 우선순위를 상속받아, CS를 실행한 후 Lock을 풀고 원래 우선순위로 복귀하는 것이다.
    - 예시
        1. L 이 CS 영역을 사용하고 있다.
            - CPU 점유 = L
        2. 이때 H가 CS 영역을 사용하고자 하지만, CS영역이 사용 중이라 블록된다.
            - CPU 점유 = L
        3. **L 이 H의 우선순위를 상속받아, M보다 우선순위가 높아졌다.**
        4. **M이 CPU를 사용하고자 한다. 하지만 할당받을 수 없다.**
            - CPU 점유 = L
            - **M보다 L의 우선순위가 높기 때문에**
        5. **L의 작업이 끝나고, CPU는 H에게 할당된다. 그리고 H는 CS영역을 사용할 수 있게 된다.**
            - CPU 점유 = H
            - L의 작업이 끝나서, lock을 돌려준다.
        6. H의 작업이 끝나고나서야, M이 CPU를 점유한다.
            - CPU 점유 = M

<br/>

### 세마포어의 문제점

- `signal()` 과 `wait()` 의 순서가 바뀌면, Deadlock이나 Starvation에 빠질 수 있다.

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