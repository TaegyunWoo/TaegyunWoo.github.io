---
category: OS
tags: [OS]
title: "[OS] 다단계 큐, 멀티 프로세서, 실시간 시스템"
date:   2021-11-15 21:00:00 
lastmod : 2021-11-15 21:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 이전 게시글
    - [[OS] CPU 스케줄링](https://taegyunwoo.github.io/os/OS_CPU_Scheduling)

이전 게시글과 이어지는 내용을 작성합니다. 이전 글을 읽고 오시는 것을 추천드립니다.

<br/><br/><br/>

# 다단계 큐와 멀티 프로세서, 실시간 시스템까지

## Multilevel Queue (다단계 큐)

### 다단계 큐란?

- 준비 큐를 아래와 같이 분리한 것을 말한다.
    - **Foreground Queue**
    - **Backgorund Queue**

<br/>

- **Foreground Queue**
    - 사용하는 스케줄링 방식
        - Round Robin
    - 특징
        - 사용자에게 높은 응답속도를 제공해야 한다.
        - 따라서, 응답속도 위주인 Round Robin을 사용한다.

- **Background Queue**
    - 사용하는 스케줄링 방식
        - FCFS
    - 특징
        - 처리량을 중점으로 두기 때문에, FCFS를 사용한다.

<br/>

### 큐 간의 스케줄링

- 프로세스 스케줄링이 아닌, 어떤 큐를 사용할 것인지에 대한 스케줄링을 할 수 있다.
    - 'Foreground Queue' vs 'Background Queue'

<br/>

- 스케줄링 방법
    - **고정 우선순위 스케줄링 (Fixed Priority Scheduling)**
        - Foreground Queue의 Task 들이 완료될 때까지, Background Queue의 Task들이 기다린다.
        - 즉, 우선순위가 낮은 Task들은 평생 기다려야할 수 있다.
        - 왜냐하면 **고정** 우선순위이기 때문이다. (Aging을 사용하면 극복할 수 있는 문제이다.)
    - **시분할 (Time Slice)**
        - 각 큐마다 사용할 수 있는 CPU 시간을 분배하는 방법이다.
        - 예시) Foreground Queue = 80% CPU 시간 할당, Background Queue = 20% CPU 시간 할당

<br/>

### 다단계 큐 스케줄링 예시

![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2042.png)

<br/>

### 다단계 피드백 큐

- 다단계 피드백 큐란?
    - **프로세스들이 큐 간 이동을 할 수 있는 다단계 큐 방식이다.**

<br/>

- 예시
    - 존재하는 큐 종류
        - Q0 : RR방식, q=8ms
        - Q1 : RR방식, q=16ms
        - Q2 : FCFS방식
    - 동작
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2043.png)
        
    - Burst Time이 35인 프로세스 처리시
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2044.png)
        
<br/><br/>

## Thread Scheduling

### 스케줄링에서의 쓰레드

- 쓰레드는 '유저 레벨'과 '커널 레벨'로 구분한다.
    
    > [이전 게시글 참고](https://taegyunwoo.github.io/os/OS_Thread#18)
    > 
- **시스템에서 쓰레드를 지원할 경우, 프로세스가 아닌 쓰레드를 스케줄링한다.**
    - 대부분의 시스템에서 쓰레드를 지원한다.
    - **따라서 대부분의 시스템이 쓰레드를 스케줄 한다.**

<br/>

- 쓰레드 스케줄링의 범위
    - **PCS**
        - Process-Contention Scope
        - 프로세스 내에서 쓰레드끼리 경쟁한다.
    - **SCS**
        - System-Contention Scope
        - 전체 시스템 내에서 쓰레드끼리 경쟁한다.

<br/>

- 리눅스에서의 쓰레드 스케줄링 예시
    - 리눅스는 Pthread Scheduling 이라는 방식으로 쓰레드를 스케줄링한다.
    - **Pthread Scheduling은 SCS 기법에 기반한다.**

<br/><br/>

## 다중 프로세서 스케줄링

### 다중 프로세서 스케줄링이란?

- Multi Core 시스템에서의 스케줄링 방식을 말한다.
- 다중 프로세서 스케줄링의 종류
    - **비대칭 다중처리**
        - master 프로세서 혼자 모든 자료구조를 관리한다.
        - 따라서, 병목현상이 발생한다.
    - **대칭 다중처리 (SMP)**
        - **"모든 프로세스가 하나의 Ready Queue에 들어가 있는 방식"** 혹은,
        - **"프로세스가 각 코어의 독립된 큐에 들어가 있는 방식"** 이 존재한다.
    
    > 비대칭·대칭 다중처리가 무엇인지 모른다면, [이전 게시글](https://taegyunwoo.github.io/os/OS_Summary1#23)을 참고하자.
    > 

*본 게시글에서는 대칭 다중처리에 대해서만 다루겠다. (가장 많이 사용되는 방식이다.)*

<br/>

### SMP Queue 구조

- **모든 프로세스가 하나의 Ready Queue에 들어가 있는 방식**
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2045.png)
    

- **프로세스가 각 코어의 독립된 큐에 들어가 있는 방식 (Per-Core Queue)**
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2046.png)
    
    - **부하 균등 (Load Balancing)**
        - Core 당 할당된 Task 개수가 같게끔 맞추는 것을 말한다.
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2047.png)
        
    - **부하 균등의 문제점**
        1. 부하 균등을 하고자 Task를 다른 Core로 옮기면 ⇒ Cache Miss 가 높아진다.  
        (자세한 것은 이후에 설명하겠다.)
        2. Cache Miss가 발생하면, **메모리 멈춤 현상 (Memory Stalling)** 이 발생한다.
            - 메모리 멈춤 현상: 데이터를 가져오느라 프로세서가 idle상태에 놓이는 것
    - **메모리 멈춤 현상 방지책 = Affinity**
        - Affinity란, 특정 Core에서만 Task가 수행되도록 하는 것이다. (Load Balancing을 하지 않는다.)
        - Affinity 기능을 통해, Cache Hit 확률을 높일 수 있다.
        - 따라서, Affinity 적용시 메모리 멈춤 현상을 최소화할 수 있다.
    
    > 바로 아래에서 자세히 설명하겠다.
    > 

<br/>

### Processor Affinity (프로세서 친화성)

- **Processor Affinity 적용을 안했을 때**
    - step 1
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2048.png)
        
    - step 2
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2049.png)
        
    - step 3
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2050.png)
        

- **Processor Affinity 적용했을 때**
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2051.png)
    
<br/>

### 부하 균등

- 부하 균등이란?
    - 부하가 모든 CPU에 동일하게 분산되도록 하는 것이다.
    - 특정한 CPU만이 많이 일하거나, 특정한 CPU가 놀지 않는 경우가 없도록 한다.

- 부하 균등의 방식
    - **푸시 이주 (Push Migration)**
        - 개별 프로세서에 대한 부하를 주기적으로 확인한다.
        - 특별히 많은 부하가 걸린 CPU가 있으면 다른 CPU로 부하를 분산한다.
    - **풀 이주 (Pull Migration)**
        - 놀고 있는(Idle 상태의) CPU들이 바쁜 프로세서들의 Task를 가지고 온다.

<br/>

### 멀티코어 프로세서의 특성

- 보통 물리적 칩 내에, 여러 개의 Core 들을 배치하는 것이 일반적이다.
- 기술이 발전함에 따라, 코어마다 다중 쓰레드들도 더 많아지고 있다.
- **하나의 쓰레드가 메모리 탐색을 위해 Stall (메모리 멈춤) 되었을 때, 다른 쓰레드가 일하게 만들어서 효율을 더 올릴 수 있다.**
    - 단일 쓰레드일 때
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2052.png)
        
    - 다중 쓰레드일 때
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2053.png)
        
<br/><br/>

## Real-Time CPU Scheduling

### 실시간 시스템의 종류

- **연성 실시간 시스템**
    - 비교적 엄격하지 않은 조건
    - 경우에 따라, Task 처리 데드라인에 못 맞출 수 도 있다.
    - 예시) 이동통신
- **강성 실시간 시스템**
    - 매우 엄격한 조건
    - 무조건 Task 처리 데드라인에 맞추는 것을 목표로 한다.
    - 예시) 미사일 시스템

<br/>

### 실시간 시스템에서의 지연의 종류

- **Interrupt Latency**
    - 인터럽트 발생부터 ISR(Interrupt Service Routine) 실행까지의 시간을 말한다.
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2054.png)
    
- **Dispatch Latency**
    - conflicts 와 dispatch 시간의 합을 의미한다.
    - conflicts : 실행 중인 프로세스를 Block하는 시간
    - dispatch : 새 프로세스를 할당하는 시간
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2055.png)
    
<br/>

### Priority-based Scheduling

- **선점형 우선순위 스케줄링**을 통해, 연성 실시간 스케줄링을 구현한다.
    - 강성 실시간 시스템에서는 이것만으로 부족하다.

- **주기성**
    - 프로세스들이 갖는 특성 중 하나이다.
    - 특성 주기마다 스케줄링을 요구한다는 특성을 말한다.
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2056.png)
    
    - **0 ≤ t ≤ d ≤ p**
        - t: 처리시간
        - d: 마감시간(deadline)
        - p: 주기
        - 이들 간에, 위 관계가 성립해야 한다.

<br/>

### Rate Montonic Scheduling

- Rate Montonic Scheduling 이란?
    - **선점형 정적 우선순위 스케줄링**을 이용하는 스케줄링 방법이다.
    - 주기가 짧은 Task = 높은 우선순위
    - 주기가 긴 Task = 낮은 우선순위

<br/>

- **Rate Montonic Scheduling 동작 예시**
    - 가정
        - P1: 주기=50, 수행시간=20
        - P2: 주기=100, 수행시간=35
        
        > 최대로 허용되는 데드라인은 주기와 동일하다.
        > 
    - 동작
        1. 두 프로세스(P1, P2)가 마감시간을 충족시켜 스케줄링이 가능한가?
            - P1의 CPU 이용률 = 20/50 = 0.4
            - P2의 CPU 이용률 = 35/100 = 0.35
            - 총 CPU 이용률 = 0.75
            - 총 CPU 이용률이 1을 넘지 않는다. ⇒ 즉, 데드라인을 충족시킬 수 있다.
        2. 스케줄링 예시
            
            ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2057.png)
            
<br/>

- 실시간 스케줄링에서 주로 사용되는 스케줄링 방식
    - **Earliest Deadline First Scheduling (EDF)**

<br/><br/>

## CPU 스케줄링 알고리즘 평가

### 평가 종류

- **분석적 평가 (결정론적 모델링)**
    - 미리 결정되어 있는 매개변수(input)를 통해 성능 평가를 수행한다.
    - 결과를 정확히 알 수 있다.
    - 실시간 스케줄링을 평가하기엔 어렵다.
        - 실시간 스케줄링의 경우, 모든 input을 미리 알 수 없기 때문이다.
    - input이란?
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2058.png)
        
        - 위와 같은 정보(Burst Time, ...)들
- **시뮬레이션**
    - 분석적 평가를 하기 힘든, 실시간 스케줄링을 평가하는 방법이다.

> 본 포스팅에선 분석적 평가(결정론적 평가)만 다룬다.
> 

<br/>

### 결정론적 평가 예시

- 목표
    - 모든 input을 알 때, 각 스케줄링 알고리즘에 대해 최소 평균 대기 시간을 구하자.

- 예시 input
    
    ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2059.png)
    
    - 도착시간은 모두 0으로 같다.

- 예시 알고리즘
    - **FCFS**
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2060.png)
        
        - 최소 평균 대기 시간: (0+10+39+42+49)/5 = 28
    - **비선점형 SJF (Shortest Job First)**
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2061.png)
        
        - 최소 평균 대기 시간: (0+3+10+20+32)/5 = 13
    - **RR (Round Robin)**
        - q = 10 일때
        
        ![Untitled](/assets/img/2021-11-15-OS_MultiLayerQueue_MultiProcessor_RealtimeSystem/Untitled%2062.png)
        
        - 최소 평균 대기 시간: (0+(10+20+2)+20+23+(30+10))/5=23
        - P1 대기시간: 0
        - P2 대기시간: 0\~10, 20\~40, 50\~52
        - P3 대기시간: 0\~20
        - P4 대기시간: 0\~23
        - P5 대기시간: 0\~30, 40\~50

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