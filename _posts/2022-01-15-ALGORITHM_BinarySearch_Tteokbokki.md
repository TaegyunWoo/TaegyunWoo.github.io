---
category: Algorithm
tags: [알고리즘, 이진탐색]
title: "[알고리즘 - 이진탐색] 떡볶이 떡 만들기"
date:   2022-01-15 14:30:00 
lastmod : 2022-01-15 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/binarysearch/떡볶이_떡_만들기.java)

<br/>

# 떡볶이 떡 만들기
## 문제
### 문제 정의

- 동빈이네 떡볶이 떡은 길이가 일정하지 않다. 대신에 한 봉지 안에 들어가는 떡의 총 길이는 절단기로 잘라서 맞춰준다.
- 절단기에 높이(H)를 지정하면 줄지어진 떡을 한 번에 절단한다. 높이가 H보다 긴 떡은 H 위의 부분이 잘릴 것이고, 낮은 떡은 잘리지 않는다.
- 예시
    - 높이가 19, 14, 10, 17cm인 떡이 나란히 있고 절단기 높이를 15cm로 지정하면 자른 뒤 떡의 높이는 15, 14, 10, 15cm 가 될 것이다.
    - 잘린 떡의 길이는 차례대로 4, 0, 0, 2cm 이다. 손님은 6cm만큼의 길이를 가져간다.
- 손님이 왔을 때 요청한 총 길이가 M일 때, 적어도 M만큼의 떡을 얻기 위해 절단기에 설정할 수 있는 높이의 최댓값을 구하는 프로그램을 작성하시오.

<br/>

### 입력조건
- 첫째 줄에 떡의 개수 N과 요청한 떡의 길이 M이 주어진다.
    - N: 1 이상, 1,000,000 이하
    - M: 1 이상, 2,000,000,000 이하
- 둘째 줄에는 떡의 개별 높이가 주어진다. 떡 높이의 총합은 항상 M 이상이므로, 손님은 필요한 양만큼 떡을 사갈 수 있다. 높이는 10억보다 작거나 같은 양의 정수 혹은 0이다.

<br/>

### 출력조건
- 적어도 M만큼의 떡을 집에 가져가기 위해 절단기에 설정할 수 있는 높이의 최댓값을 출력한다.

<br/>

### 입·출력 예시 - 1
- 입력
  ```text
  4 6
  19 15 10 17
  ```

- 출력
  ```text
  15
  ```

### 입·출력 예시 - 2
- 입력
  ```text
  4 7
  19 15 10 17
  ```

- 출력
  ```text
  14
  ```

<br/>

## 풀이
### 문제 해설
- 본 문제는 전형적인 이진 탐색 문제이자, 파라메트릭 서치 유형의 문제이다.
  - 파라메트릭 서치: 최적화 문제를 결정 문제로 바꾸어 해결하는 기법
      - 예시) 범위 내에서 조건을 만족하는 가장 큰 값을 찾으라는 최적화 문제라면, 이진 탐색으로 결정 문제를 해결하면서 범위를 좁혀갈 수 있다.
      - 보통 파라메트릭 서치 유형은 이진 탐색을 이용하여 해결한다.
  - 결정 문제: '예', '아니오'로 답하는 문제
- 문제해결의 Key Point: **적절한 높이를 찾을 때까지 절단기의 높이 H를 반복해서 조정한다.**
  - '현재 이 높이로 자르면 조건을 만족할 수 있는가?'를 확인한 뒤에 조건의 만족 여부('예' 또는 '아니오')에 따라서 탐색 범위를 좁혀서 해결할 수 있다.  
    범위를 좁힐 때는 이진 탐색의 원리를 이용하여 절반씩 탐색 범위를 좁혀 나간다.
> 본 문제에서 절단기의 높이 즉, 탐색 범위가 1~10억까지의 정수이다.  
**이처럼 큰 수를 보면 당연하다는듯이 가장 먼저 이진 탐색을 떠올려야 한다.**
- 문제 해결 절차
    1. 이진 탐색의 시작점과 끝점 그리고 중간점을 아래와 같이 설정한다.
        - 시작점: 0 (가장 짧은 떡의 길이)
        - 끝점: 배열 N에서 가장 큰 수 (가장 긴 떡의 길이)
        - 중간점: (시작점+끝점)/2
            > 소수점 이하는 버린다.
    2. 중간점을 절단기의 높이 H로 취급했을 때, 잘리는 떡의 길이 (결과값)를 계산한다.
        - `결과값 < M` 인 경우:  
          절단되는 떡의 길이를 증가시켜야하므로, 절단기의 높이를 낮춰야 한다.  
          그러므로 끝점을 ('중간점의 값'+1)로 설정한다. 그리고 중간점을 (('시작점'+'끝점')/2)로 설정한다.
        - `결과값 >= M` 인 경우:  
          절단되는 떡의 길이를 감소시켜야하므로, 절단기의 높이를 증가시켜야 한다.  
          그러므로 시작점을 ('중간점의 값'-1)로 설정한다. 그리고 중간점을 (('시작점'+'끝점')/2)로 설정한다.  
          결과값과 M이 정확히 일치하지 않을 수 있다. 이때는 결과값이 M보다 큰 최소값이어야 한다.
    3. 시작점이 끝점보다 클 때까지 2번을 반복한다.
    4. 결과적으로 중간점이 정답으로 도출된다.
- 시간복잡도
    - 잘린 떡의 총 길이를 구하는 시간: `O(N)`
    - 이진 탐색으로 절단기의 높이를 구하는 시간: `O(logK) (이때 K는 떡 길이의 최댓값)`
    - 따라서 총 시간복잡도는 `O( N * logK )` 이다.

<br/>

### 소스코드
```java
public class 떡볶이_떡_만들기 {

  private static void solution1(long[] n, long m) {
    long start = 0L; //시작점
    long end = Arrays.stream(n).max().getAsLong(); //끝점
    long middle = (end+start)/2; //중간점
    long height = middle; //절단기의 높이 (중간점이 절단기의 높이가 된다.)


    while (start <= end) {


      long resultLength = getResultLength(n, height); //잘린 떡의 총 길이 구하기

      if (resultLength >= m) {
        start = middle + 1;
      } else if (resultLength < m) {
        end = middle - 1;
      }

      middle = (end+start)/2;
      height = middle;
    }

    System.out.println(height);

  }

  /**
   * 절단기 높이에 따라, 잘린 떡의 총 길이를 구하는 메서드
   */
  private static long getResultLength(long[] n, long height) {
    long resultLength = 0L; //절단기의 높이에 따라, 구해질 떡의 총 길이 (결과)

    for (int i = 0; i < n.length; i++) {
      if (n[i]-height > 0)
        resultLength += n[i]-height;
    }

    return resultLength;
  }

  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String s = bf.readLine();
    StringTokenizer st = new StringTokenizer(s);
    int sizeOfN = Integer.parseInt(st.nextToken());
    long m = Long.parseLong(st.nextToken());

    long[] n = new long[sizeOfN];

    s = bf.readLine();
    st = new StringTokenizer(s);

    for (int i = 0; i < sizeOfN; i++) {
      n[i] = Long.parseLong(st.nextToken());
    }

    TimeCheck.start();

    solution1(n, m);

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