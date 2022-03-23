---
category: Algorithm
tags: [알고리즘, 벨만포드]
title: "[알고리즘] 벨만포드 알고리즘"
date:   2022-03-23 15:20:00 
lastmod : 2022-03-23 15:20:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 벨만포드 알고리즘
## 개요
### 최단거리 알고리즘 분류
- 모든 간선이 양수인 경우
  - 다익스트랑 알고리즘
- 음수 간선이 있는 경우
  - 벨만포드 알고리즘
- 모든 노드 간의 최단거리를 구해야 하는 경우
  - 플로이드워셜 알고리즘

<br/>

### 벨만포드 알고리즘이란?
- 출발노드부터 도착노드까지의 최단거리를 구하는 알고리즘이다.
- 음수인 간선이 존재하더라도 최단거리를 구할 수 있다.
- 음수 간선 순환이 있는 경우라도 구할 수 있다.
  > 음수 간선에 대해선, 나중에 서술한다.

<br/>

### 벨만포드 알고리즘 절차
1. 출발 노드 설정
2. 최단거리 테이블 초기화
3. 아래 과정을 (N-1)번 반복한다. (N = 정점의 개수)
  1. 현재 노드에서 갈 수 있는 모든 간선 E개를 하나씩 확인한다.
  2. 각 간선을 거쳐, 다른 노드로 가는 비용을 계산하여 최단걸리 테이블을 갱신한다.
    - 만약 계산된 비용이 최단거리 테이블에 저장된 값보다 작으면 갱신한다.
4. 만약 음수간선 순환이 발생하는지 체크하고 싶다면, 3번 과정을 한번 더 수행한다.
  - 음수간선 순환이란?
    - 순환이 발생했을 때, 이 사이클에 음수간선이 포함된 경우.  
      거리를 무한히 줄일 수 있게되어 제대로된 최단거리를 구할 수 없게 된다.
  - 만약 이때도 최단 거리 테이블이 갱신된다면 음수간선 순환이 존재하는 것이다.
  - 왜냐하면, 이론상 (도착노드를 제외한) 총 (N-1)개의 정점에서 시작되는 간선들을 모두 탐색했을 때,  
  각 정점까지의 최단거리는 항상 구할 수 있기 때문이다.  
  따라서 최단거리를 모두 구하고, (도착노드 → 도착노드) 거리를 계산했을 때 테이블이 또 다시 갱신된다면, 음의 간선이 포함된 사이클이 존재한다고 할 수 있다.

<br/>

### 다익스트라 vs 벨만포드
- 다익스트라
  - 매번 방문하지 않은 노드 중, 최단거리가 가장 짧은 노드를 선택한다.
  - 음수간선이 없을 때만, 최적해를 찾을 수 있다.
- 벨만포드
  - 매번 모든 간선을 전부 확인한다.
  - 다익스트라 알고리즘보다 시간이 더욱 오래걸린다.
  - 음수간선 사이클을 탐지해낼 수 있다.

<br/><br/>

## 소스코드
```java
public class 벨만포드 {

  public static final int INF = (int) 1e9;
  public List<Edge> edgeList; //간선 리스트
  int n; //노드 개수
  int m; //간선 개수
  int start; //출발지
  int destination; //도착지

  public 벨만포드(List<Edge> edgeList, int n, int start, int destination) {
    this.edgeList = edgeList;
    this.n = n;
    this.m = edgeList.size();
    this.start = start;
    this.destination = destination;
  }

  public void bellmanFord() {
    int[] table = new int[n]; //최단거리 테이블
    Arrays.fill(table, INF); //최단거리 테이블 초기화

    table[start] = 0; //시작노드 -> 시작노드는 거리가 0이므로

    //노드 개수만큼 반복
    for (int i = 0; i < n; i++) {

      //매 반복마다 모든 간선 확인
      for (int u = 0; u < m; u++) {
        int current = edgeList.get(u).getFrom();
        int next = edgeList.get(u).getTo();
        int distance = edgeList.get(u).getDistance();

        //만약 간선(u)의 출발노드(current)가 아직 한번도 갱신되지 않았다면, 생략
        if (table[current] == INF) continue;

        //만약 "최단거리 테이블에 저장된 값"이 "start노드 ~ next노드" 값보다 작다면
        if (table[next] > table[current] + distance) {

          //n번째 반복에서도 갱신된다면, 음수간선 사이클이 존재하는 것이다.
          if (i == n-1) {
            System.out.println("음수간선 사이클이 존재합니다!");
            return;
          }

          table[next] = table[current] + distance; //갱신한다.
        }
      }

    }

    //테이블 출력
    for (int i = 0; i < n; i++) {
      System.out.print(table[i] + " ");
    }
    System.out.println();

    //최단거리 출력
    System.out.println("최단거리: " + table[destination]);
  }

  /**
   * 실행 메서드
   */
  public static void execute() {
    System.out.println("<<음수간선 사이클이 없는 경우>>");
    List<Edge> edgeListA = new ArrayList<>();
    edgeListA.add(new Edge(0, 1, -4));
    edgeListA.add(new Edge(0, 2, 5));
    edgeListA.add(new Edge(0, 3, 2));
    edgeListA.add(new Edge(1, 3, -1));
    edgeListA.add(new Edge(2, 3, -7));
    edgeListA.add(new Edge(3, 4, 6));
    edgeListA.add(new Edge(4, 3, -4));

    벨만포드 b1 = new 벨만포드(edgeListA, 5, 0, 4);
    b1.bellmanFord();

    System.out.println("<<음수간선 사이클이 있는 경우>>");
    List<Edge> edgeListB = new ArrayList<>();
    edgeListB.add(new Edge(0, 1, -4));
    edgeListB.add(new Edge(0, 2, 5));
    edgeListB.add(new Edge(0, 3, 2));
    edgeListB.add(new Edge(1, 3, -1));
    edgeListB.add(new Edge(2, 3, -7));
    edgeListB.add(new Edge(3, 4, 3));
    edgeListB.add(new Edge(4, 2, 2));

    벨만포드 b2 = new 벨만포드(edgeListB, 5, 0, 4);
    b2.bellmanFord();
  }

  public static class Edge {
    private int from;
    private int to;
    private int distance;

    public Edge() {}

    public Edge(int from, int to, int distance) {
      this.from = from;
      this.to = to;
      this.distance = distance;
    }

    public int getFrom() {
      return from;
    }

    public int getTo() {
      return to;
    }

    public int getDistance() {
      return distance;
    }
  }

}
```
