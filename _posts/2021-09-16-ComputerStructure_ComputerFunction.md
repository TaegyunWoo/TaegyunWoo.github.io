---
category: Comput-Struct
tags: [ComputerStructure, 컴퓨터기능, 인터럽트]
title: "[Computer Structure] 컴퓨터 기능"
date:   2021-09-16 16:00:00 
lastmod : 2021-09-16 16:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 컴퓨터 기능

## 컴퓨터 구성 요소

### 컴퓨터 구성 요소와 상호 작용

![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2011.png)

- Register
    - **PC**
        - Program Counter
        - 다음에 실행할 명령어의 주소를 저장하는 공간이다.
    - **IR**
        - Instruction Register
        - 현재 실행 중인 명령어 자체를 저장하는 공간이다.
    - **MAR**
        - Memory Address Register
        - PC를 통해, 다음 명령어의 주소를 저장하는 공간이다.
    - **MBR**
        - Memory Buffer Register
        - MAR로 가져온 명령어 자체를 저장하는 공간이다.
    - **I/O AR**
        - Input/Output Address Register
        - 입출력 주소를 저장하는 공간이다.
    - **I/O BR**
        - Input/Output Buffer Register
        - 입출력 버퍼이다.

<br/>

- **명령어 가져오는 과정**

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2012.png)

> 시스템 버스 구분: '주소버스', '데이터버스', '제어버스'  
자세한 것은 나중에 설명한다.

<br/><br/>

## 컴퓨터의 기능

### 컴퓨터의 기본 기능

- 프로그램 실행 기능
    - 프로그램은 기억장치에 저장된 기계명령어들로 구성되어 있다.

<br/>

### 명령어 처리

- 명령어 사이클
    - **인출 (Fetch)**: 명령어 가져오기
    - **실행 (Execute)**: 명령어 실행하기

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2013.png)

<br/>

### 명령어 인출과 실행 과정

1. **프로세서가 기억장치로부터 명령어를 인출한다.**
    - PC(프로그램 카운터)가 다음에 인출할 명령어의 주소를 가지고 있다.
    - 프로세서가 명령어를 인출한 뒤에, PC의 내용을 증가시켜 다음 명령어를 인출할 준비를 한다.
2. **인출된 명령어를 프로세서 내부 레지스터에 적재한다.**
    - 내부 레지스터 == 명령어 레지스터 (IR: Instruction Register)
3. **프로세서는 명령어를 해석하고, 그 결과에 따라 필요한 동작을 수행한다.**

<br/>

### 프로그램 실행 예시

- **명령어 형식**

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2014.png)

    - **opcode** : 명령어
    - **address** : 연산될 데이터 주소
    - **operand** : 연산될 데이터

<br/>

- **프로그램 실행 예시**
    - 주어진 연산 코드
        - 아래와 같이 연산 코드에 대한 명령어가 주어졌다고 가정하자.
        - **'0001'** or **'1'**
            - 메모리에서 데이터를 가져와 AC에 로드한다.
        - **'0101'** or **'5'**
            - 메모리에서 데이터를 가져와 AC와 더하기 연산을 한다.
    - **실행 순서**
        1. Step 1

            ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2015.png)

            > AC : Accumulator (임시저장공간)

        2. Step 2

            ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2016.png)

        3. Step 3

            ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2017.png)

        4. Step 4

            ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2018.png)

        5. Step 5
            - 생략..
        6. Step 6
            - 생략..

<br/>

### 명령어 상태도

![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2019.png)

- **Multiple Operands**
    - 명령어에 여러 operand의 주소가 있을 수 있다.
    - 이것을 처리하는 것을 의미한다.

<br/><br/>

## 인터럽트

### 인터럽트란?

- 프로세서를 일시적으로 방해하는 것이다.
- 인터럽트를 통해 프로세서의 처리 효율을 향상시킬 수 있다.

<br/>

### 인터럽트의 효용성

- 인터럽트가 없는 경우

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2020.png)

- 인터럽트가 있는 경우

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2021.png)

<br/>

### 인터럽트에 의한 제어권 이동 순서

1. 현재 수행 중인 프로그램 실행을 중단하고, **그 문맥(context)** 를 저장한다.
    - 다음 명령어의 주소(Program Counter의 현재 내용) 와 현재 동작과 관련된 데이터를 저장한다.
2. 프로그램 카운터(PC)에 인터럽트 처리기 루틴(ISR)의 시작주소를 세팅한다.
    - PC에는 항상 다음 명령어의 주소가 저장된다.
    - 인터럽트 처리 명령어가 다음에 수행되어야한다.
    - 따라서, PC에 인터럽트 처리 명령어의 주소(ISR의 시작주소)를 세팅한다.
3. 인터럽트를 처리한다.
4. 문맥을 복원하고 인터럽트된 기존 프로그램을 다시 수행한다.

![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2022.png)

<br/>

### 인터럽트가 포함된 명령어 사이클 상태도

![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2023.png)

<br/>

### 다중 인터럽트

- 인터럽트 처리 중 다른 인터럽트가 발생하면 어떻게 처리할까?
- **순차적 인터럽트 처리**
    - 인터럽트를 처리하고 있는 중에는 추가적인 인터럽트가 불가능하다.
    - 나중에 발생한 인터럽트는 대기하다가, 프로세서가 인터럽트 가능 상태로 변경되면 처리된다.
    - 장점
        - 간단 명료하다
        - 인터럽트들이 순차적으로 처리된다.
    - 단점
        - 상대적 우선순위 또는 시간적 위급성을 고려하지 않는다.

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2024.png)

<br/>

- **우선 순위에 따른 인터럽트 처리**
    - 인터럽트의 우선 순위를 정하고, 그 순위가 높은 인터럽트를 먼저 처리한다.

    ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2025.png)

    - 예시

        ![Untitled](/assets/img/2021-09-16-ComputerStructure_ComputerFunction/Untitled%2026.png)

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