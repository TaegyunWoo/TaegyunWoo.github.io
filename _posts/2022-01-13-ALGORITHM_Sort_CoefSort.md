---
category: Algorithm
tags: [알고리즘, 정렬]
title: "[알고리즘 - 정렬] 계수 정렬"
date:   2022-01-13 14:10:00 
lastmod : 2022-01-13 14:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

> [소스코드](https://github.com/TaegyunWoo/algorithm-study/blob/main/src/main/java/sort/%EA%B3%84%EC%88%98%EC%A0%95%EB%A0%AC.java)  

<br/>

# 계수정렬
## 개요
### 계수정렬이란?
- 특정한 조건이 부합할 때만 사용할 수 있지만, 매우 빠른 정렬 알고리즘이다.
- **데이터의 크기 범위가 제한되어 정수 형태로 표현할 수 있을 때** 만 사용할 수 있다.
- 일반적으로 가장 큰 데이터와 가장 작은 데이터의 차이가 1,000,000을 넘지 않을 때, 효과적으로 사용할 수 있다.
- 가장 큰 데이터와 가장 작은 데이터의 차이가 너무 크다면 계수정렬을 사용할 수 없다.
  - 왜냐하면 계수정렬을 이용할 때는 **'모든 범위를 담을 수 있는 크기의 리스트(배열)를 선언'** 해야하기 때문이다.
  - 차이가 너무 크다면, 배열의 크기 또한 상당히 커진다.
- 계수정렬은 비교 기반의 정렬 알고리즘이 아니다.
  - 비교 기반 정렬 알고리즘: 선택정렬, 삽입정렬, 퀵정렬 

<br/>

### 계수정렬 동작 과정
> 입력에 제한된 범위가 있고, 0보다 크거나 같은 정수라고 가정한다.
1. 입력받은 수들 중, 가장 큰 수를 구한다. 그리고 `'가장 큰 수 값' + 1` 크기의 배열을 생성한다.  
   (이때 생성된 배열의 원소들은 모두 0으로 초기화되어 있다.)
2. 입력받은 수들을 하나씩 읽으며 아래 과정을 수행한다.
   - 현재 읽은 수를 index로 갖는 원소에 '+1'을 수행한다.
3. 모든 수들을 읽었다면, 1번에서 만든 **배열 자체** 가 정렬된 형태라고 할 수 있다.
   - index: 숫자
   - value: 중복횟수
4. 따라서 각 index를 해당 value만큼 출력하면 된다.

<br/><br/>

## 예시
### 예시에서 사용할 배열 형태
![](/assets/img/2022-01-13-ALGORITHM_Sort_CoefSort/Untitled25.jpg)

### Step 01
![](/assets/img/2022-01-13-ALGORITHM_Sort_CoefSort/Untitled26.jpg)

### Step 02
![](/assets/img/2022-01-13-ALGORITHM_Sort_CoefSort/Untitled27.jpg)

### Step 03
![](/assets/img/2022-01-13-ALGORITHM_Sort_CoefSort/Untitled28.jpg)

### Step 04~14
> 생략...

### Step 15
![](/assets/img/2022-01-13-ALGORITHM_Sort_CoefSort/Untitled29.jpg)

<br/>

### 결과
- 오름차순으로 정렬되었다.
```text
0 -> 0 -> 1 -> 1 -> 2 -> 2 -> 3 -> 4 -> 5 -> 5 -> 6 -> 7 -> 8 -> 9 -> 9
```

<br/><br/>

## 소스코드
```java
public class 계수정렬 {

  private static int[] solution(int[] n) {
    //배열 n의 각 원소는 정수이고, 0보다 크거나 같다고 가정한다.
    int[] sortAry = new int[Arrays.stream(n).max().getAsInt() + 1];

    for (int number : n) {
      sortAry[number]++;
    }

    return sortAry;
  }

  public static void execute() {
    Scanner scn = new Scanner(System.in);
    int size = scn.nextInt();
    int[] n = new int[size];

    scn.nextLine();

    for (int i = 0; i < size; i++) {
      n[i] = scn.nextInt();
    }

    TimeCheck.start();

    int[] answer = solution(n);

    TimeCheck.end();

    for (int i = 0; i < answer.length; i++) {
      for (int u = 0; u < answer[i]; u++) {
        System.out.print(i + " ");
      }
    }
  }
}

```

<br/>

## 복잡도
### 시간복잡도
- 모든 데이터가 양의 정수인 상황에서 데이터의 개수를 N, 데이터 중 최대값의 크기를 K라고 할 때, 계수 정렬의 시간 복잡도는 `O(N+K)` 이다.
- 배열의 모든 요소를 하나씩 확인해야하고, 만들어진 정답 배열에서 '최대크기를 갖는 숫자만큼 반복' 해야하기 때문이다.

<br/>

### 공간복잡도
- 계수 정렬은 때에 따라, 심각한 비효율성을 초래할 수 있다.
  - 예를 들어, 데이터가 '0'과 '999,999' 오직 2개만 있다고 하자.
  - 그럼 이때, 배열의 크기는 총 10,000,000이 된다.
- 따라서 계수 정렬은 동일한 값을 가지는 데이터가 여러 개 등장할 때 적합하다.
- 데이터의 특성을 파악하기 어렵다면 퀵 정렬을 이용하는 것이 유리하다.


<br><br>

---

<br>
<div style="font-style: italic;color: gray;">
  <ul>
    <li>나동빈, 『이것이 코딩 테스트다』</li>
  </ul>
  본 게시글은 위 교재를 기반으로 정리한 글입니다.
</div>