---
category: Algorithm
tags: [알고리즘, 정렬]
title: "[알고리즘 - 정렬] 성적이 낮은 순서로 학생 출력하기"
date:   2022-01-13 17:10:00 
lastmod : 2022-01-13 17:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/sort/%EC%84%B1%EC%A0%81%EC%9D%B4_%EB%82%AE%EC%9D%80_%EC%88%9C%EC%84%9C%EB%A1%9C_%ED%95%99%EC%83%9D_%EC%B6%9C%EB%A0%A5%ED%95%98%EA%B8%B0.java)  

<br/>

# 성적이 낮은 순서로 학생 출력하기
## 문제
### 문제 정의

- N명의 학생 정보가 있다. 학생 정보는 학생의 이름과 학생의 성적으로 구분된다.
- 각 학생의 이름과 성적 정보가 주어졌을 때 성적이 낮은 순서대로 학생의 이름을 출력하는 프로그램을 작성하시오.

<br/>

### 입력조건
- 첫째 줄에 학생의 수 N이 입력된다.
    - N: 1 이상, 100,000 이하
- 두 번째 줄부터 N+1 번째 줄까지 학생의 이름을 나타내는 문자열 A와 학생의 성적을 나타내는 정수 B가 공백으로 구분되어 입력된다.  
문자열 A의 길이와 학생의 성적은 100 이하의 자연수이다.

<br/>

### 출력조건
- 모든 학생의 이름을 성적이 낮은 순서대로 출력한다. 성적이 동일한 학생들의 순서는 자유롭게 출력해도 괜찮다.

<br/>

### 입·출력 예시
- 입력
  ```text
  2
  홍길동 95
  이순신 77
  김철수 84
  ```

- 출력
  ```text
  이순신 김철수 홍길동
  ```

<br/>

## 풀이
### 문제 해설
- 학생의 정보가 최대 100,000개까지 입력될 수 있으므로, 최악의 경우 `O(nlogn)`을 보장하는 알고리즘을 이용하거나, `O(n)`을 보장하는 계수 정렬을 이용하면 된다.

<br/>

### 소스코드
```java
package sort;

import time.TimeCheck;

import java.util.*;
import java.util.stream.Stream;

public class 성적이_낮은_순서로_학생_출력하기 {

  //풀이A
  private static void solution1(ArrayList<Entity> n) {
    int maxScore = 100;
    ArrayList[] answerAry = new ArrayList[maxScore + 1];

    for (int i = 0; i < answerAry.length; i++) {
      answerAry[i] = new ArrayList();
    }

    for (Entity e : n) {
      answerAry[e.score].add(e);
    }

    for (ArrayList arrayList : answerAry) {
      for (Object o : arrayList) {
        Entity e = (Entity) o;
        System.out.print(e.name + " ");
      }
    }

  }

  //풀이B
  private static void solution2(ArrayList<Entity> n) {
    n.stream().sorted(Comparator.comparing(Entity::getScore)).forEach(
      (entity) -> {System.out.print(entity.getName() + " ");}
    );
  }

  /**
   * 실행 메서드
   */
  public static void execute() {
    Scanner scn = new Scanner(System.in);
    int size = scn.nextInt();
    ArrayList<Entity> n = new ArrayList<>();
    scn.nextLine();
    for (int i = 0; i < size; i++) {
      String[] tmp = scn.nextLine().split(" ");
      n.add(new Entity(tmp[0], Integer.parseInt(tmp[1])));
    }

    TimeCheck.start();

    //solution1(n);
    solution2(n);

    TimeCheck.end();

  }

  private static class Entity {
    private String name;
    private int score;

    public Entity(String name, int score) {
      this.name = name;
      this.score = score;
    }

    public String getName() {
      return name;
    }

    public int getScore() {
      return score;
    }
  }

}

```

- `solution1` 메서드
  - 계수정렬에서 착안한 해결법이다.
  - 찾은 값(score)을 index로 삼아 '개수를 나타내는 value'에 1을 더하는 기존(계수정렬)의 방식 대신, 'List형 value'에 Entity 객체를 추가하는 방식으로 문제를 해결한다.
- `solution2` 메서드
  - 자바에서 제공하는 API를 사용하여, 문제를 해결한다.
  - 훨씬 간결한 것을 알 수 있다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>