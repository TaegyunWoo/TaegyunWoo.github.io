---
category: Algorithm
tags: [Algorithm]
title: "[알고리즘 - BFS] 미로 탈출"
date:   2022-01-10 18:22:00 
lastmod : 2022-01-10 18:22:00
sitemap :
  changefreq : daily
  priority : 1.0
---

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/dfs_bfs/%EB%AF%B8%EB%A1%9C_%ED%83%88%EC%B6%9C.java)  

<br/>

# 미로 탈출

## 문제
### 문제 정의

- 동빈이는 NxM 크기의 직사각형 형태의 미로에 같혀 있다.
- 미로에는 여러 마리의 괴물이 있어 이를 피해 탈출해야 한다.
- 동빈이의 위치는 (1,1)이고 미로의 출구는 (N,M)의 위치에 존재하며 한 번에 한칸씩 이동할 수 있다.
- 미로는 반드시 탈출할 수 있는 형태로 제시된다.
- 이때 동빈이가 탈출하기 위해 움직여야 하는 최소 칸의 개수를 구하시오.

<br/>

### 입력조건
- 첫째 줄에 두 정수 N, M이 주어진다. 다음 N개의 줄에는 각각 M개의 정수(0 혹은 1)로 미로의 정보가 주어진다. 각각의 수들은 공백없이 붙어서 입력으로 제시된다. 또한 시작 칸과 마지막 칸은 항상 1이다.
  - N: 4 이상
  - M: 200 이하

<br/>

### 출력조건
- 첫째 줄에 최소 이동 칸의 개수를 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5 6
  101010
  111111
  000001
  111111
  111111
  ```

- 출력
  ```text
  10
  ```

<br/>

## 풀이
### 문제 해설
- BFS를 활용하여 해결할 수 있다.
  - BFS는 현재 위치에서 가장 가까운 곳(그래프 깊이가 낮은 곳)을 최우선으로 방문한다는 특성을 이용하자.
- 문제 해결 절차
    1. 시작점 위치를 enqueue 한다.
    2. 하나를 dequeue 하고, dequeue 한 위치의 상하좌우 위치를 계산한다.
    3. 계산된 위치가 존재할 수 없는 위치가 아니고 몬스터도 없다면, dequeue한 곳의 값을 각 위치(상하좌우)에 더한다. 
    4. 그리고 dequeue한 곳의 인접 위치를 enqueue 한다.
    5. (N,M)의 값이 갱신될 때까지, 2번 ~ 4번을 반복한다.
- DFS가 아닌, BFS를 사용하는 이유?
    - 본 문제의 핵심은 **시작점부터 하나씩 이동횟수를 계산하는** 이다.
    - 만약 DFS를 사용한다면, 그래프에서 가장 깊게 인접한 위치를 우선으로 탐색하므로 최소 이동 횟수를 계산할 수 없다.

<br/>

### 소스코드
```java
import java.util.*;

public class 미로_탈출 {
  private static int[][] graph;
  private static int n;
  private static int m;
  private static Deque<Position> deque = new ArrayDeque<>();

  private static int solution1() {
    deque.addLast(new Position(0, 0)); //항상 가장 왼쪽 위 지점부터 시작하므로

    while (graph[n-1][m-1] == 1) {
      Position dequeuedPosition = deque.pollFirst();
      Map<String, Position> aroundPosition = new HashMap<>();

      //dequeue한 지점을 기준으로 상, 하, 좌, 우 위치 구하기
      aroundPosition.put("up", new Position(dequeuedPosition.row-1, dequeuedPosition.column));
      aroundPosition.put("down", new Position(dequeuedPosition.row+1, dequeuedPosition.column));
      aroundPosition.put("left", new Position(dequeuedPosition.row, dequeuedPosition.column-1));
      aroundPosition.put("right", new Position(dequeuedPosition.row, dequeuedPosition.column+1));

      for (Position p : aroundPosition.values()) {
        //불가능한 위치라면 무시
        if (p.row < 0 || p.row >= n || p.column < 0 || p.column >= m) {
          continue;
        }

        //몬스터가 있다면 무시
        if (graph[p.row][p.column] == 0) {
          continue;
        }

        // 인접 지점에 이전 지점의 값 더하기
        graph[p.row][p.column] += graph[dequeuedPosition.row][dequeuedPosition.column];
        deque.addLast(p); // 인접 지점 enqueue
      }

    }

    return graph[n - 1][m - 1];
  }

  public static int execute() {
    //-------입력--------
    Scanner scn = new Scanner(System.in);
    n = scn.nextInt();
    m = scn.nextInt();
    scn.nextLine();

    graph = new int[n][m];

    for (int i = 0; i < n; i++) {
      String[] s = scn.nextLine().split("");
      for (int u = 0; u < m; u++) {
        graph[i][u] = Integer.parseInt(s[u]);
      }
    }
    //------입력 종료----------

    TimeCheck.start();

    int answer = solution1();

    TimeCheck.end();

    return answer;
  }

  static class Position {
    private int row;
    private int column;

    public Position(int row, int column) {
      this.row = row;
      this.column = column;
    }

  }
}


```

<br/>

### 시간복잡도
- `solution1` 메서드를 최대 (n)번 호출한다.
- 따라서 시간복잡도는 `O(n)` 이다.
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