---
category: Algorithm
tags: [알고리즘, 최단거리]
title: "[알고리즘 - 최단거리] 전보"
date:   2022-01-21 11:30:00 
lastmod : 2022-01-21 11:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/shortestpath/전보.java)

<br/>

# 전보
## 문제
### 문제 정의

- 어떤 나라에는 N개의 도시가 있다. 그리고 각 도시는 보내고자 하는 메시지가 있는 경우, 다른 도시로 전보를 보내서 다른 도시로 해당 메시지를 전송할 수 있다.
- 하지만 X라는 도시에서 Y라는 도시로 전보를 보내고자 한다면, 도시 X에서 Y로 향하는 통로가 설치되어 있어야 한다.
  - 예를 들어 X에서 Y로 향하는 통로는 있지만, Y에서 X로 향하는 통로가 없다면 Y는 X로 메시지를 보낼 수 없다.
- 또한 통로를 거쳐 메시지를 보낼 때는 일정 시간이 소요된다.
- 어느날 C라는 도시에 위급상황이 발생했다. 그래서 최대한 많은 도시로 메시지를 보내고자 한다. 메시지는 도시 C에서 출발하여 각 도시 사이에 설치된 통로를 거쳐, 최대한 많이 퍼져나갈 것이다.
- 각 도시의 번호와 통로가 설치되어 있는 정보가 주어졌을 때, 도시 C에서 보낸 메시지를 받게 되는 도시의 개수는 총 몇 개이며 도시들이 모두 메시지를 받는 데까지 걸리는 시간은 얼마인지 계산하는 프로그램을 작성하시오.

<br/>

### 입력조건
- 첫째 줄에 도시의 개수 N, 통로의 개수 M, 메시지를 보내고자 하는 도시 C가 주어진다.
    - N : 1 이상, 30,000 이하
    - M : 1 이상, 200,000 이하
    - C : 1 이상, N 이하
- 둘째 줄부터 M+1번째 줄에 걸쳐서 통로에 대한 정보 X, Y, Z가 주어진다. 이는 특정 도시 X에서 다른 특정 도시 Y로 이어지는 통로가 있으며, 메시지가 전달되는 시간이 Z라는 의미이다.
  - X : 1 이상
  - Y : N 이하
  - Z : 1 이상, 1,000 이하

<br/>

### 출력조건
- 첫째 줄에 도시 C에서 보낸 메시지를 받는 도시의 총 개수와 총 걸리는 시간을 공백으로 구분하여 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  3 2 1
  1 2 4
  1 3 2
  ```

- 출력
  ```text
  2 4
  ```

<br/>

## 풀이
### 문제 해설
- 다익스트라 알고리즘을 사용하면 아주 간단하게 문제를 해결할 수 있다.
- N과 M의 범위가 크기 때문에, 우선순위 큐를 이용하는 것이 좋다.
- 정답으로 요구하는 `메시지를 받는 도시 개수` 와 `총 소요 시간` 을 분석해보면 아래와 같다.
  - `메시지를 받는 도시 개수` : 최단 거리 테이블에서 **초기에 초기화한 값(`INFINITE`)** 가 아닌 나머지 노드(도시) 개수
  - `총 소요 시간` : 시작노드(도시 C)에서 거리(시간)가 가장 먼 도시까지의 거리(시간)
- 따라서 다익스트라 알고리즘을 수행한 뒤, 그에 대한 결과인 **최단 거리 테이블** 을 활용하여 정답을 구하면 된다.
- 참고로 최단 거리 테이블을 초기화할 때 사용하는 무한대 값은 `(int) 1e9` 이면 충분하다.
  - 도시의 개수(N)이 최대 30,000 개이고, 시간(M)가 최대 1,000이므로 최악의 경우 시간의 총합은 `(N-1)*M`으로, 대략 3천만 시간이다. `(int) 1e9` 는 10억이다.

<br/>

### 소스코드
```java
public class 전보 {
  private static final int INFINITE = (int) 1e9; //가능한 최대값이 3억까지이므로
  private static ArrayList<ArrayList<Node>> graph = new ArrayList<>();
  private static int[] shortestTable = new int[30001];
  private static int sizeOfNode;
  private static int sizeOfEdge;
  private static int startNode;

  /**
   * 실행 메서드
   */
  public static void execute() throws Exception {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String s = bf.readLine();
    StringTokenizer st = new StringTokenizer(s);
    sizeOfNode = Integer.parseInt(st.nextToken());
    sizeOfEdge = Integer.parseInt(st.nextToken());
    startNode = Integer.parseInt(st.nextToken());

    for (int i = 0; i <= sizeOfNode; i++) {
      graph.add(new ArrayList<>());
    }

    for (int i = 0; i < sizeOfEdge; i++) {
      s = bf.readLine();
      st = new StringTokenizer(s);
      int fromNode = Integer.parseInt(st.nextToken());
      int toNode = Integer.parseInt(st.nextToken());
      int distance = Integer.parseInt(st.nextToken());
      graph.get(fromNode).add(new Node(toNode, distance));
    }

    int[] answer = solution1();

    for (int a : answer) {
      System.out.print(a + " ");
    }

  }

  /**
   * 다익스트라 알고리즘 수행 메서드
   */
  private static int[] solution1() {
    PriorityQueue<Node> priorityQueue = new PriorityQueue<>();

    Arrays.fill(shortestTable, INFINITE);
    shortestTable[startNode] = 0;
    priorityQueue.offer(new Node(startNode, 0));

    while (!priorityQueue.isEmpty()) {

      Node currentNode = priorityQueue.poll();
      int currentNodeIndex = currentNode.getNodeIndex();
      int currentDistance = currentNode.getDistance();

      if (shortestTable[currentNodeIndex] < currentDistance) {
        continue;
      }

      for (int i = 0; i < graph.get(currentNodeIndex).size(); i++) {
        int indexOfNearNode = graph.get(currentNodeIndex).get(i).getNodeIndex();
        int distanceOfCurrentToNear = graph.get(currentNodeIndex).get(i).getDistance();
        int distanceOfStartToNear = distanceOfCurrentToNear + shortestTable[currentNodeIndex];
        if (shortestTable[indexOfNearNode] > distanceOfStartToNear) {
          shortestTable[indexOfNearNode] = distanceOfStartToNear;
          priorityQueue.offer(new Node(indexOfNearNode, distanceOfStartToNear));
        }
      }

    }

    return getAnswer();
  }

  private static int[] getAnswer() {
    int cityCount = 0;
    int timeCount = 0;

    for (int d : shortestTable) {
      if (d != INFINITE) {
        cityCount++;
        timeCount = Math.max(timeCount, d);
      }
    }

    return new int[] {cityCount-1, timeCount}; //시작노드는 정답에서 제외되므로 '-1'
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

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>