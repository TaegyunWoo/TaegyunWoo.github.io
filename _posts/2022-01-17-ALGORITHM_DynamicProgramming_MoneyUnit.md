---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - DP] 효율적인 화폐 구성"
date:   2022-01-17 12:30:00 
lastmod : 2022-01-17 12:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/dp/효율적인_화폐_구성.java)

<br/>

# 효율적인 화폐 구성
## 문제
### 문제 정의

- N가지 종류의 화폐가 있다. 이 화폐들의 개수를 최소한으로 이용해서 그 가치의 합이 M원이 되도록 하려고 한다.
- 이때 각 화폐는 몇 개라도 사용할 수 있으며, 사용한 화폐의 구성은 같지만 순서만 다른 것은 같은 경우로 구분한다.
- 예시
    - 2원, 3원 단위의 화폐가 있을 때는 15원을 만들기 위해 3원을 5개 사용하는 것이 가장 최소한의 화폐 개수이다.

<br/>

### 입력조건
- 첫째 줄에 N, M이 주어진다.
    - N: 1 이상, 100 이하
    - M: 1 이상, 10,000 이하
- 이후 N개의 줄에는 각 화폐의 가치가 주어진다.  
  화폐 가치는 10,000보다 작거나 같은 자연수이다.

<br/>

### 출력조건
- 첫째 줄에 M원을 만들기 위한 최소한의 화폐 개수를 출력한다.
- 불가능할 때는 -1을 출력한다.

<br/>

### 입·출력 예시 - 1
- 입력
  ```text
  2 15
  2
  3
  ```

- 출력
  ```text
  5
  ```

<br/>

### 입·출력 예시 - 2
- 입력
  ```text
  3 4
  3
  5
  7
  ```

- 출력
  ```text
  -1
  ```

<br/>

## 풀이
### 문제 해설 - 내 풀이
- 현재 금액 i를 만드는 방법은 아래와 같다.
  - `(i-화폐단위)+1` 을 수행하면 현재 금액 i를 만드는 화폐개수를 구할 수 있다.
  - 단 화폐단위가 여러개이므로, 현재 금액 i를 만드는 화폐 개수 중 가장 작은 것을 선택해야 한다.
- 점화식
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}=min(\alpha_{i-k_{1}},\alpha_{i-k_{2}},\alpha_{i-k_{3}},...,\alpha_{i-k_{u}})+1)
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}) : 금액 i를 만드는 최소한의 화폐 개수
  - ![](https://latex.codecogs.com/svg.image?k_{t}) : 전체 화폐단위 중, t번째 화폐단위 (t는 ![](https://latex.codecogs.com/svg.image?u) 까지 존재한다.)

<br/>

### 문제 해설 - 교재 풀이
- 적은 금액부터 큰 금액까지 확인하면서 차례대로 '만들 수 있는 최소한의 화폐 개수' 를 찾으면 된다.
- 점화식
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}=min(\alpha_{i},\alpha_{i-k}+1))
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}) : 금액 i를 만드는 최소한의 화폐 개수
  - ![](https://latex.codecogs.com/svg.image?k) : 화폐 단위
  - 단, ![](https://latex.codecogs.com/svg.image?\alpha_{i}) 의 초기값은 무한대이다.


<br/>

### 소스코드
```java
public class 효율적인_화폐_구성 {

  static int[] d = new int[10000];

  /** 점화식 (내 풀이)
    d[i] = min(d[i-n[0]], d[i-n[1]], ..., d[i-n[n.length-1]])
   */
  private static int solution1(int[] n, int m) {
    //d[i] = 금액 i를 표현할 수 있는 최소 화폐 수

    //배열 d 초기화
    Arrays.fill(d, -1);
    d[0] = 0;

    for (int nowCost = 0; nowCost <= m; nowCost++) {

      int tmp = Integer.MAX_VALUE;
      for (int moneyUnit : n) {

//        if (nowCost - moneyUnit >= 0 && d[nowCost-moneyUnit] != -1 && tmp > d[nowCost-moneyUnit]+1) {
//          tmp = d[nowCost-moneyUnit]+1;
//          d[nowCost] = tmp;
//        }
//        아래와 동일한 로직

        if (nowCost - moneyUnit >= 0) { //존재 가능한 index이고
          if (d[nowCost - moneyUnit] != - 1) { //화폐단위로 표현할 수 있었던 것이고
            if (tmp > d[nowCost - moneyUnit] + 1) { //그것이 최소값이라면
              tmp = d[nowCost-moneyUnit]+1;
              d[nowCost] = tmp; //금액 i를 표현할 수 있는 최소 화폐 수를 저장한다.
            }
          }
        }

      } //내부 for문 종료

    } //외부 for문 종료

    return d[m];
  }


  /** 점화식 (교재풀이)
    금액 i를 만들 수 있다면: d[i] = min(d[i], d[i-k]+1), 단 k는 화폐 단위
    금액 i를 만들 수 없다면: d[i] = 10001
   */
  private static int solution2(int[] n, int m) {
    //d[i] = 금액 i를 만드는데 필요한 최소 화폐 수
    Arrays.fill(d, 10001); //m의 최대값이 10000이므로, 필요한 화폐 개수가 10000을 넘어간다면 만들 수 없는 금액이라는 뜻이다.
    d[0] = 0;

    //각 화폐단위를 하나씩 적용하며, 해당 화폐단위로 만들 수 있는 금액의 최소 화폐 수를 갱신한다.
    for (int moneyUnit : n) {

      for (int nowMoney = moneyUnit; nowMoney <= m; nowMoney++) {
        if (d[nowMoney - moneyUnit] != 10001) {
          d[nowMoney] = Math.min(d[nowMoney], d[nowMoney - moneyUnit] + 1);
        }
      }

    }

    if (d[m] == 10001) {
      return -1;
    }
    return d[m];
  }

  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String s = bf.readLine();
    StringTokenizer st = new StringTokenizer(s);
    int sizeOfN = Integer.parseInt(st.nextToken());
    int m = Integer.parseInt(st.nextToken());
    int[] n = new int[sizeOfN];

    for (int i = 0; i < sizeOfN; i++) {
      n[i] = Integer.parseInt(bf.readLine());
    }

    TimeCheck.start();

    int answer = solution2(n, m);

    TimeCheck.end();

    System.out.println(answer);
  }
}

```

> 교재 풀이가 내 풀이보다 깔끔하다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>