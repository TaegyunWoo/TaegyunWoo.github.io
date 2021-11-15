---
category: OS
tags: [OS]
title: "[OS] CPU 스케줄링"
date:   2021-11-10 09:00:00 
lastmod : 2021-11-10 09:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# CPU 스케줄링

## 개요

### 기본 개념

- CPU 스케줄링의 목적
    - CPU 사용률의 극대화

- 보편적인 CPU 작업 사이클의 특성
    - 일반적으로 CPU는 아래 그림과 같은 작업 사이클을 갖는다.
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2018.png)
    
    - **일반적으로 I/O wait의 시간이 CPU execution 시간보다 훨씬 길다.**
    - 보통 CPU Burst 후, I/O Burst가 온다.
    - Burst : 집중 사용시간, 연속사용시간

<br/>

- **일반적인 CPU Burst 길이 별 빈도수 히스토그램**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2019.png)
    
<br/>

### CPU 스케줄러란?

- **CPU 스케줄러 == Short-term Scheduler**
- Short-term Scheduler 란?
    - **ready → running 프로세스 상태로 바꿔주는 스케줄러이다.**
    - **즉, ready queue에서 프로세스를 선택하여 CPU에 할당시키는 스케줄러이다.**
        - **이때 스케줄링 기준(Criteria)에 따라, 어떤 프로세스를 선택할지 달라진다!**
    
    > 자세한 것은 [이전 포스팅](https://taegyunwoo.github.io/os/OS_Process1#24)을 참고하자.
    > 

- **CPU 스케줄러(Short-term Scheduler)가 동작하는 부분**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2020.png)
    

- (ready)큐: 연결 큐
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2021.png)
    
<br/><br/>

## 선점 스케줄링

### CPU 스케줄링 결정이 일어나는 상황

아래 항목들은 프로세스의 상태가 변화함에 따라, CPU 스케줄링 결정이 일어나는 상황이다.

1. **Running → Waiting**
2. **Running → Ready**
3. **Waiting → Ready**
4. **New → Ready**
5. **Terminates**

![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2022.png)

> 프로세스 상태는 [이전 게시글](https://taegyunwoo.github.io/os/OS_Process1#8)을 참고하자.

<br/>

- **Ready → Running 은?**
    - **해당 경우는 CPU 스케줄링 결정이 일어나지 않는다.**
    - **이 경우에는, 스케줄러가 이미 정리해둔(기준;Criteria에 따라 정리) Ready Queue 에서 Dispatcher가 프로세스를 꺼내어 CPU로 할당시키는 것이기 때문이다.**
    - 즉, Dispatcher가 이미 정리되어 있는 Ready Queue에서 단순히 프로세스를 꺼내는 것이기 때문이다.

<br/>

### 선점 스케줄링이란?

- 선점 스케줄링 == Preemptive Scheduling
    - 선점 스케줄링은 새치기와 비슷한 개념으로 생각할 수 있다.
    - 선점: CPU를 강제로 할당받는다.

<br/>

### 선점이 일어나는 경우 (CPU 스케줄링 결정에서)

- **Running → Ready**
- **Waiting → Ready**
- **New → Ready**
- 위 경우들은 모두 **선점으로 동작**한다.

<br/>

### 선점이 일어나지 않는 경우 (CPU 스케줄링 결정에서)

- **Running → Waiting**
- **Terminates**
- 위 경우들은 모두 **비선점으로 동작**한다.
- **즉, '해당 작업이 완료될 때(CPU 할당 후 수행 완료)'까지 방해받지 않는다.**

<br/>

### 선점 동작 방식에서의 주의사항

- 공유하는 자료를 갱신하고 있는 동안 선점 당하면 문제가 생긴다.
- 커널 작업 중에 선점 당하면, 커널의 중요한 자료에 문제가 생길 수 있다.
- 중요한 운영체제 작업 중에는 인터럽트의 발생을 불허한다.

<br/><br/>

## 디스패처 (Dispatcher)

### 디스패처의 기능

- 디스패처는 Short-term Scheduler(CPU 스케줄러)가 선택한 프로세스에게 CPU 제어권을 준다.
    - 이때 'Ready → Running'으로 상태가 변화한다.
    - **따라서 'Ready → Running'에서는 스케줄링 결정이 이루어지지 않는다!**

<br/>

### 디스패처에 의한 제어권 변경 예시

![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2023.png)

- **디스패치 지연 (Dispatch Latency)**
    - 디스패치 지연이란, **디스패처가 현재 실행 중인 프로세스를 멈추고, 다른 프로세스를 실행하는 시간을 의미**한다.
    - 즉, Context Switching 시간을 말한다.

<br/><br/>

## 스케줄링 기준 (Scheduling Criteria)

### 스케줄링하는 기준의 종류

- **CPU Utilization (CPU 이용률)**
    - CPU를 최대한 일하게 한다.
- **Throughput (처리량)**
    - 단위시간당 실행을 완료하는 프로세스들의 개수
- **Turnaround Time (총 처리 시간)**
    - 프로세스의 제출시간과 완료시간의 간격
        - 제출시간: 프로세스가 Ready Queue에 들어온 시간
        - 완료시간: 프로세스가 'Terminated'되거나, 'Waiting을 거쳐 Ready'로 변경된 순간의 시간(시각) ⇒ 'I/O 시간'을 의미한다.
    - 'Ready Queue 대기시간' + 'CPU 실행시간' + 'I/O 시간'
- **Waiting Time (대기 시간)**
    - Ready Queue에서 대기하면서 보낸 시간의 합
- **Response Time (응답 시간)**
    - 대화식 시스템에서 응답이 시작되는 데 까지 걸리는 시간
    - 예시) 터미널에 키보드로 A 입력시, A가 입력되었음을 시스템이 알아차리는데 걸리는 시간
    - 평균 응답시간이 짧아야 할 뿐만 아니라, 응답시간의 편차도 작아야 한다.
        - 즉, 어쩔땐 짧고 어쩔땐 길면 안된다.

<br/>

### 스케줄링 알고리즘을 최적화하는 기준

- Max CPU Utilization (CPU 이용률 최대화)
- Max Throughput (처리량 최대화)
- Min Turnaround Time (총 처리시간 최소화)
- Min Waiting Time (대기시간 최소화)
- Min Response Time (응답시간 최소화)

위와 같은 기준으로 스케줄링 알고리즘을 최적화할 수 있다.

<br/><br/>

## CPU 스케줄링 종류: FCFS

### First-Come, First-Served Scheduling (FCFS)

- **FCFS == FIFO == 선입선출**
    - 모두 같은 의미이다.
- **FCFS는 비선점 방식의 스케줄링 방법이다.**
    - 즉, 중간에 CPU를 뺏기지 않는다.
- Short-term Scheduler가 동작하는 방식 중 하나이다.

<br/>

- 바로 예시를 통해, 어떻게 스케줄링하는지 알아보자.
- 예시
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2024.png)
    
    - 각 프로세스의 Burst Time이 위와 같다고 가정하자.
    - 그리고 프로세스가 순차적으로 도착했다고 가정하자.
        - "P1 , P2 , P3" 순으로 도착
    - 이때, FCFS 스케줄링이 작동하는 방식을 간트차트로 나타낸 것이 아래 그림이다.
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2025.png)
    
    - **대기시간**
        - **FCFS에서의 특정 프로세스 대기시간 = 특정 프로세스의 시작시간**
        - P1 = 0
        - P2 = 24
        - P3 = 27
    - **평균 대기시간**
        - (0+24+27)/3 = 17

<br/>

- **호위 효과 (Convoy Effect)**
    - **FCFS 방식은 호위 효과라는 것을 유발시킨다.**
    - 호위 효과란, 긴 프로세스 뒤에 작은 프로세스가 있는 것을 말한다.

<br/>

### First-Come, First-Served Scheduling (FCFS) 예시

또 다른 예시를 보자.

- 가정
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2026.png)
    
    - 프로세스 도착 순서는 아래와 같다고 하자.
        - P2 , P3 , P1

- 간트차트 시각화
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2027.png)
    
    - **대기 시간**
        - P1 = 6
        - P2 = 0
        - P3 = 3
    - **평균 대기 시간**
        - (6+0+3)/3 = 3

- **이 경우, 호위 효과가 발생하지 않아 평균 대기 시간이 비교적 짧아졌다. (위 예시보단)**

<br/><br/>

## CPU 스케줄링 종류: SJF

### Shortest-Job-First Scheduling (SJF)

- SJF란?
    - 최단작업우선 방식
    - **가장 작은 CPU 버스트를 갖는 프로세스를 다음 프로세스로 스케줄링한다.**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2028.png)
    
<br/>

- **SJF 방식이 최적의 스케줄링 방식이다.**
    - 이것은 가장 적은 평균 대기 시간을 보장한다.
    - **하지만 각 프로세스의 CPU Burst Time을 미리 알기가 힘들다.**

<br/>

### Shortest-Job-First Scheduling (SJF) 예시

- 가정
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2029.png)
    

- 간트차트 시각화
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2030.png)
    
    - 짧은 Burst Time 순으로 실행된다.

- **평균 대기 시간**
    - (3 + 16 + 9 + 0) / 4 = 7

<br/>

### SJF 에서 Burst Time을 예측하는 방법

- 위에서 설명했듯이, SJF는 가장 좋은 스케줄링 방법이지만 **CPU Burst Time을 미리 알 방법이 없다.**
- 따라서, Burst Time을 예측해야한다.
    
    > CPU Burst Time을 Burst Time으로 줄여서 이야기하겠다.

- SJF의 종류
    - **SJF는 '선점형', '비선점형' 모두 존재한다.**
    - 선점형의 경우, 가장 적게 남은 시간을 우선한다.
    
    > 이후에, 예시를 통해 설명한다.
    > 

<br/>

- 예측 방법
    - **'다음 프로세스의 버스트길이'가 '직전 프로세스의 버스트길이'와 비슷할 것이다.**
    - 위 가정을 통해, 버스트 길이(Time)를 예측한다.
- 예측 함수
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\tau_{n+1}=\alpha{t_n}+(1-\alpha)\tau_n)
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\tau_{n+1}) : 다음 프로세스(n+1)의 버스트 길이 예측값
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\alpha) : 가중치 (일반적으로 1/2로 설정한다.)
    - ![O(n^2)](https://latex.codecogs.com/svg.image?t_n) : 직전 프로세스(n)의 실제 버스트 길이
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\tau_n) : 직전 프로세스(n)의 버스트 길이 예측값

<br/>

### 예측 함수의 상황들

- ![O(n^2)](https://latex.codecogs.com/svg.image?\alpha=0) 인 경우
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\tau_{n+1}=\tau_n)
    - 즉, 최근 히스토리(실제값)을 고려하지 않게 된다.

<br/>

- ![O(n^2)](https://latex.codecogs.com/svg.image?\alpha=1) 인 경우
    - ![O(n^2)](https://latex.codecogs.com/svg.image?\tau_{n+1}=t_n)
    - 즉, 최근 히스토리(실제값)만 고려한다.

<br/>

### 다음 프로세스의 Burst Time 예측 그래프 예시

![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2031.png)

<br/>

### Shortest-Remaining-Time-First (SRTF)

- SRTF 란?
    - **최소잔여우선**
    - **선점형 SJF를 SRTF라고 한다.**
    - SRTF는 분석에 있어서, 도착시간이 다른 것과 선점의 개념을 추가한다.

<br/>

- **SRTF에서 Turnaround Time (총처리시간) 구하기**
    - 프로세스의 Turnaround Time = 완료시간 - 도착시간

- **SRTF에서 Waiting Time (대기시간) 구하기**
    - 프로세스의 Waiting Time = Turnaround Time - Burst Time

<br/>

- 바로 예시를 통해 알아보자.
- 가정
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2032.png)
    

- 간트차트 시각화
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2033.png)
    

- 평균 대기 시간
    - Turnaround Time
        - P1 Turnaround Time : 17 - 0 = 17
        - P2 Turnaround Time : 5 - 1 = 4
        - P3 Turnaround Time : 26 - 2 = 24
        - P4 Turnaround Time : 10 - 3 = 7
    - Waiting Time
        - P1 Waiting Time : 17 - 8 = 9
        - P2 Waiting Time : 4 - 4 = 0
        - P3 Waiting Time : 24 - 9 = 15
        - P4 Waiting Time : 7 - 5 = 2
    - 평균 대기 시간
        - (9 + 0 + 15 + 2) / 4 = 6.5

<br/><br/>

## CPU 스케줄링 종류: 우선순위 스케줄링

### Priority Scheduling

- Priority Scheduling 란?
    - 우선순위 스케줄링
    - 각 프로세스에 숫자로 우선순위를 표기한다.
    - 그리고 우선순위에 따라, 스케줄링 한다.

<br/>

- Priority Scheduling 종류
    - **선점형(Preemptive) Priority Scheduling**
        - '새로 도착한 프로세스의 우선순위'가 '현재 실행되는 프로세스의 우선순위'보다 높으면 CPU를 선점한다.
    - **비선점형(Nonpreemptive) Priority Scheduling**
        - 우선순위가 가장 높은 프로세스를 Ready Queue의 맨 앞에 넣는다.

<br/>

- 우선순위는 Burst Time(길이)에 반비례한다.
    - 긴 Burst ⇒ 낮은 우선순위
    - 짧은 Burst ⇒ 높은 우선순위

<br/>

- 문제점
    - **우선순위 스케줄링의 경우, 기아상태가 발생할 수 있다.**
    - 기아상태: 낮은 우선순위의 프로세스가 계속해서 대기만 하는 상태
- 해결방안
    - **Aging 기법을 도입한다.**
    - Aging : 기다린 시간에 비례해서 우선순위를 점점 증가시키는 방법

<br/>

### Priority Scheduling 예시

- 가정
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2034.png)
    
    - 도착시간은 없으므로, 비선점형 우선순위 스케줄링이라고 가정한다.
    - 보통 Burst Time을 고려해서, 우선순위를 결정하지만 정확하지 않을 수 있다.

- 간트차트 시각화
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2035.png)
    
    - 단순히 우선순위가 가장 높은 순서대로 실행된다.

<br/><br/>

## CPU 스케줄링 종류: 라운드 로빈

### Round Robin

- Round Robin이란?
    - 시분할 시스템을 위해 설계된 스케줄링 방식이다.
    - 각 프로세스는 '10~100 밀리초'의 CPU 시간을 할당 받는다.
        - 시간할당량(CPU 시간) = q (Quantum)
    - 시간 q가 지나면, 실행 중이던 프로세스가 선점당하여 준비완료 큐의 끝에 가게 된다.
        - 선점: 새치기

<br/>

- 예시
    - 가정
        - 준비완료 큐: 5개의 프로세스 존재
        - q : 20밀리초
    - 이때, 각 프로세스는 매 100밀리초마다 최대 20밀리초를 할당받게 된다.

<br/>

### 시간할당량(q)에 따른 성능

- **q가 너무 클 때**
    - FCFS와 동일하게 동작한다.
    - Convoy Effect 가 발생할 수 있다.
- **q가 너무 작을 때**
    - Context Switching 이 너무 빈번하게 일어난다.
    - 따라서 오버헤드가 자주 발생하여 성능이 떨어진다.
- 정리
    - 따라서 시간할당량(q)는 보통 10ms ~ 100ms로 설정한다.
    - 그리고 Context Switching 시 오버헤드는 10마이크로초 이내여야 한다.

<br/>

### Round Robin 예시

- 가정
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2036.png)
    

- 간트차트 시각화
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2037.png)
    
<br/>

### Round Robin 특징

- 일반적으로 SJF 보다 높은 평균 처리시간(Turnaround Time)을 갖는다.
- 응답시간(Response Time) 측면에서는 RR방식이 더 우수하다.
    - 각 프로세스들에게 공평하게 기회를 주기 때문이다.

<br/>

### Time Quantum(q)와 Context Switching 간의 관계

- 가정
    - 프로세스의 CPU 시간은 10이다. (Burst Time = 10)
- **q가 12일 때**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2038.png)
    
    - Context Switching이 일어나지 않는다.
- **q가 6일 때**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2039.png)
    
- **q가 1일 때**
    
    ![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2040.png)
    
<br/>

### Time Quantum(q)와 Turnaround Time 간의 관계 예시

![Untitled](/assets/img/2021-11-10-OS_CPU_Scheduling/Untitled%2041.png)

- 결론
    - 프로세스들 중 80%가 'q보다 낮은 Burst Time'을 가져야 한다.
    - 이때, 일반적으로 좋은 성능을 기대할 수 있다.

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