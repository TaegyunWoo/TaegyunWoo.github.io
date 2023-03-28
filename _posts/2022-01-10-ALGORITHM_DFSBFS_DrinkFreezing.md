---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - DFS] 음료수 얼려먹기"
date:   2022-01-10 15:22:00 
lastmod : 2022-01-10 15:22:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/dfs_bfs/%EC%9D%8C%EB%A3%8C%EC%88%98_%EC%96%BC%EB%A0%A4%EB%A8%B9%EA%B8%B0.java)  

<br/>

# 음료수 얼려먹기

## 문제
### 문제 정의

- NxM 크기의 얼음 틀이 있다.
- 구멍이 뚤려있는 부분은 0, 칸막이가 존재하는 부분은 1로 표시된다.
- 구멍이 뚫려 있는 부분끼리 상, 하, 좌, 우로 붙어 있는 경우 서로 연결되어 있는 것으로 간주한다.
- 이때 얼음 틀의 모양이 주어졌을 때, 생성되는 총 아이스크림의 개수를 구하는 프로그램을 작성하시오.
- 예시
  - 4x5 얼음 틀이 존재한다.

    | | | | | |
    |---|---|---|---|---|
    |0|0|1|1|0|
    |0|0|0|1|1|
    |1|1|1|1|1|
    |0|0|0|0|0|
    
  - 위 예시에선 총 3개의 아이스크림이 생성된다.

<br/>

### 입력조건
- 첫 번째 줄에 얼음 틀의 세로 길이 N과 가로 길이 M이 주어진다.
  - N: 1 이상
  - M: 1000 이하
- 두 번째 줄부터 N+1 번째 줄까지 얼음 틀의 형태가 주어진다.
- 이때 구멍이 뚫려있는 부분은 0, 그렇지 않은 부분은 1이다.

<br/>

### 출력조건
- 한 번에 만들 수 있는 아이스크림의 개수를 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  4 5
  00110
  00011
  11111
  00000
  ```

- 출력
  ```text
  3
  ```

<br/>

## 풀이
### 문제 해설
- DFS를 활용하여 해결할 수 있다.
- 얼음을 얼릴 수 있는 공간이 상, 하, 좌, 우로 연결되어 있다고 하므로, 그래프 형태로 모델링이 가능하다.
- 모델링 예시
  - 3x3 크기의 얼음 틀이 아래처럼 존재 시, 다음처럼 변환할 수 있다.
    ```text
    001
    010
    101
    ```
   ![](/assets/img/2022-01-10-ALGORITHM_DFSBFS_DrinkFreezing/Untitled32.png)
  > 위 그림에서 각 원은 노드를 뜻한다.
  - 총 3묶음이 존재한다.
- 문제 해결 절차
  1. 특정한 지점의 주변 상, 하, 좌, 우를 살펴본 뒤에 주변 지점 중에서 값이 '0'이면서 아직 방문하지 않은 지점이 있다면 해당 지점을 방문한다.
  2. 방문한 지점에서 다시 상, 하, 좌, 우를 살펴보면서 방문을 다시 진행하면, 연결된 모든 지점을 방문할 수 있다.
  3. 1~2번의 과정을 모든 노드에 반복하며 방문하지 않은 지점의 수를 센다.
- BFS가 아닌, DFS를 사용하는 이유?
  - 본 문제의 핵심은 **0을 갖는 특정 지점과 0을 갖는 인접한 지점을 찾는 것** 이다.
  - 즉 어떤 지점이 0일 때, 해당 지점과 인접한 지점을 연쇄적으로 찾아나아가야 한다.
  - 이러한 과정은 **인접한 지점이 모두 1이거나 모두 방문했었을 때** 멈춘다.
  - **그리고 이러한 과정이 멈추었을 때, 0으로 이루어진 그룹 하나를 찾아냈다고 이야기할 수 있다.**
  - 0으로 이루어진 하나의 그룹이 바로, 문제에서 이야기하는 아이스크림 한 개이다.

<br/>

### 소스코드
```java
public class 음료수_얼려먹기 {
  private static int[][] graph;
  private static int n;
  private static int m;
  private static int answer;

  private 음료수_얼려먹기() {}

  public 음료수_얼려먹기(int[][] graph, int n, int m) {
    this.graph = graph;
    this.n = n;
    this.m = m;
  }

  public boolean solution1(int x, int y) {

    //불가능한 위치인 경우
    if (x < 0 || x >= n || y < 0 || y >= m) {
      return false;
    }

    // 구멍이 뚫려있는 부분이라면 (방문한적 없다면)
    if (graph[x][y] == 0) {

      //구멍이 있는 부분을 방문했을 때, 칸막이가 존재한다고 변경하여 방문처리를 할 수 있다.
      graph[x][y] = 1;

      //상하좌우로 연결된 노드 찾기
      solution1(x-1, y); //상
      solution1(x+1, y); //하
      solution1(x, y-1); //좌
      solution1(x, y+1); //우

      return true;
    }

    return false;
  }

  public static int execute() {

    //--------- 입력 로직 ------------
    Scanner sc = new Scanner(System.in);

    // N, M을 공백을 기준으로 구분하여 입력 받기
    n = sc.nextInt();
    m = sc.nextInt();
    sc.nextLine(); // 버퍼 지우기

    // 2차원 리스트의 맵 정보 입력 받기
    graph = new int[n][m];
    for (int i = 0; i < n; i++) {
      String str = sc.nextLine();
      for (int j = 0; j < m; j++) {
        graph[i][j] = str.charAt(j) - '0';
      }
    }

    //---------- 입력 로직 끝 -----------------

    음료수_얼려먹기 tmp = new 음료수_얼려먹기(graph, n, m);

    for (int i = 0; i < n; i++) {
      for (int u = 0; u < m; u++) {
        if (tmp.solution1(i, u)) {
          answer++;
        }
      }
    }

    return answer;
  }


}

```

<br/>

### 시간복잡도
- `solution1` 메서드를 총 (n * m)번 호출한다.
- 따라서 시간복잡도는 `O(mn)` 이다.
  - `solution1` 의 시간복잡도는 `O(1)` 로 가정한다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>