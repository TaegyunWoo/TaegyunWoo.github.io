---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - DP] 개미전사"
date:   2022-01-15 18:30:00 
lastmod : 2022-01-15 18:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/dp/개미전사.java)

<br/>

# 개미전사
## 문제
### 문제 정의

- 개미 전사는 부족한 식량을 충당하고자 메뚜기 마을의 식량창고를 몰래 공격하려고 한다.
- 메뚜기 마을에는 여러 개의 식량창고가 있는데 식량창고는 일직선으로 이어져 있다. 각 식량창고에는 정해진 수의 식량을 저장하고 있으며 개미 전사는 식량창고를 선택적으로 약탈하여 식량을 빼앗을 예정이다.
- 이때 메뚜기 정찰병들은 일직선상에 존재하는 식량창고 중에서 서로 인접한 식량창고가 공격받으면 바로 알아챌 수 있다.
- 따라서 개미 전사가 정찰병에게 들키지 않고 식량창고를 약탈하기 위해서는 최소한 한 칸 이상 떨어진 식량창고를 약탈해야 한다.
- 예시
  - 식량창고 4개가 다음과 같이 존재한다고 가정하자.
    ```text
    {1, 3, 1, 5}
    ```
    
    |||||
    |---|---|---|---|
    |1|3|1|5|
  
  - 이때 개미 전사는 두 번째 식량창고와 네 번째 식량창고를 선택했을 때 최댓값인 총 8개의 식량을 빼앗을 수 있다.
- 개미 전사는 식량창고가 이렇게 일직선상일 때 최대한 많은 식량을 얻기를 원한다.
- 개미 전사를 위해 식량창고 N개에 대한 정보가 주어졌을 때 얻을 수 있는 식량의 최댓값을 구하는 프로그램을 작성하시오.

<br/>

### 입력조건
- 첫째 줄에 식량창고의 개수 N이 주어진다.
    - N: 3 이상, 100 이하
- 둘째 줄에 공백으로 구분되어 각 식량창고에 저장된 식량의 개수 K가 주어진다.
    - K: 0 이상, 1,000 이하

<br/>

### 출력조건
- 첫째 줄에 개미 전사가 얻을 수 있는 식량의 최댓값을 출력하시오.

<br/>

### 입·출력 예시
- 입력
  ```text
  4
  1 3 1 5
  ```

- 출력
  ```text
  8
  ```

<br/>

## 풀이
### 문제 해설
- 개미 전사가 어떤 식량창고에 도착했을 때, 선택지는 아래와 같이 총 2가지이다.
  - 직전의 창고를 털었으므로, 현재 창고를 털지 못한다.
  - 직전의 창고를 털지 않아, 현재 창고를 털 수 있다.
- 위 2가지 선택지 중, 약탈한 식량의 개수가 최댓값이 되는 선택지를 선택하면 된다.
- 이를 점화식으로 나타내면 아래와 같다.  
  ![](https://latex.codecogs.com/svg.image?\alpha_{i}=max(\alpha_{i-1},\alpha_{i-2}+k_{i}))
  - ![](https://latex.codecogs.com/svg.image?\alpha_{i}) : i번째 창고까지 도착했을 때, 가능한 최대 식량값
  - ![](https://latex.codecogs.com/svg.image?k_{i}) : i번째 창고의 식량 개수
  > **즉 '현재 창고를 털지 않고, 직전 창고까지 턴 것'이나 '현재 창고를 털고, 2개 전 창고를 턴 것' 중 더 식량값이 큰 것을 택한다.**

<br/>

### 소스코드
```java
public class 개미전사 {

  static int[] d = new int[100]; // d[i] = i번째 창고에 도착했을 때의 최대값 (i번째 창고를 털수있는지는 확정 X)

  /**
   * Bottom Up 방식
   */
  private static int bottomUp(int[] n) {

    d[0] = n[0];
    d[1] = Math.max(n[0], n[1]);

    for (int i = 2; i < n.length; i++) {
      d[i] = Math.max(d[i-1], d[i-2]+n[i]);
    }

    return d[n.length-1];
  }

  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    int sizeOfN = Integer.parseInt(bf.readLine());
    int[] n = new int[sizeOfN];
    String s = bf.readLine();
    StringTokenizer st = new StringTokenizer(s);

    for (int i = 0; i < sizeOfN; i++) {
      n[i] = Integer.parseInt(st.nextToken());
    }

    int answer = bottomUp(n);

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