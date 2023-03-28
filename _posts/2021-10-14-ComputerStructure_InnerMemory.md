---
category: Comput-Struct
tags: [ComputerStructure]
title: "[Computer Structure] 내부 기억장치"
date:   2021-10-14 22:06:00 
lastmod : 2021-10-14 22:06:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 내부 기억장치

## 기억소자

### 기억소자란?

- 반도체 기억장치의 기본 요소
    - **기억소자 (Memory Cell)**
    - 기억소자는 1bit를 저장한다.

<br/>

### 기억소자의 동작

- **선택 단자**
    - 읽기 또는 쓰기 동작을 할 기억 소자를 선택할 때 사용된다.
- **제어 단자**
    - 수행할 동작이 읽기인지 또는 쓰기인지를 결정할 때 사용된다.
- **Data in**
    - 쓰기 동작의 경우, 기억 소자의 상태를 1, 0으로 세트할 전기 신호가 들어오는 통로이다.
- **Sense**
    - 그 단자가 소자의 상태를 출력하는데 사용한다.

- 쓰기 동작
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2066.png)
    
- 읽기 동작
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2067.png)
    
<br/><br/>

## RAM

### RAM이란?

- **기억장치의 각 단어(word)들은 선으로 연결된 주소지정 회로를 통하여 액세스된다.**
    - 선으로 연결된 주소지정 회로 = Address Bus
- 데이터 읽기 및 쓰기가 가능하다.
- 휘발성
    - 전원 공급이 끊기면 데이터가 날라간다.
- 임시 기억장치로 사용될 수 있다.
- RAM의 종류
    - 동적 RAM (DRAM)
    - 정적 RAM (SRAM)

<br/>

### DRAM

- **DRAM의 특징**
    - 커패시터(capacitor)에 전하를 충전하는 방식으로 데이터를 저장한다.
        - **커패시터에 전하가 존재하는 지의 여부에 따라 2진수(1과 0)이 구분된다.**
    - 데이터의 저장 상태를 유지하기 위해서 주기적으로 **재충전이 필요**하다.
        - 따라서, 재충전 회로가 필요하다.
    - DRAM은 아날로그 장치이다.
        - 커패시터는 일정 범위 내의 어떠한 전하값도 저장할 수 있다.
        - 단지, 특정 기준을 중심으로 1과 0을 나눈다.
    - 가격이 저렴하고 대용량 기억장치를 구성할 수 있다.
        - 따라서, 주기억장치로 사용된다.
        - 동적 기억소자가 정적 기억소자보다 더 간단하고 더 작다.

<br/>

- **DRAM 재충전 동작**
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2068.png)
    
    - 만약 재충전이 없다면?
        - 1을 Write하여 저장해도, 시간이 지나면 0으로 바뀌게 된다.

<br/>

- **DRAM 동작 방식**
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2069.png)
    
    - **주소선**
        - 이 기억소자로부터 비트 값이 읽혀지거나 쓰여질 때 활성화된다.
    - **트랜지스터**
        - 스위치 역할을 한다.
        - 전압이 주소선에 가해지면, 닫힌다. (전류가 흐른다.)
        - 그렇지 않으면 열린다. (전류가 흐르지 못한다.)
    - **Write 연산**
        - 전압이 비트 선에 가해진다.
            - 1 = 전압 High
            - 0 = 전압 Low
        - 주소 선으로 신호가 들어오면, 그 때 전하가 커패시터로 전송된다.
    - **Read 연산**
        - 주소선이 선택되면 트랜지스터가 켜진다.
        - 커패시터에 저장된 전하가 비트 라인을 통해서 감지 증폭기로 보내진다.
        - 읽기 후 커패시터 전하를 복원해야 한다.

<br/>

### SRAM

- **SRAM의 특징**
    - 플립-플롭 논리 게이트를 이용한다.
    - SRAM은 디지털 장치이다.
    - 재충전이 필요없다. (커퍼시터를 사용하지 않는다.)
        - 전력이 공급되는 동안, 재충전없이 데이터를 계속 유지한다.
    - 비트당 크기가 크고 복잡하다.
    - 캐시 기억장치로 사용된다.

<br/>

- **SRAM 동작 방식**
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2070.png)
    
    - **1인 상태**
        - ![O(kn)](https://latex.codecogs.com/svg.image?C_1) = High
        - ![O(kn)](https://latex.codecogs.com/svg.image?C_2) = Low
        - ![O(kn)](https://latex.codecogs.com/svg.image?T_1), ![O(kn)](https://latex.codecogs.com/svg.image?T_4) = Off
        - ![O(kn)](https://latex.codecogs.com/svg.image?T_2), ![O(kn)](https://latex.codecogs.com/svg.image?T_3) = On
    - **0인 상태**
        - ![O(kn)](https://latex.codecogs.com/svg.image?C_2) = High
        - ![O(kn)](https://latex.codecogs.com/svg.image?C_1) = Low
        - ![O(kn)](https://latex.codecogs.com/svg.image?T_2), ![O(kn)](https://latex.codecogs.com/svg.image?T_3) = Off
        - ![O(kn)](https://latex.codecogs.com/svg.image?T_1), ![O(kn)](https://latex.codecogs.com/svg.image?T_4) = On

<br/>

### DRAM vs SRAM

- **DRAM과 SRAM은 모두 휘발성이다.**
    - 비트값을 유지하기 위해서는 기억소자로 전력이 계속 공급되어야 한다.
- **DRAM의 기억소자**
    - 더 간단하고, 더 작다.
    - 밀도가 높다. (단위 면적당 소자수가 더 많다.)
    - 더 싸다.
    - 재충전 회로가 필요하다.
    - 대용량 기억장치에 주로 사용된다.
- **SRAM의 기억소자**
    - 더 빠르다.
    - 더 복잡하다.
    - 캐시 기억장치로 사용된다.

<br/><br/>

## ROM

### ROM이란?

- 비휘발성을 갖는다.
    - 데이터를 기억장치에 유지하기 위한 전원 장치가 필요없다.
- ROM의 내용을 읽기는 가능하지만, 쓰기는 불가능하다.
    - Read Only Memory
- 데이터나 프로그램이 영구 저장되어 있어 액세스 시간이 짧다.
    - 보조기억장치로부터 옮겨올 필요가 없기 때문이다.
- ROM 활용
    - 시스템 초기화 및 진단 프로그램 (BIOS)
    - (CPU)제어 유닛의 마이크로 프로그래밍
    - 빈번히 사용되는 라이브러리 서브루틴
- 단점
    - 데이터를 삽입하는 과정에서 비교적 높은 고정 비용이 든다.
    - 하나의 비트라도 잘못되면 모든 ROM을 버려야 한다.

<br/><br/>

## 반도체 기억장치 유형

### 유형

![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2071.png)

- **Read-only Memory**
    - 공장에서 Write된 상태로 나온다.
- **Programmable ROM**
    - 빈 ROM
    - 전용 장치로 Write 할 수 있다.
    - 역시 한번 Write된 내용은 변경할 수 없다.
- **Erasable PROM**
    - UV light 를 이용하여 지울 수 있다.
- **Electrically Erasable PROM**
    - 전기적으로 지울 수 있다.
- **Flash Memory**
    - USB, SSD 등

<br/><br/>

## 기억장치 모듈 설계

### 전형적인 16Mbit DRAM 설계 (4M*4)

![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2072.png)

<br/>

### 메모리 읽기 동작의 타이밍도: 비동기식

![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2073.png)

- **CPU 주소 중에서 절반만(11비트) DRAM 칩의 주소 입력으로 들어온다.**
    - '행 선택 후 열 선택' 방식으로 사용하므로
- **"메모리 읽기/쓰기를 위한 주소 발행" 및 "RAS/CAS 신호 발생"은 기억장치 제어기가 담당한다.**
    - 기억장치 제어기: "North Bridge 칩셋" or "CPU 내부"로 구현

<br/><br/>

## SDRAM

### SDRAM이란?

- **SDRAM은 클록 신호와 동기화되어 대기 상태 없이 데이터 교환이 가능하다.**
    - **프로세서는 명령어와 주소를 발송하고, SDRAM이 그 요구를 처리하는 동안 프로세서는 다른 업무를 수행한다.**
    - 전통적인 DRAM의 동작 방식 (SDRAM과 비교해보자.)
        - 프로세서는 주소와 제어 신호를 기억장치로 보내어 R/W 데이터를 지정한다.
        - 액세스 시간만큼 지연 후 DRAM에 R/W를 수행한다.
- **SDRAM은 온-칩 병렬성의 기회를 높여주는 이중-뱅크 구조를 가지고 있다.**
    
    > 이후에 다시 상세히 설명한다.
    > 

- **SDRAM은 버스트 모드를 사용한다.**
    - 한 번의 액세스 동작 시, 여러 바이트들을 연속적으로 전송한다.
    - 일련의 데이터 비트들이 첫 번째 비트가 액세스된 후에 클록에 맞춰 출력된다.
    - **즉, 한번 액세스하면 한 개의 바이트를 전송하는 것이 아니라, 뒤에 이어지는 바이트를 연속적으로 클록에 맞춰 전송한다.**

<br/>

### 버스트 읽기 동작 타이밍도: 동기식

![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2074.png)

- 한 번의 액세스로 데이터 4개를 가져왔다.
    - 버스트길이 = 4

<br/>

### DDR SDRAM

- DDR이란?
    - Double Data Rate
- DDR에서 속도를 높이는 방법의 종류
    - 데이터를 클록 사이클당 2번씩 보내는 방법
    - 버스 상에서 더 높은 클록율을 사용하는 방법
    - 선인출 버퍼를 사용하는 방법

<br/>

- **데이터를 클록 사이클당 2번씩 보내는 방법**
    - 클록의 상승/하강 엣지마다 데이터를 보내는 방법이다.
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2075.png)
        
    - 원리
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2076.png)
        
<br/>

- **버스 상에서 더 높은 클록율을 사용하는 방법**
    - 클록율을 증가시켜서 SDRAM의 속도를 향상시킨다.
    - 원리
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2077.png)
        
<br/>

- **선인출 버퍼를 사용하는 방법**
    - **DDR 세대에 따른 선인출 버퍼의 변화**
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2078.png)
        
    - 데이터를 미리 꺼내둘 수 있는 버퍼의 크기가 클수록 성능상의 이점이 있다.

<br/><br/>

## 기억장치 채널

### 기억장치 채널이란?

- 주기억장치와 CPU 캐시 간의 데이터 전달 통로이다.
- 형태와 원리
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2079.png)
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2080.png)
    
    > 메모리 뱅크에 대해서는 아래에서 계속 설명한다.
    > 

<br/>

### 기억장치 뱅크

- 기억장치 뱅크란?
    - 기억장치를 분할해, 독립적으로 액세스할 수 있도록 구성한 논리적인 단위이다.
    - 일반적으로 하나의 메모리 슬롯이라고 생각하면 된다.

<br/>

- **기억장치 인터리빙(Memory Interleaving)**
    - 각 뱅크는 기억장치 읽기/쓰기를 독립적으로 서비스할 수 있다.
    - K개의 뱅크를 가진 시스템은 K개의 요구를 동시에 서비스할 수 있으므로, 기억장치 쓰기/읽기 속도를 K배만큼 증가시킬 수 있다.
    - **"기억장치의 연속적인 단어들"이 서로 다른 뱅크에 저장되어 있다면, 기억장치의 블록 전송은 속도가 높아진다.**

<br/><br/>

## 기억장치 랭크

- 기억장치 랭크란?
    - 데이터 입출력 폭이 **64비트**가 되도록 구성한 기억장치 모듈이다.
        - 64비트: 메모리 산업표준그룹에서 제정된 표준 기억장치 랭크
    - 다수의 기억장치 칩들을 병렬로 구성하여 하나의 랭크를 만든다.
        - x4 조직의 SDRAM 사용시(4비트) ⇒ 16개의 칩 병렬 접속
        - x8 조직의 SDRAM 사용시(8비트) ⇒ 8개의 칩 병렬 접속

<br/>

- 기억장치 랭크 시각화
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2081.png)
    
    - **SDRAM 모듈**: SDRAM 칩이 모여있는 모듈
    - **SDRAM 칩**: 실제 메모리 역할을 하는 칩

<br/>

- SDRAM 모듈의 종류
    - **SIMM (Single In-line Memory Module)**
        - 단면 모듈
    - **DIMM (Double In-line Memory Module)**
        - 양면 모듈

<br/>

### 단면 모듈

- **1Rx8**
    - x8 조직의 SDRAM 8개를 병렬접속하여 하나의 랭크를 구성한다.

<br/>

### 양면 모듈

- **2Rx8**
    - x8 조직의 SDRAM들을 각 면에 8개씩 병렬접속하여 두 개의 랭크로 구성된 기억장치 모듈

- **여러 종류의 모듈들**
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2082.png)
    
<br/><br/>

## 플래시 메모리

### 플래시 메모리란?

- 플래시 메모리는 전기적으로 데이터를 지우고 다시 기록할 수 있는 비휘발성 기억장치이다.
- 플래시 메모리는 내부 및 외부 기억장치로 사용할 수 있다.

<br/>

- 플래시 메모리의 특징
    - 작고, 가볍고, 기계적인 충격에 강하다.
    - 기록 횟수에 제한이 있으며, 블록을 지운 뒤에 쓸 수 있다.

<br/>

- 플래시 메모리 종류
    - NOR형
    - NAND형

<br/>

- 플래시 메모리 동작
    - **부동 게이트 (Floating Gate)**
        - 부동 게이트를 통해, 정보를 저장한다.
    
    - 기호
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2083.png)
        
    
    - 플래시 메모리의 내부 구조
        
        ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2084.png)
        
<br/>

### 플래시 메모리의 동작 원리

- 플래시 메모리의 메모리 셀
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2085.png)
    
    ![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2086.png)
    

- **쓰기 동작**
    - 제어 게이트로 고전압을 인가하면, N 채널의 전자가 부동 게이트로 유입된다.
    - 부동 게이트에 전자가 충분히 존재하는 상태 ⇒ '0' 을 나타낸다.
- **삭제 동작**
    - p 층에 고전압을 인가하면 부동 게이트에 갇힌 전자가 p 채널로 빠져나온다.
- **읽기 동작**
    - 드레인 및 게이트에 전압을 인가하면 읽는다.

<br/>

### NOR 형

- **메모리 셀 배열이 트랜지스터들의 병렬로 접속된다.**
    - 셀 단위로 액세스가 가능하다.
- 바이트/워드 단위로 읽기와 쓰기가 가능하다.
    - NAND 형에 비해서 읽기 속도가 빠르다.
- 덮어쓰기/지우기는 NAND 형에 비해서 속도가 느리다.
    - 왜냐하면 랜덤 액세스가 불가능하기 때문이다.
- 낮은 저장 밀도를 갖는다.
- 휴대 전화 운영체제, 윈도우의 BIOS 프로그램을 저장하는데 사용된다.

![Untitled](/assets/img/2021-10-14-ComputerStructure_InnerMemory/Untitled%2087.png)

<br/>

### NAND 형

- **메모리 셀 배열이 트랜지스터들의 직렬로 접속된다.**
    - 메모리 셀 단위 액세스가 불가능하다.
- 블록들로 구성되며, 각 블록은 페이지들(2KB, 4KB 등)로 구성된다.
- 읽기/쓰기는 페이지 단위로, 삭제는 블록 단위로 이루어진다.
- NOR 형보다 읽기 속도는 느리지만, 쓰기 속도는 더 빠르다.
- NOR 형보다 셀당 면적이 작아서, 대용량 저장장치에 유리하다.
- SSD, USB 플래시 드라이브, 메모리 카드에 사용된다.

<br/>

### SLC, MLC, TLC

- **SLC, MLC, TLC 는 부동 게이트로 들어가는 전자들의 수를 조정하여 각 셀에 저장되는 상태의 수를 증가시킨 것이다.**
    - 즉, 제어 게이트의 전압을 세부적으로 구분한 것이다.
- **SLC**
    - Single-Level Cell
    - 두 가지 상태를 가짐으로써 한 비트를 저장하는 셀이다.
- **MLC**
    - Multi-Level Cell
    - 셀의 상태를 4가지로 구분할 수 있다.
    - 00, 01, 10, 11
    - 메모리 셀 당 2비트씩 저장된다.
- **TLC**
    - Triple-Level Cell
    - 셀의 상태를 8가지로 구분할 수 있다.
    - 000, 001, 010, 011, 100, 101, 110, 111
    - 메모리 셀당 3비트씩 저장된다.

- **MLC과 TLC의 문제점**
    - 전자 수 조정을 위한 세밀한 작업이 필요하다.
        - 전자의 개수를 통해, 셀의 상태를 구분하므로
    - 데이터 구분의 어려움으로 인한 액세스 속도 저하
    - 오류 발생 빈도 증가
    - 수명 단축

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