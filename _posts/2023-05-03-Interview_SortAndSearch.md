---
category: Interview
tags: [코딩인터뷰 완전분석]
title: "[코딩인터뷰 완전분석] 면접 대비 정리 - 정렬과 탐색"
date:   2023-05-03 17:20:00 +0900
lastmod : 2023-05-03 17:20:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

> 이 글은 게일 라크만 맥도웰 저자의 ‘코딩인터뷰 완전분석’을 읽으며, 깨달은 사실과 내용을 정리하는 글입니다.  
자세한 내용이 궁금하다면 [http://www.yes24.com/Product/Goods/44305533](http://www.yes24.com/Product/Goods/44305533) 에서 책을 구매하시면 좋겠습니다.
> 

이번 포스팅에서는 면접에서 등장할 수 있는 정렬과 탐색 알고리즘에 대해 정리해보겠습니다.

많은 정렬 및 탐색 문제는 잘 알려진 알고리즘들을 변용해서 출제된다고 저자가 설명합니다.

그렇기 때문에, 여러 가지 정렬 알고리즘의 차이점을 잘 이해하고, 해당 상황에서 어떤 알고리즘이 어울릴지 살펴두는 것이 좋습니다.

*책과는 다르게 각 알고리즘을 구현한 코드를 추가했습니다. 필요하신 분은 참고하시면 되겠습니다*

<br/><br/>

## 정렬 알고리즘

먼저 정렬 알고리즘에 대해 알아볼까요?

문제에서 가장 자주 출제되는 알고리즘은 아래와 같으므로, 해당 부분을 특히 잘 이해하고 있으면 좋겠습니다.

- 병합 정렬 (Merge Sort)
- 퀵 정렬 (Quick Sort)
- 버킷 정렬 (Bucket Sort, or 기수 정렬)

<br/>

### 버블 정렬

마치 거품이 올라가는 모습과 비슷하게 동작한다고 하여, 버블 정렬이라는 이름이 붙었습니다.

- 로직
    1. 현재 원소가 그다음 원소의 값보다 크면 두 원소를 바꾸고, 이것을 배열의 끝까지 반복한다.
    2. 배열의 첫 원소부터 순차적으로 반복한다.
- 복잡도
    - 평균 및 최악 시간복잡도 : `O(n^2)`
    - 공간 복잡도 : `O(1)`

버블 정렬을 구현한 코드는 아래와 같습니다.

```java
import java.util.Arrays;

public class BubbleSort {
  //최적화 X
  public void basicSort(int[] array) {
    for (int count = 0; count < array.length; count++) {
      for (int idx = 0; idx < array.length-1; idx++) {
        if (array[idx] > array[idx+1]) {
          int tmp = array[idx+1];
          array[idx+1] = array[idx];
          array[idx] = tmp;
        }
      }
    }
  }

  //최적화 O
  public void optimizedSort(int[] array) {
    for (int count = 0; count < array.length; count++) {
      boolean isUpdated = false;

      for (int idx = 0; idx < array.length - 1 - count; idx++) {
        if (array[idx] > array[idx+1]) {
          int tmp = array[idx+1];
          array[idx+1] = array[idx];
          array[idx] = tmp;
          isUpdated = true;
        }
      }

      if (!isUpdated) break;
    }
  }

  public static void main(String[] args) {
    BubbleSort bubbleSort = new BubbleSort();
    int[] array1 = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};
    int[] array2 = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};

    bubbleSort.basicSort(array1); //최적화 X
    bubbleSort.optimizedSort(array2); //최적화 O

    System.out.println(Arrays.toString(array1)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
    System.out.println(Arrays.toString(array2)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
  }
}
```

간단히 버블 정렬을 최적화할 수도 있습니다.  
**‘배열의 처음부터 끝까지 2개 원소를 살펴보는 작업’에서 아무런 변화가 없다면 이미 정렬된 상태이므로 반복을 중단시켜도 됩니다.**  
**또한, ‘배열의 처음부터 끝까지 2개 원소를 살펴보는 작업’을 수행할 때마다, 가장 큰 수부터 배열의 뒷쪽에 정렬된 상태가 됩니다. 따라서, 한번 반복할 때마다, 뒷쪽 원소를 1개씩 확인하지 않아도 됩니다.**

<br/>

### 선택 정렬

선택 정렬은 **다음 수를 넣을 위치를 미리 지정해두고, 적절한 수를 찾는 간단한 알고리즘**입니다.

- 로직
    1. 배열의 첫 원소부터 끝까지 살펴보며, 가장 작은 원소를 찾는다.
    2. 찾은 원소와 첫 원소를 교체한다. (Pass)
    3. 배열의 두번째 원소부터 마지막 원소에 대해서 반복한다.
- 복잡도
    - 평균 및 최악 시간복잡도 : `O(n^2)`
    - 공간 복잡도 : `O(1)`

구현 코드는 아래와 같습니다.

```java
import java.util.Arrays;

public class SelectionSort {
  public void sort(int[] array) {
    for (int insertIdx = 0; insertIdx < array.length; insertIdx++) {
      int min = Integer.MAX_VALUE;
      int minIdx = 0;

      //최소값 찾기
      for (int idx = insertIdx; idx < array.length; idx++) {
        if (min > array[idx]) {
          min = array[idx];
          minIdx = idx;
        }
      }

      //최솟값과 교체
      int tmp = array[insertIdx];
      array[insertIdx] = array[minIdx];
      array[minIdx] = tmp;
    }
  }

  public static void main(String[] args) {
    SelectionSort selectionSort = new SelectionSort();
    int[] array = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};

    selectionSort.sort(array);

    System.out.println(Arrays.toString(array)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
  }
}
```

<br/>

### 병합 정렬 (중요!)

병합 정렬은 분할정복 알고리즘의 일종입니다.

배열을 절반씩 나눈 뒤, 각각 정렬한 다음 이 둘을 다시 합해서 정렬하는 방법이며, 주로 재귀함수를 통해 구현됩니다.

- 로직
    1. 주어진 배열을 절반으로 나누고, 왼쪽·오른쪽 배열에 대해서 다시 나누는 작업을 재귀 호출한다.
    2. 만약 현재 쪼개진 배열의 크기가 1 이하라면, 재귀 호출을 중단한다.
    3. 현재 쪼개진 왼쪽·오른쪽 배열의 각 원소를 비교하며, 정렬된 하나의 배열로 합치고 종료한다.
    4. 각 재귀를 종료할 때마다, 3번을 반복한다.
- 복잡도
    - 평균 및 최악 시간복잡도 : `O(nlogn)`
    - 공간복잡도 : 상황에 따라 다름 (재귀 호출의 깊이)

직접 코드를 보고 구현해보는 것이 병합 정렬을 이해하는 가장 좋은 방법이라고 생각합니다. 구현 코드는 아래와 같습니다.

```java
import java.util.Arrays;

public class MergeSort {
  /**
   * subArray : 보조 배열
   * left : 현재 살펴볼 배열 범위의 가장 왼쪽 인덱스
   * right : 현재 살펴볼 배열 범위의 가장 오른쪽 인덱스
   */
  public void sort(int[] array, int[] subArray, int left, int right) {
    if (left >= right) return; //재귀 탈출 조건

    int mid = (left + right) / 2;
    sort(array, subArray, left, mid);
    sort(array, subArray, mid + 1, right);
    merge(array, subArray, left, right, mid);
  }

  //병합 메서드
  private void merge(int[] array, int[] subArray, int left, int right, int mid) {
    //배열의 이전 상태를 보조 배열에 저장
    for (int i = left; i <= right; i++) {
      subArray[i] = array[i];
    }

    int idxA = left; //분할된 배열 A의 첫 인덱스
    int idxB = mid + 1; //분할된 배열 B의 첫 인덱스
    int arrayIdx = left; //결과를 저장할 첫 인덱스

    //분할된 배열 A와 B의 각 원소를 비교하며 array 업데이트
    while (idxA <= mid && idxB <= right) {
      if (subArray[idxA] < subArray[idxB]) { //분할된 배열 A의 값이 더 크다면
        array[arrayIdx] = subArray[idxA++];
      } else { //분할된 배열 B의 값이 더 크거나 같다면
        array[arrayIdx] = subArray[idxB++];
      }

      arrayIdx++;
    }

    //저장되지 못하고 남은 나머지 부분 처리
    if (idxA != mid+1) { //분할된 배열 A의 원소가 남은 경우
      while (idxA <= mid) {
        array[arrayIdx++] = subArray[idxA++];
      }
    }

    /* [아래 부분은 필요없음 (배열 B에 남은 원소를 저장하는 로직)]
    else if (idxB != right+1) { //분할된 배열 B의 원소가 남은 경우
      while (idxB <= mid) {
        array[arrayIdx++] = subArray[idxB++];
      }
    }

     [생략 가능한 이유]
     어차피 해당 부분은 적절한 위치의 array에 저장되어 있음
     Ex. {1, 4, 5 | 2, 8, 9} 일때, {1, 2, 4, 5} 까지 정렬해서 저장하게 되고, 배열 B는 {8, 9}가 남게됨
         하지만 array 상에서 이미 해당 위치에 저장되어 있으므로, 결과적으로 {1, 2, 4, 5, 8, 9}가 됨
    */
  }

  public static void main(String[] args) {
    MergeSort mergeSort = new MergeSort();
    int[] array = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};
    int[] subArray = new int[array.length];

    mergeSort.sort(array, subArray, 0, array.length-1);

    System.out.println(Arrays.toString(array)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
  }
}
```

<br/>

### 퀵 정렬 (중요!)

퀵 정렬은 무작위로 선택한 피봇(pivot)을 기준으로 정렬을 수행해나가는 알고리즘입니다.

- 로직
    1. 피봇을 하나 선택한다.
    2. 배열의 가장 왼쪽부터 탐색하며, 피봇보다 큰 수를 찾는다.
    3. 배열의 가장 오른쪽부터 탐색하며, 피봇보다 작은 수를 찾는다.
    4. 2번과 3번에서 찾은 원소끼리 교체한다.
    5. 2번 결과의 위치가 3번 결과의 위치보다 뒤에 있을 때까지 2·3·4를 반복한다. (즉 역전될 때까지) 
    6. 역전된 3번 결과의 위치 (피봇보다 작으며, 2번 결과의 위치보다 앞에 있는) 와 피봇을 교체한다.
    7. 교체된 피봇의 위치를 기준으로 왼쪽과 오른쪽에 대해 1번부터 반복한다.
- 복잡도
    - 평균 시간복잡도 : `O(nlogn)`
    - 최악 시간복잡도 : `O(n^2)` (피봇으로 가장 큰 수나 작은 수를 계속 선택하는 경우)
    - 공간복잡도 : 상황에 따라 다름 (재귀 호출의 깊이)

퀵 정렬은 보통 `O(nlogn)` 시간이 걸리지만, 피봇으로 적절하지 못한 수를 계속 선택한다면 시간복잡도가 `O(n^2)` 가 됩니다.  
여기서 피봇으로 적절하지 못한 수란, ‘현재 탐색하는 범위에서 가장 크거나 작은 수’를 의미합니다.

아래는 구현 코드입니다.

```java
import java.util.Arrays;

public class QuickSort {

  /**
   * start : 정렬할 범위의 가장 왼쪽 인덱스
   * end : 정렬할 범위의 가장 오른쪽 인덱스
   */
  public void sort(int[] array, int start, int end) {
    if (start >= end) return; //재귀 탈출 조건 (원소가 1개만 남았을 때도 탈출, 그 상태로 정렬된 상태이기 때문에)

    int pivot = start; //피봇 인덱스 (가장 왼쪽부터 시작)
    int left = start + 1; //왼쪽부터 탐색하며 찾은 '피벗보다 큰 수의 인덱스'
    int right = end; //왼쪽부터 탐색하며 찾은 '피벗보다 작은 수의 인덱스'

    while (true) {
      while (left <= end && array[pivot] > array[left]) left++; //피봇보다 큰 수 찾기
      while (right > start && array[pivot] < array[right]) right--; //피봇보다 작은 수 찾기 (start는 피봇 위치이므로 제외)

      if (left >= right) { //만약 left와 right가 교차됐다면, 피봇과 right 교체
        int tmp = array[pivot];
        array[pivot] = array[right];
        array[right] = tmp;
        break;

      } else { //만약 교차되지 않았다면, left와 right 교체
        int tmp = array[right];
        array[right] = array[left];
        array[left] = tmp;
      }
    }

    sort(array, start, right - 1);
    sort(array, right + 1, end);
  }

  public static void main(String[] args) {
    QuickSort quickSort = new QuickSort();
    int[] array = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};

    quickSort.sort(array, 0, array.length-1);

    System.out.println(Arrays.toString(array)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
  }
}
```

<br/>

### 기수 정렬

기수 정렬은 Queue를 사용해서, 각 자리수를 기준으로 정렬하는 과정을 반복하는 알고리즘입니다.

- 로직
    1. 0부터 9까지의 인덱스를 갖는 배열을 생성한다. 이 배열의 각 원소는 Queue이다.
    2. 기준이 되는 자리수의 값을 인덱스로 삼아서 수를 enqueue 한다.
        - Ex) `queueArray[number의 1의 자리 값].enqueue(number)`
    3. 배열의 0번째 인덱스부터 dequeue 한 결과를 순서대로 저장한다.
    4. 3번에서 저장한 결과를 가지고, 다음 자리수에 대해 2·3번을 반복한다.
- 복잡도
    - 시간복잡도 : `O(kn) (k : 자리수의 개수, n : 원소 개수)`

구현 코드는 아래와 같습니다.

```java
import java.util.*;

public class RadixSort {

  public void sort(int[] array) {
    //버킷 초기화
    Deque<Integer>[] dequeArray = new ArrayDeque[10]; //기수별로 저장할 deque 배열
    for (int i = 0; i < dequeArray.length; i++) {
      dequeArray[i] = new ArrayDeque<>();
    }

    int maxDigitSize = getMaxDigitSize(array); //가장 큰 수의 자리수 개수
    int digit = 10; //자리수 기준

    //최대 자리수만큼 반복
    for (int i = 0; i < maxDigitSize; i++) {
      //자리수를 기준으로 정렬
      for (int num : array) {
        dequeArray[(num % digit) / (digit / 10)].offerLast(num);
      }

      //정렬 결과를 순서대로 array에 저장
      int arrayIdx = 0;
      for (int u = 0; u < dequeArray.length; u++) {
        while (!dequeArray[u].isEmpty()) {
          array[arrayIdx++] = dequeArray[u].pollFirst();
        }
      }

      //자리수 기준 증가
      digit *= 10;
    }

  }

	//최대 수의 자리수 개수를 구하는 메서드
  private int getMaxDigitSize(int[] array) {
    int maxNum = Integer.MIN_VALUE;
    for (int num : array) {
      maxNum = Math.max(maxNum, num);
    }
    int count = 0;
    while (maxNum != 0) {
      count++;
      maxNum /= 10;
    }
    return count;
  }

  public static void main(String[] args) {
    RadixSort radixSort = new RadixSort();
    int[] array = {4, 2, 7, 10, 3, 5, 1, 4, 8, 21, 16, 19};

    radixSort.sort(array);

    System.out.println(Arrays.toString(array)); // [1, 2, 3, 4, 4, 5, 7, 8, 10, 16, 19, 21]
  }
}
```

<br/><br/>

## 탐색 알고리즘 (이진 탐색)

탐색의 일반적인 알고리즘은 이진 탐색입니다. 이진 탐색은 **정렬된 배열**에서 원소 x를 찾고자 할 때 사용됩니다.

- 로직
    1. 배열의 가장 왼쪽 원소 인덱스를 `min` , 가장 오른쪽 원소 인덱스를 `max` , `(min + max) / 2 = mid` 로 설정합니다.
    2. 타겟값보다 `mid` 가 가르키는 값이 더 큰 경우
        - `max = mid - 1` 로 업데이트합니다.
    3. 타겟값보다 `mid` 가 가르키는 값이 더 작은 경우
        - `min = mid + 1` 로 업데이트합니다.
    4. 타겟값과 `mid` 가 가르키는 값이 같은 경우
        - 타겟값을 찾았으므로 종료합니다.
    5. `mid = (min + max) / 2` 로 업데이트합니다.
    6. `min <= max` 일때까지 반복합니다.
- 복잡도
    - 시간복잡도 : `O(nlogn)`

예시 문제로 주어진 배열에서 특정 원소가 오름차순으로 몇번째에 위치하는지 계산해보겠습니다.

구현 코드는 아래와 같습니다.

```java
import java.util.Arrays;

public class Search {
  /**
   * 이진탐색을 통해, targetValue가 몇 번째로 작은 수인지 계산하는 메서드
   * @param rawArray 주어진 배열
   * @param targetValue 찾을 수
   */
  public int binarySearch(int[] rawArray, int targetValue) {
    //먼저 오름차순 정렬을 해야 한다. (중복 존재 허용X 라고 가정)
    Integer[] array = new Integer[rawArray.length];
    for (int i = 0; i < rawArray.length; i++) {
      array[i] = rawArray[i];
    }
    Arrays.sort(array);

    int minIdx = 0; //탐색 범위의 최소(가장 좌측) 인덱스
    int maxIdx = array.length-1; //탐색 범위의 최대(가장 우측) 인덱스
    int midIdx = (minIdx + maxIdx) / 2; //탐색 범위의 중간 인덱스

    while (minIdx <= maxIdx) {
      if (array[midIdx] < targetValue) { //타겟값보다 작은 경우
        minIdx = midIdx + 1; //최소 인덱스 증가

      } else if (array[midIdx] > targetValue) { //타겟값보다 큰 경우
        maxIdx = midIdx - 1; //최대 인덱스 감소

      } else { //타겟값을 찾은 경우
        return midIdx; //정답 반환

      }

      midIdx = (minIdx + maxIdx) / 2; //중간 인덱스 업데이트
    }

    return -1; //찾지 못한 경우 -1 반환
  }

  public static void main(String[] args) {
    Search search = new Search();
    int[] array = {4, 2, 7, 10, 3, 5, 1, 8, 21, 16, 19};

    int result = search.binarySearch(array, 5);

    System.out.println(result); //찾을 값이 오름차순으로 몇 번째에 위치했는지에 대한 결과 (0번째부터 시작) -> 4

    Arrays.sort(array);
    System.out.println(Arrays.toString(array)); //정렬된 결과
  }
}
```

<br/>

### 하지만 이진 탐색에 국한되지 말자

이진 탐색이 탐색 알고리즘 중 가장 잘 알려진 것이긴 합니다.  
하지만 이것에 국한되지 않아야 함에 주의하라고 저자가 이야기합니다.

예를 들어, 어떤 노드를 찾는 탐색 작업에는 이진 트리를 사용할 수도 있고, 해시테이블을 사용할 수도 있습니다.

따라서 이진 탐색에만 얽매이지 않도록 하는 것이 중요합니다.

<br/><br/>

## 마무리하며

이번 포스팅을 통해 여러 정렬과 이진 탐색 알고리즘에 대해 간략히 정리해봤습니다.

단순히 알고리즘을 암기하는 것 뿐만 아니라, 그 원리까지 잘 이해하고 있어야 한다고 생각합니다.

어떤 알고리즘을 이해하기 가장 쉽고 빠른 방법은 직접 구현해보는 것입니다.

이론적인 내용을 이해한 상태에서 바로 코딩해봅시다!

기본이 되는 주제인 만큼 여러번 반복해서 숙달하면, 분명히 도움이 될 것 같습니다.