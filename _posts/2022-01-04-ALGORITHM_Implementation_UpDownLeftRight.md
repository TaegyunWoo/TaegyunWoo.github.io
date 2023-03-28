---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - 구현] 상하좌우"
date:   2022-01-04 16:30:00 
lastmod : 2022-01-04 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/implementation/%EC%83%81%ED%95%98%EC%A2%8C%EC%9A%B0.java)

<br/>

# 구현 : 상하좌우

## 문제
### 문제 정의

- 여행가 A는 N*N 크기의 정사각형 공간 위에 서있다.
- 이 공간은 1*1 크기의 정사각형으로 나누어져 있다.
- 가장 왼쪽 위 좌표는 (1,1) 이며, 가장 오른쪽 아래 좌표는 (N,N)에 해당한다.
  
|||||
|---|---|---|---|
|(1,1)|(1,2)|...|(1,N)|
|...|...|...|...|
|(N,1)|(N,2)|...|(N,N)|

- 그리고 A가 이동할 계획이 적힌 계획서가 놓여있다.
- 계획서에는 하나의 줄에 띄어쓰기를 기준으로 하여 L,R,U,D 중 하나의 문자가 반복적으로 적혀 있다. 각 문자의 의미는 다음과 같다.
  - L : 왼쪽으로 한칸 이동
  - R : 오른쪽으로 한칸 이동
  - U : 위쪽으로 한칸 이동
  - D : 아래쪽으로 한칸 이동
- 이때 여행가 A가 N*N 크기의 정사각형 공간을 벗어나는 움직임은 무시된다.
  - 예를 들어, (1,1)의 위치에서 L 혹은 U를 만나면 무시된다.
- 여행가 A가 최종적으로 도착하는 곳의 좌표를 출력하라.

<br/>

### 입력조건
- 첫째줄에 N의 자연수가 주어진다.
  - N: 1 이상, 100 이하
- 둘째줄에 이동 계획서 내용이 주어진다.
  - 이동횟수: 1 이상, 100 이하

<br/>

### 출력조건
- 여행가 A가 이동 계획서에 따라 이동했을 때, 최종 위치의 좌표(X,Y)를 공백으로 구분하여 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5
  R R R U D D
  ```

- 출력
  ```text
  3 4
  ```

<br/>

## 풀이
### 문제 해설
- 계획서에 적힌 내용이 가능할 때 이동한다.

<br/>

### 소스코드
```java
public class 상하좌우 {

  private static int[] touristPosition = {1, 1};

  public static int[] solution1(int n, String[] move) {

    for (String s : move) {
      switch (s) {
        case "U":
          if (touristPosition[0] != 1) {
            touristPosition[0]--;
          }
          break;
        case "D":
          if (touristPosition[0] != n) {
            touristPosition[0]++;
          }
          break;
        case "L":
          if (touristPosition[1] != 1) {
            touristPosition[1]--;
          }
          break;
        case "R":
          if (touristPosition[1] != n) {
            touristPosition[1]++;
          }
          break;
      }
    }

    return touristPosition;
  }

}
```

<br/>

### 시간복잡도
- `solution1` 의 경우
    - 이동 계획서에 적힌 내용만큼 반복문이 수행된다.
    - 따라서 총 시간복잡도는 `O(N)` 이다.
        > 참고로, 시간복잡도에서의 N은 input N이 아니다.


<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>