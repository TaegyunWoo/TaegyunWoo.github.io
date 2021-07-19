---
category: Algorithm
tags: [알고리즘, 분할정복, 퀵정렬]
title: "[알고리즘 - 분할정복] 퀵정렬"
date:   2021-07-16 15:00:00 
lastmod : 2021-07-19 15:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 분할 정복 : 퀵 정렬

## 개요

### 특징

- 분할 정복 알고리즘으로 분류되지만, 정복 후 분할하는 알고리즘임
- 각 부분문제의 크기가 일정하지 않다
- 거대한 입력 크기에 대해 가장 좋은 성능을 가짐
- 피봇 : 기준 원소

<br>

### 예시

![예시](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_25.png)

<br><br>

## 알고리즘

### 의사코드

![의사 코드](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_26.png)

> 별도의 병합과정 필요X

<br>

### 2번 절차

![2번 절차 - 1](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_27.png)

![2번 절차 - 2](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_28.png)

<br>

### 상세 원리

![상세 원리](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_29.png)

> 피봇은 i가 끝까지 도달한 뒤,  
j가 가르키는 요소와 위치를 변경한다.

1. A[i]번째 요소 검사 (피봇보다 작은지)
2. 작다면 : j += 1  
크다면 : i+=1 한 뒤, 처음으로 돌아가기
3. A[j]와 A[i] 위치 변환
4. i+=1
5. i가 끝까지 갈때까지 반복

<br><br>

## 시간 복잡도

### 특징

- 퀵 정렬의 성능 : 피봇 선택이 좌우
- 피봇으로 가장 작은 숫자 or 가장 큰 숫자가 선택되면  
⇒ 한 부분으로 치우치는 분할을 야기함

<br>

### 최악 경우 시간복잡도

- 계속해서 "가장 큰 수" or "가장 낮은 수"를 선택한다면

- $O(n^2)$

<br>

### 평균, 최선 경우 시간복잡도

- 피봇을 항상 랜덤하게 선택한다고 가정시

- ![O(nlog_2n)](https://latex.codecogs.com/svg.image?O(nlog_2n)) = O(nlogn)$

<br>

### 피봇 선정 방법

1. 랜덤 선정
2. "가장 왼쪽 숫자" & "중간 숫자" & "가장 오른쪽 숫자" 중 중앙값으로 피봇 선정

![피봇 선정 방법](/assets/img/2021-07-16-ALGORITHM_QuickSort/Untitled_30.png)

<br><br>

## 퀵 정렬 성능 향상 방법

### 방법
- 삽입 정렬 동시 사용

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>성결대학교 컴퓨터 공학과 임태수 교수님 (2021)</li>
    <li>양성봉, 『알기 쉬운 알고리즘』</li>
  </ul>
  본 게시글은 위 강의 및 교재를 기반으로 정리한 글입니다.
</div>