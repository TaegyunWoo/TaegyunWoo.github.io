---
category: Algorithm
tags: [알고리즘, 구현]
title: "[알고리즘 - 구현] 게임 개발"
date:   2022-01-06 16:30:00 
lastmod : 2022-01-06 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/implementation/%EA%B2%8C%EC%9E%84_%EA%B0%9C%EB%B0%9C.java)

<br/>

# 구현 : 게임 개발

## 문제
### 문제 정의

- 현민이는 게임 캐릭터가 맵 안에서 움직이는 시스템을 개발 중이다.
- 캐릭터가 있는 장소는 1x1 크기의 정사각형으로 이뤄진 NxM 크기의 직사각형으로, 각각의 칸은 육지 또는 바다이다.
- 캐릭터는 동서남북 중 한 곳을 바라본다.
- 맵의 각 칸은 (A,B)로 나타낼 수 있고, A는 북쪽으로부터 떨어진 칸의 개수, B는 서쪽으로부터 떨어진 칸의 개수이다.
- 캐릭터는 상하좌우로 움직일 수 있고, 바다로 되어 있는 공간에는 갈 수 없다.
- 캐릭터의 움직임을 설정하기 위해 정해 놓은 매뉴얼은 아래와 같다.
  1. 현재 위에서 현재 방향을 기준으로 왼쪽 방향(반시계 방향으로 90도 회전한 방향)부터 차례대로 갈 곳을 정한다.
  2. 캐릭터의 바로 왼쪽 방향에 아직 가보지 않은 칸이 존재한다면, 왼쪽 방향으로 회전한 다음 왼쪽으로 한 칸을 전진한다. 왼쪽 방향에 가보지 않은 칸이 없다면, 왼쪽 방향으로 회전만 수행하고 1단계로 돌아간다.
  3. 만약 네 방향 모두 이미 가본 칸이거나 바다로 되어 있는 칸인 경우에는, 바라보는 방향을 유지한 채로 한 칸 뒤로 가고 1단계로 돌아간다. 단, 이때 뒤쪽 방향이 바다인 칸이라 뒤로 갈 수 없는 경우에는 움직임을 멈춘다.
- 현민이는 위 과정을 반복적으로 수행하면서 캐릭터의 움직임에 이상이 있는지 테이스하려고 한다.
- 매뉴얼에 따라 캐릭터를 이동시킨 뒤에, 캐릭터가 방문한 칸의 수를 출력하는 프로그램을 만드시오.

<br/>

### 입력조건
- 첫째 줄에 맵의 세로 크기 N과 가로 크기 M을 공백으로 구분하여 입력한다.
  - N : 3 이상
  - M : 50 이하
- 둘째 줄에 게임 캐릭터가 있는 칸의 좌표 (A,B)와 바라보는 방향 d가 각각 서로 공백으로 구분하여 주어진다. 방향 d의 값으로는 다음과 같이 4가지가 존재한다.
  - 0: 북쪽
  - 1: 동쪽
  - 2: 남쪽
  - 3: 서쪽
- 셋째 줄부터 맵이 육지인지 바다인지에 대한 정보가 주어진다. N개의 줄에 맵의 상태가 붂쪽부터 남쪽 순서대로, 각 줄의 데이터는 서쪽부터 동쪽 순서대로 주어진다. 맵의 외곽은 항상 바다로 되어 있다.
- 처음에 게임 캐릭터가 위치한 칸의 상태는 항상 육지이다.
<br/>

### 출력조건
- 첫째 줄에 이동을 마친 후 캐릭터가 방문한 칸의 수를 출력한다.

<br/>

### 입·출력 예시
- 입력
  ```text
  4 4
  1 1 0
  1 1 1 1
  1 0 0 1
  1 1 0 1
  1 1 1 1
  ```

- 출력
  ```text
  3
  ```

<br/>

## 풀이
### 문제 해설
- 본 문제는 전형적인 시뮬레이션 문제이다.
- 요구사항을 오류 없이 성실하게 구현하기만 하면 된다.
- 방문 처리, 바다 표시를 할 별도의 배열을 생성하여 처리한다면 보다 간단하게 해결할 수 있다.

<br/>

### 소스코드
```java
package implementation;

import java.util.Arrays;

public class 게임_개발 {

  //가본적있는지 표시하는 배열
  private static boolean[][] wentMark; //true면 가본적있음

  private static int answer = 0;

  //---첫번째 방법---
  public static int solution1(int mapHeight, int mapWidth, int[][] map, int positionA, int positionB, int lookingSide) {
    int isThereAnyToGo = 0;
    initWentMarkArr(mapHeight, mapWidth, map);


    //좌측이 한번도 안간곳인지, 바다인지 확인
    while (true) {
      switch (lookingSide) {
        case 0 :
          if (wentMark[positionA][positionB - 1]) {
            //가본적이 있다면
            lookingSide = turnLeft(lookingSide);
            isThereAnyToGo++;
          } else {
            //가본적이 없다면
            lookingSide = turnLeft(lookingSide);

            //전진
            int[] goForward = goForward(positionA, positionB, lookingSide);
            positionA = goForward[0];
            positionB = goForward[1];

            //체크
            wentMark[positionA][positionB] = true;
          }
          break;
        case 1 :
          if (wentMark[positionA-1][positionB]) {
            //가본적이 있다면
            lookingSide = turnLeft(lookingSide);
            isThereAnyToGo++;
          } else {
            //가본적이 없다면
            lookingSide = turnLeft(lookingSide);

            //전진
            int[] goForward = goForward(positionA, positionB, lookingSide);
            positionA = goForward[0];
            positionB = goForward[1];

            //체크
            wentMark[positionA][positionB] = true;
          }
          break;
        case 2 :
          if (wentMark[positionA][positionB+1]) {
            //가본적이 있다면
            lookingSide = turnLeft(lookingSide);
            isThereAnyToGo++;
          } else {
            //가본적이 없다면
            lookingSide = turnLeft(lookingSide);

            //전진
            int[] goForward = goForward(positionA, positionB, lookingSide);
            positionA = goForward[0];
            positionB = goForward[1];

            //체크
            wentMark[positionA][positionB] = true;
          }
          break;
        case 3 :
          if (wentMark[positionA+1][positionB]) {
            //가본적이 있다면
            lookingSide = turnLeft(lookingSide);
            isThereAnyToGo++;
          } else {
            //가본적이 없다면
            lookingSide = turnLeft(lookingSide);

            //전진
            int[] goForward = goForward(positionA, positionB, lookingSide);
            positionA = goForward[0];
            positionB = goForward[1];

            //체크
            wentMark[positionA][positionB] = true;
          }
          break;
      }
      if (isThereAnyToGo == 4) {
        break;
      }
    }

    //뒤로 한칸 이동
    goBackIfCan(positionA, positionB, lookingSide, map);

    return answer;

  }

  //---두번째 방법---
  public static int solution2(int mapHeight, int mapWidth, int[][] map, int positionA, int positionB, int lookingSide) {
    int isThereAnyToGo = 0;
    boolean isWent = false;
    initWentMarkArr(mapHeight, mapWidth, map);


    //좌측이 한번도 안간곳인지, 바다인지 확인
    while (true) {
      isWent = false;
      if (lookingSide == 0) {
        if (wentMark[positionA][positionB - 1]) {
          //가본적이 있거나 바다라면
          isWent = true;
        }
      } else if (lookingSide == 1) {
        if (wentMark[positionA-1][positionB]) {
          //가본적이 있거나 바다라면
          isWent = true;
        }
      } else if (lookingSide == 2) {
        if (wentMark[positionA][positionB+1]) {
          //가본적이 있거나 바다라면
          isWent = true;
        }
      } else if (lookingSide == 3) {
        if (wentMark[positionA+1][positionB]) {
          //가본적이 있거나 바다라면
          isWent = true;
        }
      }

      if (isWent) {
        //가본적이 있거나 바다라면
        lookingSide = turnLeft(lookingSide);
        isThereAnyToGo++;
      } else {
        //가본적이 없다면
        lookingSide = turnLeft(lookingSide);

        //전진
        int[] goForward = goForward(positionA, positionB, lookingSide);
        positionA = goForward[0];
        positionB = goForward[1];

        //체크
        wentMark[positionA][positionB] = true;
      }

      if (isThereAnyToGo == 4) {
        break;
      }
    }

    //뒤로 한칸 이동
    goBackIfCan(positionA, positionB, lookingSide, map);

    return answer;

  }

  private static void initWentMarkArr(int mapHeight, int mapWidth, int[][] map) {
    wentMark = new boolean[mapHeight][mapWidth];
    for (int i = 0; i < mapHeight; i++) {
      for (int u = 0; u < mapWidth; u++) {
        if (map[u][i] == 1) {
          wentMark[i][u] = true;
        }
      }
    }
  }

  private static int turnLeft(int nowLookingSide) {
    if (nowLookingSide == 0) {
      return 3;
    }
    return nowLookingSide-1;
  }

  private static int[] goForward(int positionA, int positionB, int lookingSide) {
    switch (lookingSide) {
      case 0 :
        positionA--;
        break;
      case 1:
        positionB++;
        break;
      case 2:
        positionA++;
        break;
      case 3:
        positionB--;
        break;
    }
    answer++;
    return new int[] {positionA, positionB};
  }

  private static int[] goBackIfCan(int positionA, int positionB, int lookingSide, int[][] map) {
    switch (lookingSide) {
      case 0 :
        if (map[positionA+1][positionB] == 0) {
          positionA++;
          answer++;
        }
        break;
      case 1:
        if (map[positionA][positionB-1] == 0) {
          positionB--;
          answer++;
        }
        break;
      case 2:
        if (map[positionA-1][positionB] == 0) {
          positionA--;
          answer++;
        }
        break;
      case 3:
        if (map[positionA][positionB+1] == 0) {
          positionB++;
          answer++;
        }
        break;
    }
    return new int[] {positionA, positionB};
  }
}

```

- `solution2`에서는 `solution1`을 리팩토링하여, 중복되는 코드를 최대한 줄였다.


<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>