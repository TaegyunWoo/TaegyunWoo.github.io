---
category: Comput-Struct
tags: [ComputerStructure]
title: "[Computer Structure] CPU 제어유닛"
date:   2021-12-02 19:00:00 
lastmod : 2021-12-02 19:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 제어 유닛

## 개요

### CPU 구조

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled.png)

<br/>

### 프로세서의 기능 정의

- 명령어 세트
    - 연산 코드
    - 주소지정 방식
    - 레지스터
- 시스템 버스
    - I/O 모듈 인터페이스
    - 기억장치 모듈 인터페이스
- 시스템 버스와 운영체제
    - 인터럽트 처리 조직

<br/>

### 제어 유닛

- 제어 유닛이란?
    - 프로세서 내의 여러가지 요소들을 제어한다.
    - 컴퓨터 전체를 작동시키는 엔진이다.
- **프로세서의 한 부분으로서 실제로 어떤 일이 일어나게끔 한다.**
    - 실행될 프로그램에 의해 결정되는 순서대로 **마이크로 연산**을 실행한다.
        - 마이크로 연산: 기계 명령어가 실행되는 과정 (자세한 것은 조금있다 다룬다.)
    - 제어유닛은 각 마이크로 연산을 실행시키기 위한 **제어신호**를 발생시킨다.
- 제어 유닛이 생성하는 제어신호의 종류
    - **프로세서 외부로의 제어신호**
        - 기억장치 및 I/O 모듈과의 데이터 교환
    - **프로세서 내부로의 제어신호**
        - 레지스터들 간에 데이터 이동
        - ALU 연산
        - 내부 연산 통제
- 제어 유닛 구현 방법
    - **하드 와이어**
        - H/W 기반 제어 방식
    - **마이크로 프로그래밍**
        - S/W 기반 제어 방식
        
        > 우리는 마이크로 프로그래밍을 위주로 알아볼 것이다.
        > 

<br/><br/>

## 마이크로 연산

### 마이크로 연산?

- 프로그램
    - 일련의 **명령어 사이클**로 구성되어 실행된다.
- 명령어 사이클
    - 다수의 더 작은 단위(**서브 사이클**)들로 구성된다.
- 서브 사이클
    - 프로세서 레지스터를 포함하고 있는 일련의 단위로 구성된다.
    - 이것을 **마이크로 연산**이라고 한다.
    - 가장 세분화된 프로세서의 기능적 or 원자 연산이다.

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%201.png)

<br/>

### 명령어 인출 과정

1. PC에 저장된 주소를 MAR(기억장치 주소 레지스터)로 옮긴다.
    - MAR이 시스템 버스의 주소 버스에 연결된 유일한 레지스터이다.
2. 명령어를 가져온다.
    1. (MAR 내에 있는) 원하는 주소가 주소 버스로 내보내지고, 제어 유닛은 제어 버스로 READ 명령을 보낸다.
    2. 그 결과로, 기억장치로부터 읽혀진 명령어는 데이터 버스를 통하여 MBR(기억장치 버퍼 레지스터)에 저장된다.
    3. 다음 명령어의 실행 준비를 위해 PC가 1 증가한다.
3. MBR의 내용을 IR(명령어 레지스터)로 이동시킨다.
    - 간접 사이클이 수행될 경우를 위해 MBR을 비워두기 위함이다.
    
    > 간접 사이클에 대한 내용은 [이전 게시글](https://taegyunwoo.github.io/comput-struct/ComputerStructure_ProcessorGroupAndFunction#14)을 참고하자.
    > 

<br/>

### 마이크로 연산: 명령어 인출 사이클

- 명령어 인출의 마이크로 연산 절차
    - t1
        - **MAR ← (PC)**
    - t2
        - **MBR ← memory**
        - **PC ← (PC) + I**
        
        > I : 명령어의 길이
        > 
    - t3
        - **IR ← (MBR)**
- 한 클록 사이클
    - t1, t2, t3 은 각각 하나의 클록 사이클 시간을 나타낸다.
    - t1 : 첫번째 클록 사이클 시간
    - t2 : 두번째 클록 사이클 시간
    - t3 : 세번째 클록 사이클 시간
- 인출 사이클 구성
    - 3단계의 클록(t)
    - 4개의 마이크로 연산
- 시각화
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%202.png)
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%203.png)
    
<br/>

### 마이크로 연산: 간접 사이클

- 명령어가 인출되면, 제어 유닛은 **간접 주소지정 방식을 사용한 오퍼랜드**가 있는지 확인한다.
    - 명령어가 간접 주소지정 방식을 사용할 때
        - **간접 사이클이 실행 사이클보다 먼저 수행되어야 한다.**
- 간접 사이클의 마이크로 연산 절차
    - t1
        - **MAR ← (IR(address))**
            - address : 유효 주소가 담긴 곳의 주소
            
            > 유효 주소에 대한 내용은 [이전 게시글](https://taegyunwoo.github.io/comput-struct/ComputerStructure_InstructionSet#33)을 참고하자.
            > 
    - t2
        - **MBR ← memory**
            - memory에서 유효 주소값을 가져온다.
    - t3
        - **IR(address) ← (MBR(address))**
            - address : 유효주소값
- 상세 설명
    - **IR의 주소 필드가 MBR로부터 들어오는 주소로 갱신된다.**
        - 이때 들어오는 주소는 직접주소(유효주소)이다.
    - **그러면 IR은 이제 간접 주소 지정방식이 사용되지 않은 상태가 된다.**
        - 실제로 원하는 명령어의 주소가 IR에 담기기 때문이다.
    - **간접 사이클을 마치고나면, 최종적으로 실행 사이클을 위한 준비가 완료된다.**

<br/>

### 마이크로 연산: 인터럽트 사이클

- 인터럽트 사이클이란?
    - 인터럽트가 발생했을 때, 수행되는 사이클이다.
- 인터럽트 사이클의 마이크로 연산 절차
    - t1
        - **MBR ← (PC)**
            - 복귀할 주소값을 백업할 준비를 한다.
            - 참고로, **주소값을 데이터처럼 처리**하기 때문에 MBR에 넘긴다.
    - t2
        - **MAR ← save-address**
            - save-address: 복귀할 주소가 저장된 곳의 주소
        - **PC ← routine-address**
            - routine-address: ISR의 시작주소
    - t3
        - **memory ← (MBR)**
            - MBR에 담긴 복귀 주소값을 메모리에 저장한다.
- 시각화
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%204.png)
    
- 상세 설명
    - PC의 내용이 MBR로 보내져, 인터럽트 수행이 끝난 후에 복귀할 때 사용할 수 있게끔 한다.
    - MAR에는 PC의 내용이 저장될 위치의 주소가 적재된다. 그리고 PC에는 인터럽트 처리 루틴의 시작 주소가 적재된다.
    - PC의 이전 값을 가지고 있는 MBR의 내용을 기억장치에 저장한다.

<br/>

### 마이크로 연산: 실행 사이클 - ADD

- 인출·간접·인터럽트 사이클과 실행 사이클의 차이점
    - 인출·간접·인터럽트 사이클은 매번 동일한 마이크로 연산이 반복된다.
    - 실행 사이클은 N개의 서로 다른 연산코드(실행 명령어)를 위해, N가지 마이크로 연산들이 존재한다.
        - 즉, 실행 사이클은 어떤 실행인지에 따라 **마이크로 연산들이 다양하게 사용**된다.
- `ADD R1, X` 의 마이크로 연산 절차
    - `ADD R1, X` 의 의미: R1(레지스터1)의 값과 X값을 더하고, R1에 결과를 저장한다.
    - t1
        - **MAR ← (IR(address))**
            - address: 연산할 X의 주소
    - t2
        - **MBR ← memory**
            - X에 저장된 값을 MBR에 전달한다.
    - t3
        - **R1 ← (R1) + (MBR)**
            - ALU를 통해, 값을 계산한 뒤 R1에 저장한다.
- 상세설명
    - 만약 레지스터(R1)가 아닌 다른 메모리 공간에 결과를 저장한다면, 결과 저장 위치 계산 등의 과정이 추가적으로 필요해진다.

<br/>

### 마이크로 연산: 실행 사이클 - ISZ

- `ISZ X` 의 마이크로 연산 절차
    - `ISZ X` 의 의미: X의 값을 1 증가시키고, 결과가 0이면 바로 다음 명령어를 스킵한다.
    - t1
        - **MAR ← (IR(address))**
            - address: 1 증가시킬 값이 저장된 X의 주소
    - t2
        - **MBR ← memory**
            - X에 저장된 값을 MBR에 전달한다.
    - t3
        - **MBR ← (MBR) + 1**
    - t4
        - **memory ← (MBR)**
        - **if (MBR) == 0 then PC ← (PC) + I**
            - (PC) : 다음에 수행할 명령어의 주소가 담겨있다.
            - I : 명령어 하나의 길이

<br/>

### 명령어 사이클 정리

- 명령어 사이클의 각 단계는 마이크로 연산으로 구성된다.
    - 인출, 간접, 인터럽트, 실행 사이클
- **명령어 사이클 코드 (Instruction Cycle Code: ICC)**
    - ICC를 이용하여 프로세서의 실행 위치를 확인한다.
    - 즉 ICC를 통해, 제어 유닛이 어떤 제어를 해야하는지 판단한다.
    - ICC=00 : 인출
    - ICC=01 : 간접
    - ICC=10 : 실행
    - ICC=11 : 인터럽트
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%205.png)
    
<br/><br/>

## 프로세서의 제어

### 제어 유닛의 특성을 결정하는 요소

아래 요소들이 제어 유닛의 특성을 결정한다.

- **프로세서의 기본 구성요소**
    - ALU, 레지스터, 내부/외부 데이터 통로, 제어 유닛
- **프로세서가 실행할 마이크로 연산**
- **마이크로 연산이 실행되기 위해 제어 유닛이 수행해야 할 기능**

<br/>

### 제어 유닛의 기능

- **순서 제어**
    - 실행될 프로그램에 근거하여, 프로세서가 일련의 마이크로 연산을 적절한 순서대로 처리하도록 해준다.
- **실행**
    - 각 마이크로 연산이 실행되도록 해준다.

**제어신호에 의해서 '순서제어'와 '실행'이 처리된다.**

<br/>

### 제어 신호

- 제어 유닛이 기능을 수행하려면 아래 항목들이 필요하다.
    - **입력**
        - 시스템의 상태를 결정해주는 것
    - **출력**
        - 시스템 동작을 제어하는 것
- 제어 신호의 종류
    - 제어유닛 input 신호
    - 제어유닛 output 신호
- **매 클록마다 제어 유닛은 모든 입력을 읽어들이고 제어신호를 발생시킨다.**

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%206.png)

하나씩 자세히 알아보자.

<br/>

### 입력 제어 신호

- **클록**
    - 제어 유닛이 시간을 지키도록 하는데 사용된다.
    - t1, t2, ... 을 구분한다.
    - 프로세서 사이클 시간 or 클록 사이클 시간
- **명령어 레지스터**
    - Opcode에 따라, 어떤 마이크로 연산을 실행할지 결정한다.
- **플래그**
    - 프로세서의 상태와 이전 ALU 연산의 결과를 검사한 결과를 나타낸다.
- **제어 버스로부토 들어오는 제어 신호들**
    - 인터럽트 신호와 확인 신호 등

<br/>

### 출력 제어 신호

- **프로세서 내의 제어 신호**
    - 레지스터들 간의 데이터 전송을 발생시키는 신호이다.
    - 특정 ALU 기능들을 활성화시키는 신호들이다.
- **제어 버스로 나가는 제어 신호**
    - 기억장치로 보내지는 제어 신호나
    - I/O 모듈로 보내지는 제어 신호가 있다.

<br/>

### 마이크로 연산에 필요한 제어 신호들

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%207.png)

<br/>

### 제어 신호 예시

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%208.png)

<br/><br/>

## 마이크로프로그램 방식의 제어 유닛

### 마이크로프로그램 방식이란?

- 제어 기억장치에 저장된 마이크로 프로그램을 실행하는 방식으로 제어 유닛을 동작시키는 것이다.
- 마이크로프로그램?
    - 마이크로 명령어들의 집합
- 마이크로 명령어?
    - 명령어 사이클에서 '각 주기(t1, t2, ...)에서 실행되는 각 마이크로-연산'을 지정해주는 2진 비트들
    - 마이크로 명령어 == 제어 단어
    - 마이크로명령어의 실행 결과로 제어 신호를 발생시킨다.
- 마이크로프로그램 방식을 사용하면, 명령어 설계가 바뀌어도 제어 메모리에 저장된 프로그램만 변경하면 된다.

<br/>

### 제어 유닛의 구성

- **명령어 해독기 (Decoder)**
    - 명령어 레지스터(IR)로부터 들어오는 명령어의 연산코드를 해독하여, 해당 연산을 수행하기 위한 루틴의 시작 주소를 계산한다.
- **제어 주소 레지스터 (Control Address Register, CAR)**
    - 다음에 실행할 마이크로명령어의 주소를 저장한다.
    
    > Program Counter와 유사하다.
    > 
- **제어 기억장치 (Control Memory)**
    - 마이크로 프로그램이 저장되는 공간이다.
- **제어 버퍼 레지스터 (Control Buffer Register, CBR)**
    - 제어 기억장치로부터 읽혀진 마이크로명령어를 일시 저장한다.
- **순서 제어 유닛 (Sequencing Logic)**
    - 마이크로명령어의 실행순서를 결정한다.
    - 제어 주소 레지스터에 주소를 적재한다.

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%209.png)

<br/>

### 제어 기억장치

- 제어 단어 (or 마이크로 명령어)들이 저장되어 있다.
- 각 루틴 내의 마이크로명령어들은 순차적으로 진행된다.
- 예시
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2010.png)
    
<br/>

### 제어 유닛 동작 순서

1. 명령어 레지스터에 들어온 명령어(Main Memory에 저장된 기계명령어)를 해독해, 연산코드(opcode)를 구한다.
2. 연산코드에서 해당 명령에 필요한 **제어 서브루틴**의 시작주소를 계산한다.
3. 명령어를 실행하기 위해, 순서제어 논리 유닛(Sequencing Logic)이 제어 기억장치로 READ 명령을 보낸다.
4. 제어 주소 레지스터(CAR)에 명시된 제어 기억장치의 주소에 저장되어 있는 단어가 읽혀져서 제어 버퍼 레지스터(CBR)로 보내진다.
5. CBR의 내용에 따라, 제어 신호들과 다음 주소 정보가 발생된다.
6. 순서제어 유닛은 CBR과 ALU 플래그에 근거하여 새로운 주소를 CAR에 적재한다.

<br/>

### 마이크로프로그램 제어 방식

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2011.png)

<br/>

### 제어 서브루틴의 시작 주소 계산

- 명령어 해독기(Decoder)는 연산코드를 이용하여 **제어 기억장치 내 해당 실행 사이클 루틴의 시작주소** 를 찾는다.
- 시작 주소 해독 방법
    - 명령어의 연산 코드를 특정 비트 패턴과 혼합시킴으로써, 그 연산의 수행에 필요한 실행 사이클 루틴의 시작 주소를 찾아낸다.
    - 예시
        
        ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2012.png)
        
        - OPCODE 앞에 '1'을 추가하고, 뒤에 '00'을 추가한다. ⇒ 이것이 해당 명령어가 포함된 루틴의 시작 주소이다.
        - 연산 코드 = 0001 ⇒ 시작주소 = 1000100
        - 연산 코드 = 0100 ⇒ 시작주소 = 1010000

<br/>

### 마이크로명령어의 형식 예시

- **연산 필드가 두 개이면, 두 개의 마이크로-연산들을 동시에 수행할 수 있다.**
- **조건 필드 (CD)**
    - 분기에 사용될 조건 플래그를 지정한다.
- **분기 필드 (BR)**
    - 분기의 종류와 다음에 실행할 마이크로명령어의 주소를 결정하는 방법을 명시한다.
- **주소 필드 (ADF)**
    - 분기가 발생하는 경우: 목적지 마이크로명령어의 주소
    - 분기가 발생하지 않는 경우: 다음에 실행할 마이크로명령어의 주소

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2013.png)

> 위 경우, 연산필드가 2개이다.
> 

- 마이크로명령어 코드 예시
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2014.png)
    

- 조건 필드 예시
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2015.png)
    

- 분기 필드 예시
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2016.png)
    
<br/>

### 마이크로프로그래밍 예시

- **인출 사이클 루틴**
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2017.png)
    
    - 변환된 마이크로명령어 형태
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2018.png)
    

<br/>

- **간접 사이클 루틴**
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2019.png)
    
    - 변환된 마이크로명령어 형태
        
        ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2020.png)
        
<br/>

### 실행 사이클 루틴

- 사상 방식을 이용하여, 각 연산 코드에 대한 실행 사이클 루틴의 시작 주소를 결정하고, 각 명령어 실행을 위한 루틴을 작성한다.
- 각 연산 코드에 대한 사상 결과
    
    ![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2021.png)
    
<br/><br/>

## 마이크로 명령어 구조

### 수직적 마이크로명령어

- 코드화된 비트들을 이용해, 마이크로명령어의 각 기능 코드를 구성하는 구조이다.
- 해독기(Decoder)를 이용해, 그 코드를 필요한 수만큼의 제어 신호로 확장한다.
    - n개 비트의 연산 코드 ⇒ 총 ![math](https://latex.codecogs.com/svg.image?2^n) 개의 제어 신호를 생성할 수 있다.
- **수행될 각 동작에 대해, 하나의 코드를 사용한다.**
- 특징
    - 제어 기억장치의 용량 절감
        - 코드화되어있으므로
    - 약간의 논리회로와 시간 지연 추가
    - 수평적 마이크로명령어보다 간결
    - 병렬 처리에 제약

![Untitled](/assets/img/2021-12-02-ComputerStructure_ControlUnit/Untitled%2022.png)

<br/>

### 수평적 마이크로명령어

- 마이크로명령어의 각 필드의 비트가 제어 신호에 대응된다.
- 마이크로명령어는 병렬로 수행될 수 있는 다수의 마이크로-연산을 지정한다.
- '프로세서 내부의 각 제어선'과 '시스템 버스의 각 제어선'에 대해 한 비트씩 할당한다.
    - 비트값이 1인 모든 제어선을 on 시키고, 0인 제어선을 off시킨다.
    - 조건 비트에 따라, 마이크로명령어의 순서를 제어한다.
- 특징
    - 하드웨어가 간단하고, 해독에 따른 지연이 없다.
    - 마이크로명령어 비트수가 길기 때문에, 큰 용량의 제어 기억장치가 필요하다.
    - 병렬 처리가 가능하다.

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