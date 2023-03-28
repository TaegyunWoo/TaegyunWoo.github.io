---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - 정렬] 두 배열의 원소 교체"
date:   2022-01-14 13:00:00 
lastmod : 2022-01-14 13:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/sort/%EB%91%90_%EB%B0%B0%EC%97%B4%EC%9D%98_%EC%9B%90%EC%86%8C_%EA%B5%90%EC%B2%B4.java)  

<br/>

# 두 배열의 원소 교체
## 문제
### 문제 정의

- 동빈이는 두 개의 배열 A와 B를 가지고 있다. 두 배열은 N개의 원소로 구성되어 있으며, 배열의 원소는 모두 자연수이다.
- 동빈이는 최대 K번의 바꿔치기 연산을 수행할 수 있는데, 바꿔치기 연산이란 배열 A에 있는 원소 하나와 배열 B에 있는 원소 하나를 골라서 두 원소를 서로 바꾸는 것을 말한다.
- 동빈이의 최종 목표는 배열 A의 모든 원소의 합이 최대가 되도록 하는 것이다.
- N, K, 그리고 배열 A와 B의 정보가 주어졌을 때, 최대 K번의 바꿔치기 연산을 수행하여 만들 수 있는 배열 A의 모든 원소의 합의 최댓값을 출력하는 프로그램을 작성하시오.
- 예시
  - N=5, K=3이고 배열 A와 n가 다음과 같다고 하자.
    - A = {1, 2, 5, 4, 3}
    - B = {5, 5, 6, 6, 5}
  - 이 경우, 다음과 같이 세 번의 연산을 수행할 수 있다.
    - 연산 1) 배열 A의 원소 '1'과 배열 B의 원소 '6'을 바꾸기
    - 연산 2) 배열 A의 원소 '2'과 배열 B의 원소 '6'을 바꾸기
    - 연산 3) 배열 A의 원소 '3'과 배열 B의 원소 '5'을 바꾸기
  - 세 번의 연산 이후 배열 A와 배열 B의 상태는 다음과 같이 구성될 것이다.
    - A = {6, 6, 5, 4, 5}
    - B = {3, 5, 1, 2, 5}
  - 이때 배열 A의 모든 원소의 합은 26이 되며, 이보다 더 합을 크게 만들 수는 없다. 따라서 정답은 26이다.

<br/>

### 입력조건
- 첫째 줄에 N, K가 공백으로 구분되어 입력된다.
    - N: 1 이상, 100,000 이하
    - K: 0 이상, N 이하
- 두 번째 줄에 배열 A의 원소들이 공백으로 구분되어 입력된다. 모든 원소는 10,000,000보다 작은 작은 자연수이다.
- 세 번째 줄에 배열 B의 원소들이 공백으로 구분되어 입력된다. 모든 원소는 10,000,000보다 작은 작은 자연수이다.

<br/>

### 출력조건
- 최대 K번의 바꿔치기 연산을 수행하여 만들 수 있는 배열 A의 모든 원소의 합의 최댓값을 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5 3
  1 2 5 4 3
  5 5 6 6 5
  ```

- 출력
  ```text
  26
  ```

<br/>

## 풀이
### 문제 해설
- '배열 A의 가장 작은 원소'와 '배열 B의 가장 큰 원소'를 최대 K번 바꿔치기하면 된다.
  - 이때 '배열 A의 원소'가 '배열 B의 원소'보다 작을 때만 바꿔치기를 수행해야 한다.
- 그러므로 배열 A와 B를 각각 정렬하고 바꿔치기 연산을 수행하면 된다.
- 배열의 원소가 10,000,000 보다 작다고 하므로, 계수정렬을 사용할 수 없다.
- 그러므로 `O(NlogN)`을 보장하는 자체 라이브러리 정렬함수를 사용하자.

<br/>

### 소스코드
```java
public class 두_배열의_원소_교체 {

  //첫번째 방법
  private static long solution1(Integer[] a, Integer[] b, int n, int k) {
    long answer = 0;

    Arrays.sort(a);
    Arrays.sort(b);

    for (int i = 0; i < k; i++) {

      if (a[i] >= b[n-1]) continue;

      int tmp = a[i];
      a[i] = b[--n];
      b[n] = tmp;
    }

    for (int i = 0; i < a.length; i++) {
      answer += a[i];
    }

    return answer;
  }

  //두번째 방법
  private static long solution2(Integer[] a, Integer[] b, int n, int k) {
    int answer = 0;

    Arrays.sort(a);
    Arrays.sort(b, Collections.reverseOrder());

    for (int i = 0; i < k; i++) {
      if (a[i] < b[i]) {
        int tmp = a[i];
        a[i] = b[i];
        b[i] = tmp;
      }
    }

    for (int i = 0; i < a.length; i++) {
      answer += a[i];
    }

    return answer;
  }

  public static void execute() {
    Scanner scn = new Scanner(System.in);
    int n = scn.nextInt();
    int k = scn.nextInt();
    Integer[] a = new Integer[n];
    Integer[] b = new Integer[n];
    scn.nextLine();

    for (int i = 0; i < n; i++) {
      a[i] = scn.nextInt();
    }
    scn.nextLine();
    for (int i = 0; i < n; i++) {
      b[i] = scn.nextInt();
    }

//    long answer = solution1(a, b, n, k);
    long answer = solution2(a, b, n, k);

    System.out.println(answer);

  }
}

```

- `solution1` 메서드
    - 배열 A와 B를 모두 오름차순으로 정렬한다.
    - 그리고 배열 A의 앞쪽 원소와 배열 B의 뒷쪽 원소를 하나씩 바꿔치기한다.
- `solution2` 메서드
    - 배열 A를 오름차순으로 정렬하고, 배열 B를 내림차순으로 정렬한다.
    - `solution1` 보다 소스코드가 더 명확해졌다.

<br/>


<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>