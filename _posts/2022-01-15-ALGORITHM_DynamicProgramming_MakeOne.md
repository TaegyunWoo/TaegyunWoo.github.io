---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - DP] 1로 만들기"
date:   2022-01-15 17:30:00 
lastmod : 2022-01-15 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/dp/만들기_1로.java)

<br/>

# 1로 만들기
## 문제
### 문제 정의

- 정수 X가 주어질 때 정수 X에 사용할 수 있는 연산은 다음과 같이 4가지이다.
  - X가 5로 나누어떨어지면, 5로 나눈다.
  - X가 3으로 나누어떨어지면, 3으로 나눈다.
  - X가 2로 나누어떨어지면, 2로 나눈다.
  - X에서 1을 뺀다.
- 정수 X가 주어졌을 때, 연산 4개를 적절히 사용해서 1을 만들려고 한다. 연산을 사용하는 횟수의 최솟값을 출력하시오.
- 예시
  - 정수가 26이라면 다음과 같이 계산해서 3번의 연산이 최솟값이다.
  1. 26 - 1 = 25
  2. 25 / 5 = 5
  3. 5 / 5 = 1

<br/>

### 입력조건
- 첫째 줄에 정수 X가 주어진다.
    - X: 1 이상, 30,000 이하

<br/>

### 출력조건
- 첫째 줄에 연산을 하는 횟수의 최솟값을 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  26
  ```

- 출력
  ```text
  3
  ```

<br/>

## 풀이
### 문제 해설
- 문제를 풀기 전에 함수가 호출되는 과정을 그림으로 그려보자. 이는 아래와 같다.  
  - ![](/assets/img/2022-01-15-ALGORITHM_DynamicProgramming_MakeOne/Untitled01.jpg)
- 이것을 점화식으로 표현하면 아래와 같다.
  ![](https://latex.codecogs.com/svg.image?\alpha_{i}=min(\alpha_{i-1},\alpha_{i/5},\alpha_{i3},\alpha_{i/2})+1)
  - ![](https://latex.codecogs.com/svg.image?i) : 현재 숫자
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}) : 현재 숫자 i를 만들기 위해 필요한 최소 연산 횟수

<br/>

### 소스코드
```java
package dp;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class 만들기_1로 {

  static int[] d = new int[30001];

  /**
   * Bottom Up 방식
   */
  private static int bottomUp(int x) {

    for (int i = 2; i <= x; i++) {

      d[i] = d[i-1] + 1;

      if (i % 2 == 0) {
        d[i] = Math.min(d[i], d[i / 2] + 1);
      }
      if (i % 3 == 0) {
        d[i] = Math.min(d[i], d[i / 3] + 1);
      }
      if (i % 5 == 0) {
        d[i] = Math.min(d[i], d[i / 5] + 1);
      }

    }

    return d[x];
  }

  /**
   * Top Down 방식
   */
  private static int topDown(int x) {
    if (x == 1) {
      return 0;
    }

    if (d[x] != 0) {
      return d[x];
    }

    d[x] = topDown(x-1) + 1;

    if (x % 5 == 0) {
      int tmp = topDown(x/5) + 1;
      if (d[x] > tmp) {
        d[x] = tmp;
      }
    }

    if (x % 3 == 0) {
      int tmp = topDown(x/3) + 1;
      if (d[x] > tmp) {
        d[x] = tmp;
      }
    }

    if (x % 2 == 0) {
      int tmp = topDown(x/2) + 1;
      if (d[x] > tmp) {
        d[x] = tmp;
      }
    }

    return d[x];
  }

  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    int x = Integer.parseInt(bf.readLine());

//    int answer = bottomUp(n);
    int answer = topDown(x);

    System.out.println(answer);
  }
}

```

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>