---
category: Algorithm
tags: [알고리즘, 정렬]
title: "[알고리즘 - 정렬] 위에서 아래로"
date:   2022-01-13 13:10:00 
lastmod : 2022-01-13 13:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/sort/%EC%9C%84%EC%97%90%EC%84%9C_%EC%95%84%EB%9E%98%EB%A1%9C.java)  

<br/>

# 위에서 아래로
## 문제
### 문제 정의

- 하나의 수여렝는 다양한 수가 존재한다.
- 이러한 수는 크기에 상관없이 나열되어 있다.
- 이 수를 큰수부터 작은 수의 순서로 정렬해야 한다.
- 수열을 내림차순으로 정렬하는 프로그램을 만드시오.

<br/>

### 입력조건
- 첫째 줄에 수열에 속해 있는 수의 개수 N이 주어진다.
    - N: 1 이상, 500 이하
- 두 번째 줄부터 N+1 번째 줄까지 N개의 수가 입력된다. 수의 범위는 1 이상 100,000 이하의 자연수이다.

<br/>

### 출력조건
- 입력으로 주어진 수열이 내림차순으로 정렬된 결과를 공백으로 구분하여 출력한다.  
  동일한 수의 순서는 자유롭게 출력해도 된다.

<br/>

### 입·출력 예시
- 입력
  ```text
  3
  15
  27
  12
  ```

- 출력
  ```text
  27 15 12
  ```

<br/>

## 풀이
### 문제 해설
- 아주 기본적인 정렬을 할 수 있는 물어보는 문제이다.
- 수의 개수가 500개 이하로 매우 적으며, 모든 수는 1 이상 100,000 이하이므로 어떠한 정렬 알고리즘을 사용해도 문제를 해결할 수 있다.
- 따라서 자바 라이브러리가 제공하는 기본 `sort()` 메서드를 사용하자.

<br/>

### 소스코드
```java
public class 위에서_아래로 {

  private static Integer[] solution1(Integer[] n) {
    Arrays.sort(n, Collections.reverseOrder());
    return n;
  }

  public static void execute() {
    Scanner scn = new Scanner(System.in);
    int size = scn.nextInt();
    Integer[] n = new Integer[size];
    scn.nextLine();

    for (int i = 0; i < size; i++) {
      n[i] = scn.nextInt();
      scn.nextLine();
    }

    TimeCheck.start();

    Integer[] answer = solution1(n);

    TimeCheck.end();

    for (int i : answer) {
      System.out.print(i + " ");
    }
  }
}

```

<br/>

### 시간복잡도
- `sort()` 메서드는 TimSort 방식을 사용하며, 이것의 시간복잡도는 `O(nlogn)`이다.

<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>