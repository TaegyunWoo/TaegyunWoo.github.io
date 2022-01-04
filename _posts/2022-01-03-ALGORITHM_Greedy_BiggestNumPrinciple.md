---
category: Algorithm
tags: [알고리즘, 그리디]
title: "[알고리즘 - 그리디] 큰 수의 법칙"
date:   2022-01-03 17:30:00 
lastmod : 2022-01-03 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/greedy/%ED%81%B0_%EC%88%98%EC%9D%98_%EB%B2%95%EC%B9%99.java)

# Greedy : 큰 수의 법칙

## 문제
### 문제 정의

- 큰 수의 법칙은 다양한 수로 이루어진 배열이 있을 때, 주어진 수들을 M번 더하여 가장 큰수를 만드는 법칙이다.
- 단, 배열의 특정한 인텍스에 해당하는 수가 연속해서 K번을 초과하여 더해질 수 없다.
- 예시
    - 배열 {2, 4, 5, 4, 6} 이 있고, M이 8, K가 3이라고 가정하자.
    - 이 경우, 특정한 인덱스 수가 연속해서 세 번까지만 더해질 수 있으므로, 큰 수의 법칙에 따른 결과는 아래와 같다.
    ```text
    6 + 6 + 6 + 5 + 6 + 6 + 6 + 5
    즉, 46이 정답이다.
    ```
- 단, 서로 다른 인덱스에 해당하는 수가 같은 경우에도 서로 다른 것으로 간주한다.
- 예시
    - 배열 {3, 4, 3, 4, 3} 가 있고, M이 7, K가 2라고 가정하자.
    - 이 경우, 두번째 원소에 해당하는 4와 네번째 원소에 해당하는 4를 번갈아 두번씩 더하는 것이 가능하다.
    ```text
    4 + 4 + 4 + 4 + 4 + 4 + 4
    즉, 28이 정답이다.
    ```
- 배열의 크기 N, 숫자가 더해지는 횟수 M, 연속 허용 횟수 K가 주어질 때 큰 수의 법칙에 따른 결과를 출력하라.

<br/>

### 입력조건
- 첫째줄에 N, M, K의 자연수가 주어지며, 각 수는 공백으로 구분된다.
  - N: 2 이상, 1000 이하
  - M: 1 이상, 10000 이하
  - K: 1 이상, 10000 이하
- 둘째줄에 N개의 자연수가 주어진다. 각 자연수는 공백으로 구분된다.
  - 단 각각의 자연수는 1 이상 10000 이하의 수로 주어진다.
- 입력으로 주어지는 K는 항상 M보다 작거나 같다.

<br/>

### 출력조건
- 첫째줄에 큰 수의 법칙에 따라 더해진 답을 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5 8 3
  2 4 5 4 6
  ```

- 출력
  ```text
  46
  ```

<br/><br/>

## 풀이
### 문제 해설
- 입력으로 주어진 배열에서 사용되는 원소는 오직 `가장 큰 수` 와 `두 번째로 큰 수` 이다.
- 따라서 먼저, `가장 큰 수` 와 `두 번째로 큰 수` 를 구한다.
- 그 후, `M`과 `K`에 맞춰 정답을 구한다.
  - `가장 큰 수` 를 `K` 번 더한다.
  - `K` 번 보다 많이 반복되어 더할 수 없으므로, 이제 `두 번째로 큰 수` 를 한 번 더한다.
  - `가장 큰 수` 를 다시 `K` 번 더한다.
  - 이것을 `M` 에 맞춰 반복한다.

<br/>

### 소스코드
```java
public class 큰_수의_법칙 {

  public static int solution1(int n, int m, int k, int[] arr) {
    int answer = 0;
    int biggestNum = 0;
    int secondaryBigNum = 0;

    //가장 큰 수와 2번째로 큰 수 찾기
    for (int i : arr) {
      if (biggestNum < i) {
        biggestNum = i;
      }
    }
    for (int i : arr) {
      if (secondaryBigNum < i && biggestNum != i) {
        secondaryBigNum = i;
      }
    }

    //정답 찾기
    int temp = 0;
    for (int i = 0; i < m; i++) {
      if (temp < k) {
        answer += biggestNum;
        temp++;
      } else {
        answer += secondaryBigNum;
        temp = 0;
      }
    }

    return answer;
  }

  public static int solution2(int n, int m, int k, int[] arr) {
    int answer = 0;
    int biggestNum = 0;
    int secondaryBigNum = 0;

    //가장 큰 수와 2번째로 큰 수 찾기
    for (int i : arr) {
      if (biggestNum < i) {
        biggestNum = i;
      }
    }
    for (int i : arr) {
      if (secondaryBigNum < i && biggestNum != i) {
        secondaryBigNum = i;
      }
    }

    //정답 찾기
    int 반복횟수 = m / (k+1); // (k+1)개의 원소로 구성된 수열이 반복되는 횟수
    int 나머지 = m % (k+1); // 반복 후 남은 개수, 즉 가장큰 수가 더해져야하는 횟수
    int 수열의_합 = biggestNum * k + secondaryBigNum;
    answer = 수열의_합 * 반복횟수 + (biggestNum * 나머지);

    return answer;
  }

  public static int solution3(int n, int m, int k, int[] arr) {
      int answer = 0;
      int biggestNum = 0;
      int secondaryBigNum = 0;

      Arrays.sort(arr); //O(nlogn) 시간 소요 (Dual-Pivot Quick Sort)

      //가장 큰 수와 2번째로 큰 수 찾기
      biggestNum = arr[arr.length - 1];
      secondaryBigNum = arr[arr.length - 2];

      //정답 찾기
      int 반복횟수 = m / (k+1); // (k+1)개의 원소로 구성된 수열이 반복되는 횟수
      int 나머지 = m % (k+1); // 반복 후 남은 개수, 즉 가장큰 수가 더해져야하는 횟수
      int 수열의_합 = biggestNum * k + secondaryBigNum;
      answer = 수열의_합 * 반복횟수 + (biggestNum * 나머지);

      return answer;
  }
}
```

<br/>

### 시간복잡도
- `solution1` 의 경우
  - 가장 큰 수와 두 번째로 큰 수 구하기
    - 크기가 n인 input 배열을 처음부터 끝까지 스캔하여, 가장 큰 수를 구한다.
    - 크기가 n인 input 배열을 처음부터 끝까지 다시 스캔하여, 두 번째로 큰 수를 구한다.
    - 따라서 가장 큰 수와 두 번째로 큰 수를 구하는 시간복잡도는 `O(2n)` , 즉 `O(n)` 이다.
  - 정답 구하기
    - 가장 큰 수를 한 번 더한다. 이 연산을 k번 반복한다.
    - 두번째로 큰 수 를 한번 더한다.
    - 다시 가장 큰 수를 한 번 더한다. 이 연산을 k번 반복한다.
    - 각 원소가 더해지는 총 연산 횟수가 m번일 때까지 반복한다.
    - 따라서 정답을 구하는 시간복잡도는 `O(m)` 이다.
  - 따라서 총 시간복잡도는 `O(n) + O(m)` 이다.

<hr/>

- `solution2` 의 경우
  - 가장 큰 수와 두 번째로 큰 수 구하기
    > `solution1`와 동일한 로직이다. 따라서 시간복잡도가 `O(n)` 으로 같다.
  - 정답 구하기
    - **수열이 반복되는 것에 주목해야 한다.**
    - 예를 들어 가장 큰 수가 6이고 두번째로 큰 수가 4일 때, k가 3이라면 아래 수열이 반복될 것이다.
    ```text
    {6, 6, 6, 4}, {6, 6, 6, 4}, ...
    ```
    - 해당 수열이 총 `m / (수열의 길이)` 번 반복된다.
    - 수열의 길이는 `k+1` 과 같다. 왜냐하면 한 수열 내에서 가장 큰 수가 k번 반복되고, 두번째로 큰 수가 하나 포함되기 때문이다.
    - 또한, `m / (수열의 길이)`가 정확히 나눠지지 않는 경우도 고려해야 한다.
    - `m / (수열의 길이)` 의 나머지만큼 가장 큰 수가 반복해서 더해진다.
    - 예를 들어, 가장 큰 수가 6이고 두번째로 큰 수가 4일 때, k가 3이고 m이 7이면 아래와 같다.
    ```text
    // m/(수열의 길이) = 7/(3+1) = 1..3 이므로
    
    {6, 6, 6, 4}, {6, 6, 6}
    ```
    - 정리해보면, 정답을 구할 때 사용할 수 있는 식은 아래와 같다.
    ```text
    - j = 수열 반복 횟수 = m/(수열의 길이)
    - h = 수열 반복 후, 가장 큰 수를 추가적으로 더해줘야 하는 횟수 = m-(수열 반복 횟수)
    - g = 수열의 원소들의 합 = (가장 큰 수)*k + (두번째로 큰 수)
    - 정답 = g*j + h(가장 큰 수)
    ```
    - 위 수식을 계산하는 시간복잡도는 `O(1)` 이다.
  - 따라서 총 시간 복잡도는 `O(n)` 이다. 즉 `가장 큰 수` 와 `두번째로 큰 수` 를 구하는 시간만큼만 걸린다.

<hr/>

- `solution3` 의 경우
  - 가장 큰 수와 두 번째로 큰 수 구하기
    - `Arrays` 에서 제공하는 `Arrays.sort()` 함수를 사용하여 배열을 정렬하고, `가장 큰 수` 와 `두번째로 큰 수` 를 구한다.
    - `Arrays.sort()` 는 `Dual-Pivot Quick Sort` 를 사용하며, 정렬하는데 걸리는 시간복잡도는 `O(nlogn)` 이다.
  - 정답 구하기
    > `solution2`와 동일한 로직이다. 따라서 시간복잡도가 `O(1)` 로 같다. 
  - 따라서 총 시간복잡도는 `O(nlogn)` 이다.
  - 즉 `solution3` 이 `solution2` 보다 빠르다. 

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>