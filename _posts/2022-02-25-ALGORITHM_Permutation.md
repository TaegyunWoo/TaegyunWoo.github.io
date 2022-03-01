---
category: Algorithm
tags: [알고리즘, 순열]
title: "[알고리즘 - 순열] 순열과 순열 알고리즘"
date:   2022-02-25 22:30:00 
lastmod : 2022-02-25 22:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 순열과 순열 알고리즘
## 개요
### 순열이란?
- ![공식](https://latex.codecogs.com/svg.image?_nP_r) = n개의 원소 중, 순서를 고려하여 r개를 선택하는 경우의 수
- ![공식](https://latex.codecogs.com/svg.image?_nP_r) = "n개 중 1개를 고르는 경우의 수" * "(n-1)개 중 1개를 고르는 경우의 수" * "(n-2)개 중 1개를 고르는 경우의 수" * (n-(r-1))개 중 1개를 고르는 경우의 수
- ![공식](https://latex.codecogs.com/svg.image?_nP_r=n*(n-1)*(n-2)*...*(n-r+1))

<br/>

### 순열 알고리즘 구현 방식
- 재귀함수와 `visited` 배열을 활용하여, 순열을 구한다.
- DFS, 백트래킹 기법을 사용한다.

<br/><br/>

## 예시 코드
### 문제
- ![공식](https://latex.codecogs.com/svg.image?_3P_2) 구하기
- 주어진 배열: `{1, 2, 3}`

<br/>

### 소스코드
```java
public class 순열 {

  static int[] numbers = {1, 2, 3};
  static boolean[] visited = new boolean[3];
  static int[] result = new int[3];
  static int n = 3;
  static int r = 2;

  public static void permutation(int depth) {
    if (depth == r) { //r개의 원소를 모두 골랐다면 탈출
      print();
    }

    for (int i = 0; i < n; i++) {
      if (visited[i]) continue; //i번째 원소가 이미 선택되어 있다면, 뽑는것을 생략한다.

      visited[i] = true; //i번째 원소를 뽑는다.
      result[depth] = numbers[i]; //뽑은 순서를 기억하기 위해, result 배열에 depth를 인덱스로 삼아 저장한다.
      permutation(depth+1); //재귀호출을 하여, 나머지를 뽑는다.

      //위 재귀호출을 통해, 나머지를 모두 뽑아 결과를 확인했으므로
      //i번째 원소를 뽑지 않고, 다음 원소를 고려하도록 false로 설정한다.
      visited[i] = false;
    }

  }

  /**
   * 출력 메서드
   */
  private static void print() {
    for (int i = 0; i < r; i++) {
      System.out.print(result[i] + " ");
    }
    System.out.println();
  }

}

```

<br/>

### 재귀 흐름
1.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled0.png)

<br/>

2.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled1.png)

<br/>

3.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled2.png)

<br/>

4.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled4.png)

<br/>

5.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled5.png)

<br/>

6.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled6.png)

<br/>

7.
![](/assets/img/2022-02-25-ALGORITHM_Permutation/Untitled7.png)

> 이하 생략

<br/>

## 중복 가능한 순열 구하기
### 코드
```java
public class 중복_순열 {
  static int[] numbers = new int[] {1, 2, 3};
  static int r = 3;
  static int[] result = new int[3];
  
  //3개의 원소 중, 순서를 고려하여 중복해서 3개를 뽑기
  public static void permutation(int depth, int r) { //n은 필요없다.
    if (r == 0) { //3개를 모두 뽑았다면
      printResult();
      return;
    }

    for (int i = 0; i < numbers.length; i++) { //각 원소를 뽑는다.
      result[depth] = numbers[i]; //i번째 원소를 뽑는다.
      
      //중복 가능하므로, visited에 기록할 필요가 없다.
      
      permutation(depth + 1, r - 1);
    }
  }
  
  private static void printResult() {
    for (int number : result) {
      System.out.print(number + " ");
    }
    System.out.println();
  }
}

```

### 상세 설명
- `visited` 배열이 사용되지 않음에 주목하자.
  - 중복이 가능하므로, 뽑았었던 것을 다음 선택에서 제외할 필요가 없다.
- 하지만 결과를 확인하기 위해선, 뽑았던 숫자를 기록해야 한다.
  - 따라서 `result` 배열에 기록하였다.