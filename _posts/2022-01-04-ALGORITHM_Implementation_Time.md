---
category: Algorithm
tags: [알고리즘, 구현]
title: "[알고리즘 - 구현] 시각"
date:   2022-01-04 17:00:00 
lastmod : 2022-01-04 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/implementation/%EC%8B%9C%EA%B0%81.java)

<br/>

# 구현 : 시각

## 문제
### 문제 정의

- 정수 N이 입력되면 00시 00분 00초부터 N시 59분 59초까지의 모든 시각 중에서 3이 하나라도 포함되는 모든 경우의 수를 구하는 프로그램을 작성하시오.
- 예를 들어 1을 입력했을 때, 다음은 3이 하나라도 포함되어 있으므로 세어야 하는 시각이다.
  - 00시 00분 03초
  - 01시 37분 35초
- 반면에 다음은 3이 하나도 포함되어 있지 않으므로 세면 안되는 시각이다.
  - 00시 02분 55초
  - 01시 27분 45초

<br/>

### 입력조건
- 첫째줄에 N의 자연수가 주어진다.
    - N: 0 이상, 23 이하

<br/>

### 출력조건
- 00시 00분 00초부터 N시 59분 59초까지의 모든 시각 중에서 3이 하나라도 포함되는 모든 경우의 수를 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5
  ```

- 출력
  ```text
  11475
  ```

<br/>

## 풀이
### 문제 해설
- 모든 시각의 경우를 하나씩 모두 세서 쉽게 풀 수 있다.
- 하루는 86400초로, 최악의 경우에도 나올 수 있는 경우의 수가 86400개 밖에 없기 때문이다.

<br/>

### 소스코드
```java
public class 시각 {

  private static int n;
  private static int hour = 0;
  private static int minute = 0;
  private static int second = 0;
  private static int answer = 0;

  public static int solution1(int input) {
    n = input;
    for (; hour <= n; hour++) {
      for (; minute < 60; minute++) {
        for (; second < 60; second++) {
          if (String.valueOf(second).contains("3")) {
            answer++;
          } else if (String.valueOf(minute).contains("3")) {
            answer++;
          } else if (String.valueOf(hour).contains("3")) {
            answer++;
          }
        }
        second = 0;
      }
      minute = 0;
    }

    return answer;
  }

  public static int solution2(int input) {
    n = input;

    while (hour != -1) {
      if (String.valueOf(second).contains("3")) {
        answer++;
      } else if (String.valueOf(minute).contains("3")) {
        answer++;
      } else if (String.valueOf(hour).contains("3")) {
        answer++;
      }

      nextSecond();
    }

    return answer;
  }

  private static void nextSecond() {
    if (second != 59) {
      second++;
    } else {
      second = 0;
      nextMinute();
    }
  }

  private static void nextMinute() {
    if (minute != 59) {
      minute++;
    } else {
      minute = 0;
      nextHour();
    }
  }

  private static void nextHour() {
    if (hour < n) {
      hour++;
    } else {
      hour = -1;
    }
  }
}
```

<br/>

### 시간복잡도
- `solution1` 의 경우
    - 하루가 86400초이므로, 최악의 경우(23을 입력할 경우) 전체 반복 횟수는 86400번이다.
    - 이것을 공식화하면, `((N+1)/24)*86400`이다.
    - 즉, 전체 시간복잡도는 `O(N)`이다.

<hr/>

- `solution2` 의 경우
  - `solution1` 과 다르지 않다.
  - 코드의 형태만 다르다.


<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>