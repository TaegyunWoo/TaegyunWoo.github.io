---
category: Algorithm
tags: [알고리즘, 다이나믹프로그래밍, 연속행렬곱셈]
title: "[알고리즘 - 동적계획] 연속 행렬 곱셈"
date:   2021-07-31 13:30:00 
lastmod : 2021-07-31 13:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 동적 계획 : 연속 행렬 곱셈

## 개요

### 연속 행렬 곱셈이란?

- 연속된 행렬들의 곱셈에 필요한 원소간의 최소 곱셈 횟수를 찾는 문제

<br>

### 행렬곱이란

![행렬곱](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2057.png)

<br>

### 곱셈횟수

![곱셈횟수](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2058.png)

<br>

### 연속된 행렬

![연속된 행렬](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2059.png)

<br>

### 문제배경

- 연속된 행렬이 3개 이상일때, 결합법칙이 성립된다.
- 행렬들의 곱셈 순서에 따라, 최종 곱셈 횟수가 달라진다.

- ex) ![A*B*C](https://latex.codecogs.com/svg.image?A*B*C) 일때
  - ![A*B*C](https://latex.codecogs.com/svg.image?(A*B)*C)
  - ![A*B*C](https://latex.codecogs.com/svg.image?A*(B*C))
  - 위와 같이 2가지의 경우의 수가 가능하다.  
  - 단, ![B*(A*C)](https://latex.codecogs.com/svg.image?B*(A*C)) 와 같이 행렬의 순서 자체를 바꾸는 것은 불가능하다.

<br><br>

## 알고리즘

### 최적화 원리

- 최적의 곱셈 순서를 ![A_1*((((A_2*A_3)*A_4)*A_5)*A_6)](https://latex.codecogs.com/svg.image?A_1*((((A_2*A_3)*A_4)*A_5)*A_6)) 이라고 가정하자.
- 이때, ![(A_2*A_3)*A_4](https://latex.codecogs.com/svg.image?(A_2*A_3)*A_4) 은  반드시 연속된 행렬 ![A_2,A_3,A_4](https://latex.codecogs.com/svg.image?A_2,A_3,A_4) 의 최적의 곱셈 순서이다.
- 즉, 작은 문제의 해가 큰 문제의 해에 포함된다. ⇒ 다이나믹 프로그래밍

<br>

### 의사코드

![의사코드](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2060.png)

- 행렬
  - ![A_1,A_2,A_3,...,A_n](https://latex.codecogs.com/svg.image?A_1,A_2,A_3,...,A_n)
- C[i][j]
  - ![C[i][j]=A_i*A_2*A_3*...*A_j](https://latex.codecogs.com/svg.image?C[i][j]=A_i*A_2*A_3*...*A_j) 까지의 최소 곱셈 횟수
- C[i][j] 구하는 식
  - ![C[i][j]=C[i][k]+C[k+1][j]+(d_{i-1}*d_k*d_j)](https://latex.codecogs.com/svg.image?C[i][j]=C[i][k]+C[k+1][j]+(d_{i-1}*d_k*d_j))

- 참고
  - k는 C[i][j]를 구하기 위해 연속된 행렬들이 2개로 나눠지는데, k가 바로 이때의 기준 행렬의 번호이다.

<br>

### C[i][j] 계산 방법

- C[1][2] 일때 (행렬의 개수가 2개일 때)
  - ![C[1][2]=d_0*d_1*d_2](https://latex.codecogs.com/svg.image?C[1][2]=d_0*d_1*d_2)

    ![C[1][2]](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2061.png)

- C[1][3] 일때 (행렬의 개수가 3개일 때)
  - ![C[1][3]=min[A_1(A_2A_3),(A_1A_2)A_3]](https://latex.codecogs.com/svg.image?C[1][3]=min[A_1(A_2A_3),(A_1A_2)A_3])

  - ![(A_1A_2)A_3](https://latex.codecogs.com/svg.image?(A_1A_2)A_3)
  의 최소곱셈개수 =
  ![(d_0*d_1*d_2)+(d_0*d_2*d_3)](https://latex.codecogs.com/svg.image?(d_0*d_1*d_2)+(d_0*d_2*d_3))
  
    ![C[1][3]: 1](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2062.png)  
  <br>
  - ![A_1(A_2A_3)](https://latex.codecogs.com/svg.image?A_1(A_2A_3))
  의 최소곱셈개수 =
  ![(d_0*d_1*d_3)+(d_1*d_2*d_3)](https://latex.codecogs.com/svg.image?(d_0*d_1*d_3)+(d_1*d_2*d_3))

    ![C[1][3]: 2](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2063.png)

<br>

### 동작 예시

![동작 예시](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2064.png)

<br>

### 절차 : 1~2번 라인

- 배열의 대각선 원소들인 ![C[1][1],C[2][2],...,C[n][n]](https://latex.codecogs.com/svg.image?C[1][1],C[2][2],...,C[n][n]) 을 0으로 초기화 시킨다.
- 왜냐하면 ![A_1*A_1](https://latex.codecogs.com/svg.image?A_1*A_1) 과 같은 행렬끼리는 (즉, 자기자신끼리는) 계산할 필요가 없기 때문이다.

<br>

### 절차 : 3번 라인

- L의 크기에 따라, 배열C가 점차 채워진다.

![3번 라인](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2065.png)

<br>

### 절차 : 5번 라인

- j=i+L 은 C[i][j]를 계산하기 위한 것

![5번 라인 - 1](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2066.png)

![5번 라인 - 2](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2067.png)

<br>

### 절차 : 6번 라인

- 최소 곱셈 횟수를 찾기 위해 C[i][j]로 초기화시킨다.

<br>

### 절차 : 7~10번 라인

- k값을 i ~ (j-1) 까지 변화시킴
- k값을 변화시키며, C[i][j]를 찾음

![7~10번 라인](/assets/img/2021-07-31-ALGORITHM_DP_ChainedMatrixMultiplications/Untitled%2068.png)

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