---
category: Comput-Struct
tags: [ComputerStructure, 기억장치, Memory]
title: "[Computer Structure] 기억장치"
date:   2021-09-30 17:06:00 
lastmod : 2021-09-30 17:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 컴퓨터 기억장치 시스템 개요

## 개요

### 메모리의 역할

- 프로그램 명령이나 데이터를 저장한다.
- 프로그램은 메모리에서 활동 중이다.
    - 따라서, 메인 메모리가 클수록 동시에 많은 프로그램을 실행할 수 있다.
    - 메인 메모리: RAM, Cache

<br/>

### 메인 메모리의 수행작업

- 현재 실행 중인 프로그램들을 저장한다.
- 메모리의 각 영역은 서로 침범하지 못하도록, 시스템 소프트웨어 프로그래머가 정한다.

<br/>

### 기억장치의 계층구조

![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2033.png)

<br/><br/>

## 기억장치의 주요 특성

### 주요 특성들

- **Location (위치)**
- **Capacity (용량)**
- **Unit of Transfer (전송 단위)**
- **Access Method (데이터 접근 방식)**
- **Performance (성능)**
- **Physical Type (물리적 유형에 따른 기억장치)**
- **Physical Characteristics (기억장치의 물리적인 특징)**

<br/>

### Location: 위치

- 내부기억 장치
- 외부기억 장치

위 두가지 위치로 기억장치가 구분된다.

<br/>

### Capacity: 용량

- 내부 기억장치의 용량
    - **바이트** 또는 **단어** 단위로 표현한다.
    - 일반적인 단어의 길이: 8, 16, 32 비트
    - 단어 = 메모리의 폭(너비)
    
    ![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2034.png)
    
    > 버스의 대역폭 역시 단어의 길이만큼 넓어한다.
    > 
    
- 외부 기억장치의 용량
    - 일반적으로 **바이트 단위**로 표현한다.

<br/>

### Unit of Transfer: 전송 단위

- 내부 기억장치에서의 전송 단위
    - 기억장치 모듈로의 **데이터 선들의 개수(Data Bus)** 와 같다.
    - 주기억장치에 있어서, **전송 단위는 한번에 읽고 쓸수있는 비트의 수**이다.
- 외부 기억장치에서의 전송 단위
    - 단어보다 훨씬 더 큰 단위인 **블록단위**를 사용한다.

![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2035.png)

- **단어 (word)**
    - 기억 장치 조직의 단위
    - 단어의 길이
        - 일반적으로 수를 표현하기 위해 사용되는 비트의 수
        - 명령어의 길이

- **주소지정 단위**
    - 주소의 길이 A비트와 주소지정 단위의 수 N간의 관계
    - ![2^A=N](https://latex.codecogs.com/svg.image?2^A=N)
        - A : 주소의 길이
        - N : 단어의 개수
    - 예시
        - 단어의 개수 = 1024 일때의 단어의 길이(A) 는?
        - ![2^{A}=1024](https://latex.codecogs.com/svg.image?2^{A}=1024) 이므로, ![A=10](https://latex.codecogs.com/svg.image?A=10)

<br/>

### Access Method: 데이터 접근 방식

![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2048.png)

<br/>

### Performance: 성능

- **액세스 시간 성능**
    - 임의 액세스 기억장치 (Cache, M.M 등의 반도체 장치)
        - 액세스 시간: 주소가 기억장치에 도착하는 순간부터 저장되거나 읽혀지는 순간까지의 시간
        - **액세스 시간이 모든 기억장소에 대해서 동일하다.**
    - 비임의 액세스 기억장치 (HDD, Tape 등)
        - 액세스 시간: 읽기-쓰기 매커니즘을 원하는 위치로 이동하는데 걸리는 시간
        - **데이터가 저장되어 있는 위치에 따라 다르다.**
        - 예시(HDD)
            
            ![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2036.png)
            

- 기억장치 사이클 시간
    - 주로 임의 액세스 기억장치에 적용된다.
    - 액세스 시간과 다음 액세스를 시작하기 위해 요구되는 동작에 걸리는 **추가적인 시간을 합한 시간**

- **전송률 성능**
    - 데이터가 기억장치로 전송되어 들어가거나 나오는 비율
    - 임의 액세스 기억장치
        - ![Transfer\_Rate=\frac{DataBus\_Bandwidth}{Cycling\_Time}](https://latex.codecogs.com/svg.image?TransferRate=\frac{DataBusBandwidth}{CyclingTime})
    - 비임의 액세스 기억장치
        - ![T_N = T_A + \frac{n}{R}](https://latex.codecogs.com/svg.image?T_N=T_A+\frac{n}{R})
        - 전송율 = ![\frac{1}{T_N}](https://latex.codecogs.com/svg.image?\frac{1}{T_N})
        - ![T_N](https://latex.codecogs.com/svg.image?T_N) : N개의 비트들을 읽거나 쓰는데 걸리는 평균 시간
        - ![T_A](https://latex.codecogs.com/svg.image?T_A) : 평균 액세스 시간
        - ![n](https://latex.codecogs.com/svg.image?n) : 비트들의 수
        - ![R](https://latex.codecogs.com/svg.image?R) : 전송률, 초당 비트수

<br/>

### **Physical Type:** 물리적 유형에 따른 기억장치

- 반도체
    - RAM, Cache, Register, SSD
- 자기
    - Disk, Tape
- 광
    - CD, DVD

<br/>

### **Physical Characteristics: 기억장치의 물리적인 특징**

- 휘발성 여부
- 삭제 가능 여부
    - ROM의 경우, 데이터 삭제가 불가능하다.

<br/><br/><br/>

# 기억장치 시스템

## 기억장치 설계시 고려사항

### 주요 고려사항

- 얼마나 큰가
- 얼마나 빠른가
- 가격은 얼마나 비싼가

<br/>

### 기억장치 주요 특성들 간의 Tradeoff

- 액세스 속도 ↑ , 비트당 가격 ↑
- 용량 ↑ , 비트당 가격 ↓
- 용량 ↑ , 액세스 속도 ↓

이러한 트레이드오프 관계를 해결하기 위해 **계층적인 기억장치**를 사용한다.

<br/>

### 계층적인 기억장치

![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2037.png)

<br/>

### 기억장치 종류와 액세스 시간

![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2038.png)

<br/><br/>

## 기억장치의 효용성

### 효용성 계산 예시 문제

- 문제
    - 프로세서가 두 레벨의 기억장치를 액세스한다고 가정하자.
    - 레벨 1은 1,000개의 단어를 저장하며 액세스 시간은 0.01ms이다.
    - 레벨 2는 100,000 개의 단어를 저장하며 액세스 시간은 0.1ms이다.
    - 만약 액세스될 단어가 레벨 1에 있다면, 프로세서는 그 단어를 직접 액세스한다.
    - 만약 액세스될 단어가 레벨 2에 있다면, 먼저 레벨 1로 이동된 다음에 프로세서에 의해 액세스된다.
    - 이때 기억장치 액세스의 95%가 레벨 1에서 발견된다고 가정할 때, 그 단어를 액세스하는 데 걸리는 평균 시간은?

<br/>

- 해결
    - 해당 단어 평균 접근 시간 = (레벨1에 있을 확률 \* 레벨1의 액세스 시간) + (레벨2에 있을 확률 \* (레벨1의 액세스 시간 + 레벨2의 액세스 시간))
    - 해당 단어 평균 접근 시간![R](https://latex.codecogs.com/svg.image?=(0.95*0.01ms)+(0.05*(0.01ms+0.1ms))=0.015ms)
    
    ![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2039.png)

<br/>

## 기억장치의 성능 개선 원리

### 참조의 지역성

- 프로그램 동작의 특성
    - 대부분의 경우 순차적으로 프로그램이 진행되므로, 바로 전에 인출된 마지막 명령어의 다음 명령어가 인출된다.
        
        > 프로그램이 실행되는 동안 순차적이지 않은 분기/호출 명령은 작은 부분을 차지한다.
        > 
    - 짧은 시간 동안에 몇 개의 프로시저(함수)들로 국한된 명령어들을 참조한다.
        - 즉, 한정된 수의 프로시저(함수)를 계속 호출하면서 실행하는 경향이 있다.
    - 대부분의 반복 구조는 여러 번 반복되는 비교적 적은 수의 명령어들로 구성되어 있다.
    - 대부분의 프로그램들에는 "배열 또는 레코드와 같은 데이터 구조를 처리하는 계산"이 많이 포함된다.
        - 즉, 서로 인접한 데이터에 대해 처리하는 로직이 많이 존재한다.
    - **이러한 특징을 참조의 지역성이라고 한다.**
    - **참조의 지역성을 최대한으로 활용하면 Cache에 원하는 Data나 명령어가 존재할 확률을 높일 수 있다.**

<br/>

- 참조의 지역성을 고려한 Cache와 Main Memory 동작 방식
    
    ![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2040.png)

<br/>

### 공간적 지역성

- 공간적 지역성이란?
    - 실행들이 특정 클러스터의 기억 장소들에 대하여 이루어지는 경향
    - (프로세스가 명령어들을 순차적으로 액세스하는 경향)
    - 즉, 명령어 A 실행시 물리적으로 인근에 있는 명령어를 다음에 실행할 가능성이 높다.

<br/>

### 시간적 지역성

- 시간적 지역성이란?
    - 프로세서가 최근에 사용되었던 기억 장소들을 액세스 하는 경향

<br/><br/>

## 기억장치 동작과 성능

### 기억장치의 동작

- 상위 단계 기억장치(M1)은 하위 단계 기억장치(M2)보다 용량이 더 작고, 빠르고, 비트당 가격이 비싸다.
- M1은 용량이 큰 M2의 내용의 일부분을 저장하는 임시 저장장치로 사용된다.
- 먼저 M1에 있는 항목을 액세스하려고 시도하되, 성공(Hit, 적중률)하면 액세스가 이루어진다.
- 성공하지 못하면, 기억장치의 한 블록이 M2에서 M1으로 복사되며, 그런 다음 액세스는 M1을 경유하여 이루어진다.

> Hit: M1(캐시)에 원하는 데이터나 명령어가 존재하는 경우

<br/>

### 기억장치의 성능

- **비용적 성능**
    - ![C_S=\frac{C_1S_1+C_2S_2}{S_1+S_2}](https://latex.codecogs.com/svg.image?C_s=\frac{C_1S_1+C_2S_2}{S_1+S_2})
        - ![C_S](https://latex.codecogs.com/svg.image?C_S) : 결합된 2단계 기억장치의 비트당 평균 비용
        - ![C_1](https://latex.codecogs.com/svg.image?C_1) : 상위 단계 기억장치 M1의 비트당 평균 비용
        - ![C_2](https://latex.codecogs.com/svg.image?C_2) : 하위 단계 기억장치 M2의 비트당 평균 비용
        - ![S_1](https://latex.codecogs.com/svg.image?S_1) : M1의 크기
        - ![S_2](https://latex.codecogs.com/svg.image?S_2) : M2의 크기
    - ![C_S](https://latex.codecogs.com/svg.image?C_S) 와 ![C_2](https://latex.codecogs.com/svg.image?C_2) 가 근접하는 것이 바람직하다.
    - ![C_1>>C_2](https://latex.codecogs.com/svg.image?C_1>>C_2) 일때는, ![S_1<<S_2](https://latex.codecogs.com/svg.image?S_1<<S_2) 가 되어야 한다.

<br/>

- **액세스 시간 ($T_s$)**
    - ![T_s=H*T_1+(1-H)*(T_1+T_2)=T_1+(1-H)*T_2](https://latex.codecogs.com/svg.image?T_s=H*T_1+(1-H)*(T_1+T_2)=T_1+(1-H)*T_2)
        - ![T_1](https://latex.codecogs.com/svg.image?T_1) : M1의 액세스 시간
        - ![T_2](https://latex.codecogs.com/svg.image?T_2) : M2의 액세스 시간
        - ![H](https://latex.codecogs.com/svg.image?H) : 적중률(Hit, 참조가 M1에서 발견되는 비율)
    - ![T_S](https://latex.codecogs.com/svg.image?T_S) 와 ![T_1](https://latex.codecogs.com/svg.image?T_1) 가 근접하는 것이 바람직하다.
    - ![T_1<<T_2](https://latex.codecogs.com/svg.image?T_1<<T_2) 일때는 적중률이 1에 가까워야 한다.
    - M1의 최적 사이즈 조건
        - 가격을 낮추기 위해서는 M1의 용량이 작아야 하고, 적중률과 성능을 개선하기 위해서는 용량이 커야 한다.

<br/>

- **액세스 효율**
    - 액세스 효율은 평균 액세스 시간(![T_S](https://latex.codecogs.com/svg.image?T_S))이 M1의 액세스 시간(![T_1](https://latex.codecogs.com/svg.image?T_1))에 얼마나 가까운지를 측정하는 척도이다.
    - ![\frac{T_1}{T_S}=\frac{1}{1+(1-H)\frac{T_2}{T_1}}](https://latex.codecogs.com/svg.image?\frac{T_1}{T_S}=\frac{1}{1+(1-H)\frac{T_2}{T_1}})

<br/>

- **지역성이 적중률에 미치는 영향**
    - 지역성이 강하다면 **상위 단계 기억장치의 크기가 비교적 작아도 높은 적중률**을 얻을 수 있다.
        - 캐시의 크기가 비교적 작더라도 주기억장치의 크기에 상관없이 0.75 이상의 적중률을 얻을 수 있다.
        
        ![Untitled](/assets/img/2021-09-30-ComputerStructure_Memory/Untitled%2041.png)

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>성결대학교 컴퓨터 공학과 최정열 교수님 (2021)</li>
    <li>William Stalling, 『컴퓨터시스템구조론(10판)』</li>
  </ul>
  본 게시글은 위 강의 및 교재를 기반으로 정리한 글입니다.
</div>