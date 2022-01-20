---
category: Algorithm
tags: [알고리즘, 최단거리]
title: "[알고리즘 - 최단거리] 미래도시"
date:   2022-01-20 16:30:00 
lastmod : 2022-01-20 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/shortestpath/미래도시.java)

<br/>

# 미래도시
## 문제
### 문제 정의

- 방문 판매원 A는 많은 회사가 모여 있는 공중 미래 도시에 있다. 공중 미래 도시에는 1번붜 N번까지의 회사가 있는데 특정 회사끼리는 서로 도로를 통해 연결되어 있다.
- 방문 판매원 A는 현재 1번 회사에 위치해 있으며, X번 회사에 방문해 물건을 판매하고자 한다.
- 공중 미래 도시에서 특정 회사에 도착하기 위한 방법은 회사끼리 연결되어 있는 도로를 이용하는 방법이 유일하다. 또한 연결된 2개의 회사는 양방향으로 이동할 수 있다.  
  공중 미래 도시에서의 도로는 마하의 속도로 사람을 이동시켜주기 때문에 특정 회사와 다른 회사가 도로로 연결되어 있다면, 정확히 1만큼의 시간으로 이동할 수 있다.
- 또한 오늘 방문 판매원 A는 기대하던 소개팅에도 참석하고자 한다. 소개팅의 상대는 K번 회사에 존재한다. 방문 판매원 A는 X번 회사에 가서 물건을 판매하기 전에 먼저 소개팅 상대의 회사에 찾아가서 함께 커피를 마실 예정이다.
- 따라서 방문 판매원 A는 가능한 한 빠르게 이동하고자 한다. 방문 판매원이 회사 사이를 이동하게 되는 최소 시간을 계산하는 프로그램을 작성하시오.
- 이때 소개팅 상대방과 커피를 마시는 시간 등은 고려하지 않는다고 가정한다.
- 예시
    - N = 5, X = 4, K = 5 이고 회사 간 도로가 7개면서 각 도로가 다음과 같이 연결되어 있을 때를 가정할 수 있다.
      ```text
      (1번, 2번), (1번, 3번), (1번, 4번), (2번, 4번), (3번, 4번), (3번, 5번), (4번, 5번) 
      ```
    - 이때 방문 판매원 A가 최종적으로 4번 회사에 가는 경로를 `(1번 -> 3번 -> 5번 -> 4번)` 으로 설정하면, 소개팅에도 참석할 수 있으면서 총 3만큼의 시간으로 이동할 수 있다.
    - 따라서 이 경우 최소 이동 시간은 3이다.

<br/>

### 입력조건
- 첫째 줄에 회사의 개수 N과 경로의 개수 M이 공백으로 구분되어 차례대로 주어진다.
  - N : 1 이상
  - M : 100 이하
- 둘째 줄부터 M+1번째 줄에는 연결된 두 회사의 번호가 공백으로 구분되어 주어진다.
- M+2번째 줄에는 X와 K가 공백으로 구분되어 차례대로 주어진다.
  - k : 1 이상, 100 이하

<br/>

### 출력조건
- 첫째 줄에 판맨원 A가 K번 회사를 거쳐 X번 회사로 가는 최소 이동 시간을 출력한다.
- 만약 X번 회사에 도달할 수 없다면 `-1` 을 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  5 7
  1 2
  1 3
  1 4
  2 4
  3 4
  3 5
  4 5
  4 5
  ```

- 출력
  ```text
  3
  ```

<br/>

## 풀이
### 문제 해설
- 본 문제는 전형적인 플로이드 워셜 알고리즘 문제이다.
  - 다익스트라 알고리즘을 활용하여 문제를 해결할 수 있지만, 플로이드 워셜을 사용하는 것이 훨씬 편하고 성능도 좋다.
  - 아래 소스코드에는 플로이드 워셜과 다익스트라 모두 활용한 예시가 작성되어있으니 참고하자.
- 문제의 핵심은 1번 노드에서 X를 거쳐 K로 가는 최단 거리를 구하는 것이다.
  - `1번 노드에서 X까지의 최단 거리` + `X에서 K까지의 최단 거리`

<br/>

### 소스코드
```java
public class 미래도시 {
  private static final int INFINITE = (int) 1e9;
  private static int sizeOfNode;
  private static int sizeOfEdge;
  private static int x;
  private static int k;
  private static ArrayList<ArrayList<Node>> graph = new ArrayList<>();
  private static int[] shortestTable;
  private static int[] answer = new int[2];

  /**
   * 실행 메서드
   */
  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String s = bf.readLine();
    StringTokenizer st = new StringTokenizer(s);
    sizeOfNode = Integer.parseInt(st.nextToken());
    sizeOfEdge = Integer.parseInt(st.nextToken());

    shortestTable = new int[sizeOfNode + 1];

    for (int i = 0; i <= sizeOfNode; i++) {
      graph.add(new ArrayList<>());
    }

    for (int i = 0; i < sizeOfEdge; i++) {
      s = bf.readLine();
      st = new StringTokenizer(s);
      int nodeA = Integer.parseInt(st.nextToken());
      int nodeB = Integer.parseInt(st.nextToken());
      int distance = 1;
      graph.get(nodeA).add(new Node(nodeB, distance)); //양방향이므로
      graph.get(nodeB).add(new Node(nodeA, distance)); //양방향이므로
    }

    s = bf.readLine();
    st = new StringTokenizer(s);
    x = Integer.parseInt(st.nextToken());
    k = Integer.parseInt(st.nextToken());

    //다익스트라 사용시 (총 2번 호출된다.)
//    int distanceOfK = solution1(1, k);
//    int distanceOfX = solution1(k, x);

    //플로이드 워셜 사용시 (1번만 호출해도 된다.)
    int[][] floyedGraph = solution2();
    int distanceOfK = floyedGraph[1][k];
    int distanceOfX = floyedGraph[k][x];

    if (distanceOfK >= INFINITE || distanceOfX >= INFINITE) {
      System.out.println(-1);
    } else {
      System.out.println(distanceOfX + distanceOfK);
    }

  }

  /**
   * 다익스트라 수행 메서드
   */
  private static int solution1(int startNode, int targetNode) {

    PriorityQueue<Node> priorityQueue = new PriorityQueue<>();

    Arrays.fill(shortestTable, INFINITE);
    shortestTable[startNode] = 0;
    priorityQueue.offer(new Node(startNode, 0));


    while (!priorityQueue.isEmpty()) {
      Node currentNode = priorityQueue.poll();
      int currentNodeIndex = currentNode.getNodeIndex();
      int currentNodeDistance = currentNode.getDistance();

      if (currentNodeDistance > shortestTable[currentNodeIndex]) continue;

      for (int i = 0; i < graph.get(currentNodeIndex).size(); i++) {
        int indexOfNearNode = graph.get(currentNodeIndex).get(i).getNodeIndex();
        int distanceOfCurrentToNear = graph.get(currentNodeIndex).get(i).getDistance();
        int distanceOfStartToNear = currentNodeDistance + distanceOfCurrentToNear;

        if (shortestTable[indexOfNearNode] > distanceOfStartToNear) {
          shortestTable[indexOfNearNode] = distanceOfStartToNear;
          priorityQueue.offer(new Node(indexOfNearNode, distanceOfStartToNear));
        }
      }
    }

    return shortestTable[targetNode];
  }

  /**
   * 플로이드 워셜 수행 메서드
   */
  private static int[][] solution2() {
    int[][] floyedGraph = new int[sizeOfNode+1][sizeOfNode+1];

    // ------------ 플로이드 워셜에 사용될 그래프 초기화 ---------
    for (int i = 0; i < floyedGraph.length; i++) {
      Arrays.fill(floyedGraph[i], INFINITE);
    }

    for (int i = 1; i < floyedGraph.length; i++) {
      for (int u = 1; u < floyedGraph[i].length; u++) {
        if (i == u) floyedGraph[i][u] = 0;
      }
    }

    for (int i = 0; i < graph.size(); i++) {
      for (int u = 0; u < graph.get(i).size(); u++) {
        floyedGraph[i][graph.get(i).get(u).getNodeIndex()] = graph.get(i).get(u).getDistance();
      }
    }
    // ------------ 플로이드 워셜에 사용될 그래프 초기화 (끝) ---------

    for (int k = 1; k <= sizeOfNode; k++) {
      for (int a = 1; a <= sizeOfNode; a++) {
        for (int b = 1; b <= sizeOfNode; b++) {
          floyedGraph[a][b] = Math.min(floyedGraph[a][b], floyedGraph[a][k] + floyedGraph[k][b]);
        }
      }
    }

    return floyedGraph;

  }

  private static class Node implements Comparable<Node> {
    private int nodeIndex;
    private int distance;

    public Node(int nodeIndex, int distance) {
      this.nodeIndex = nodeIndex;
      this.distance = distance;
    }

    public int getNodeIndex() {
      return nodeIndex;
    }

    public int getDistance() {
      return distance;
    }

    @Override
    public int compareTo(Node other) {
      if (this.distance < other.distance) {
        return -1;
      }
      return 1;
    }

  }
}

```

- 다익스트라 알고리즘을 사용할 시, `1번 노드에서 X까지의 최단 거리` 를 구할 때와 `X에서 K까지의 최단 거리` 를 구할 때, 총 2번 알고리즘을 수행해야 한다.
- 또한 소스코드도 복잡하다.
- 따라서 플로이드 워셜 알고리즘을 사용하는 것이 좋다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>