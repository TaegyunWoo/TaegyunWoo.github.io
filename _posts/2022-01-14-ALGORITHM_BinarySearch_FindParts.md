---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - 이진탐색] 부품 찾기"
date:   2022-01-14 16:30:00 
lastmod : 2022-01-14 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/binarysearch/%EB%B6%80%ED%92%88_%EC%B0%BE%EA%B8%B0.java)

<br/>

# 부품 찾기
## 문제
### 문제 정의

- 동빈이네 전자 매장에는 부품이 N개 있다. 각 부품은 정수 형태의 고유한 번호가 있다. 어느 날 손님이 M개 종류의 부품을 대량으로 구매하겠다며 당일 날 견적서를 요청했다.
- 동빈이는 때를 놓치지 않고 손님이 문의한 부품 M개 종류를 모두 확인해서 견적서를 작성해야 한다.
- 이때 가게 안에 부품이 모두 있는지 확인하는 프로그램을 작성해보자.
- 이때 손님이 요청한 부품 번호의 순서대로 부품을 확인해 부품이 있으면 yes를, 없으면 no를 출력한다. 구분은 공백으로 한다.
- 예시
  - 가게의 부품이 총 5개일 때 부품 번호가 다음과 같다고 하자.
    ```text
    N = 5
    {8, 3, 7, 9, 2}
    ```
  - 손님은 총 3개의 부품이 있는지 확인 요청했는데 부품 번호는 다음과 같다.
    ```text
    M = 3
    {5, 7, 9}
    ```
  - 결과
    ```text
    no yes yes
    ```

<br/>

### 입력조건
- 첫째 줄에 정수 N이 주어진다.
  - N: 1 이상, 1,000,000 이하
- 둘째 줄에는 공백으로 구분하여 N개의 정수가 주어진다. 이때 정수는 1보다 크고 1,000,000 이하이다.
- 셋째 줄에는 정수 M이 주어진다.
  - M: 1 이상, 100,000 이하
- 넷째 줄에는 공백으로 구분하여 M개의 정수가 주어진다. 이때 정수는 1보다 크고 1,000,000 이하이다.

<br/>

### 출력조건
- 첫째 줄에 공백으로 구분하여 각 부품이 존재하면 yes를, 없으면 no를 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5
  8 3 7 9 2
  3
  5 7 9
  ```

- 출력
  ```text
  no yes yes
  ```

<br/>

## 풀이
### 문제 해설 - 이진 탐색을 사용하는 경우
- 본 문제처럼 다량의 데이터 검색이 필요할 땐, 이진 탐색 알고리즘을 이용해 효과적으로 처리할 수 있다.
- 문제 해결 절차
    1. 매장 내 N개의 부품을 번호 기준으로 정렬한다.
    2. 그 이후에 M개의 찾고자 하는 부품이 각각 매장에 존재하는지 검사하면 된다.
- 시간복잡도
  - N개의 부품을 정렬하는 시간: `O(NlogN)`
  - 정렬된 N개의 부품 중에서 특정 부품을 찾는 시간: `O(M * logN)`
  - 따라서 총 시간복잡도는 `O( (N+M) * logN )` 이다.

<br/>

### 문제 해설 - 계수 정렬을 사용하는 경우
- 이진 탐색을 사용하지 않고, 계수 정렬을 사용하여 문제를 해결할 수도 있다.
- 자세한 것은 소스코드를 참고하자.
- 계수정렬 사용시, 시간복잡도는 아래와 같다.
  - 매장 내 부품 번호 중 가장 큰 번호를 찾는 시간: `O(N)`
  - 정답 배열을 만드는 시간: `O(N)`
  - 부품이 존재하는지 찾는 시간: `O(M)`
  - 따라서 총 시간복잡도는 `O(2N+M) ≒ O(N+M)` 이다.

### 소스코드
```java
public class 부품_찾기 {

  /**
   * 이진탐색 직접 구현
   */
  private static void solution1(int[] n, int[] m) {
    String answer = "";
    Arrays.sort(n); //이진탐색을 수행하기 위해, 먼저 정렬을 해야 한다.

    for (int i = 0; i < m.length; i++) {

      int start = 0;
      int end = n.length - 1;
      int middle = (start+end)/2;

      //이진탐색 (반복문 활용)
      while (true) {

        //값이 없는 경우
        if (start == end && m[i] != n[middle]) {
          answer += "no ";
          break;
        }

        if (m[i] < n[middle]) {
          end = middle - 1;
          middle = (start+end)/2;
        }
        else if (m[i] > n[middle]) {
          start = middle + 1;
          middle = (start+end)/2;
        }
        else if (m[i] == n[middle]) {
          answer += "yes ";
          break;
        }
      }

    }

    System.out.println(answer);
  }

  /**
   * 이진탐색 라이브러리 활용
   */
  private static void solution2(int[] n, int[] m) {
    String answer = "";
    Arrays.sort(n); //이진탐색을 수행하기 위해, 먼저 정렬을 해야 한다.

    for (int p : m) {
      int result = Arrays.binarySearch(n, p);
      if (result >= 0) { //p를 배열 n에서 찾았다면, 해당 index가 result에 담긴다.
        answer += "yes ";
      } else { //p를 배열 n에서 찾지 못했다면, 음수가 result에 담긴다.
        answer += "no ";
      }
    }

    System.out.println(answer);
  }

  /**
   * 계수 정렬 활용
   * 모든 값이 0 이상의 정수이고, 최대 크기가 100,000,000 이기때문에 사용할 수 있다.
   */
  private static void solution3(int[] n, int[] m) {
    String answer = "";
    int max = Arrays.stream(n).max().getAsInt();
    int[] answerAry = new int[max+1];

    for (int i = 0; i < n.length; i++) {
      answerAry[n[i]]++;
    }

    for (int i = 0; i < m.length; i++) {
      if (answerAry[m[i]] > 0) {
        answer += "yes ";
      } else {
        answer += "no ";
      }
    }

    System.out.println(answer);
  }

  /**
   * 실행 메서드
   */
  public static void execute() throws Exception {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    int sizeOfN = Integer.parseInt(br.readLine());
    int[] n = new int[sizeOfN];

    String s = br.readLine();
    StringTokenizer st = new StringTokenizer(s);
    for (int i = 0; i < sizeOfN; i++) {
      n[i] = Integer.parseInt(st.nextToken());
    }

    int sizeOfM = Integer.parseInt(br.readLine());
    int[] m = new int[sizeOfM];

    s = br.readLine();
    st = new StringTokenizer(s);
    for (int i = 0; i < sizeOfM; i++) {
      m[i] = Integer.parseInt(st.nextToken());
    }

    TimeCheck.start();
//    solution1(n, m);
//    solution2(n, m);
    solution3(n, m);
    TimeCheck.end();
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