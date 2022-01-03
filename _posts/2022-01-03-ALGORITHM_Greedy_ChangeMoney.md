---
category: Algorithm
tags: [알고리즘, 그리디]
title: "[알고리즘 - 그리디] 거스름돈"
date:   2022-01-03 17:00:00 
lastmod : 2022-01-03 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/greedy/%EA%B1%B0%EC%8A%A4%EB%A6%84%EB%8F%88.java)

# Greedy : 거스름돈

## 문제
### 문제 정의

- 당신은 음식점의 계산을 도와주는 점원이다.
- 카운터에는 거스름돈으로 사용할 500원, 100원, 50원, 10원짜리 동전이 무한히 존재한다고 가정한다.
- 손님에게 거슬러 줘야할 돈이 N원일 때, 거슬러줘야할 동전의 최소 개수를 구하라.
- 단, 거슬러 줘야할 돈 N은 항상 10의 배수이다.

## 풀이
### 문제 해설
- 가장 큰 화폐 단위부터 돈을 거슬러 주면 된다.

### 시간복잡도
- `solution1` 의 경우
  - n의 크기와 반복(`while`) 횟수가 비례한다.  
  - 따라서 n(거스름돈의 총액)이 무한하게 크다면, O(n) 시간이 걸린다.
  ```
  // 실제 테스트 결과
  n = 813249870 일 때,
  소요시간 16ms
  ```

- `solution2` 의 경우
  - 반복(`while`) 횟수가 동전 종류(k)에 비례한다.
  - 따라서 n(거스름돈의 총액)에 상관없이, O(k) 시간이 걸린다.
  ```
  // 실제 테스트 결과
  n = 813249870 일 때,
  소요시간 8ms
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