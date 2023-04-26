---
category: Interview
tags: [코딩인터뷰 완전분석]
title: "[코딩인터뷰 완전분석] 면접 대비 정리 - 자료구조"
date:   2023-04-26 23:20:00 +0900
lastmod : 2023-04-26 23:20:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

> 이 글은 게일 라크만 맥도웰 저자의 ‘코딩인터뷰 완전분석’을 읽으며, 깨달은 사실과 내용을 정리하는 글입니다.  
자세한 내용이 궁금하다면 [http://www.yes24.com/Product/Goods/44305533](http://www.yes24.com/Product/Goods/44305533) 에서 책을 구매하시면 좋겠습니다.

이번에는 면접에서 자주 등장하는 자료구조에 대해 정리해보는 시간을 갖겠습니다!

자료구조가 컴퓨터 지식의 기초 중 기초로 중요하기 때문에, 알고 있는 부분이더라도 다시 한번 확인해보는게 좋겠습니다.

*한번에 많은 종류의 자료구조를 다루기 때문에, 분량상 많은 내용을 생략했습니다.  
더 많은 내용이 궁금하시다면 중간중간 등장하는 추가 포스팅 링크를 참고하시면 도움이 될 것 같습니다.*

<br/><br/>

## 해시 테이블

해시 테이블은 Key-Value 쌍으로 자료를 저장합니다.

해시 테이블을 구현하기 위해선, **연결리스트와 해시 코드 함수**가 필요합니다.

<br/>

### 해시 테이블의 저장 과정

1. 해시 함수를 통해, Key의 해시 코드를 계산합니다. 이때 어떤 자료형으로도 Key를 만들 수 있습니다. 해시 코드는 int나 long형으로 도출됩니다.  
**키의 개수는 무한하지만, int나 long의 범위는 유한하기 때문에 서로 다른 두 개의 Key값이 동일한 해시 코드를 갖을 수 있다는 것을 알아둡시다.**
2. `hash(key) % array_length` 으로 key-value를 저장할 배열의 인덱스를 구합니다.  
**물론 서로 다른 해시 코드가 같은 인덱스를 가리킬 수도 있습니다.**
    - 배열의 크기는 유한하기 때문에, `%` 연산을 통해서 Index Out of Bounds 오류를 예방해야 합니다.
3. 배열의 각 인덱스는 `LinkedList<>` 같은 연결리스트가 존재합니다. 해당 인덱스에 **Key와 Value를 함께 저장**합니다.
    - 연결리스트에 저장하는 이유 : 서로 다른 Key가 동일한 해시코드나 Index를 갖을 수 있기 때문에, 데이터 누락없이 저장하려면 연결리스트에 이어서 저장해야 합니다.
    - 서로 다른 Key가 동일한 해시코드를 갖는 것을 **해시 충돌**이라고 합니다.
    
    > [해시 충돌]  
    [https://taegyunwoo.github.io/algorithm/ALGORITHM_HashAlgorithm](https://taegyunwoo.github.io/algorithm/ALGORITHM_HashAlgorithm)  
    [해시 충돌 해결방법]  
    [https://taegyunwoo.github.io/interview/CS_VectorVsArrayList](https://taegyunwoo.github.io/interview/CS_VectorVsArrayList)
    > 

<br/>

### 해시 테이블 조회 과정

1. 해시 함수를 통해, 주어진 Key의 해시코드를 구합니다.
2. `hash(key) % array_length` 으로 조회할 배열의 인덱스를 구합니다.
3. 해당 인덱스의 연결리스트의 원소를 **하나씩 선형탐색**하며, 주어진 Key와 동일한 데이터를 찾습니다.
4. 찾은 연결리스트의 원소를 반환합니다.

여기서 핵심은 **연결리스트를 선형탐색**한다는 것입니다.

따라서 충돌이 자주 발생한다면, **최악의 경우 수행 시간은 `O(n) (n은 Key의 개수)`** 가 됩니다.

하지만 일반적으로 해시에 대해 이야기 할 때는 충돌을 최소화하도록 구현된 경우를 가정하기 때문에, 일반적인 시간복잡도는 `O(1)` 입니다.

<br/>

### 해시 테이블 구현

```java
import java.util.*;

public class HashTable {
  public static final int MAX_SIZE = 10; //데이터를 저장할 배열의 크기
  private List<KeyValue>[] listAry; //데이터를 저장할 배열

  public HashTable() {
    listAry = new LinkedList[MAX_SIZE];
    for (int i = 0; i < listAry.length; i++) {
      listAry[i] = new LinkedList<>();
    }
  }

	//저장 메서드
  public void put(String key, String value) {
    //TODO - 파라미터 Null 체크
    int index = getIndex(key);
    listAry[index].add(new KeyValue(key, value));
  }

	//조회 메서드
  public String get(String key) {
    //TODO - 파라미터 Null 체크
    int index = getIndex(key);
    for (KeyValue item : listAry[index]) {
      if (item.key.equals(key)) return item.value;
    }
    return null;
  }

	//배열의 인덱스 값을 구하는 메서드
  private int getIndex(String key) {
    return key.length() % MAX_SIZE; //이 경우, Key의 길이가 해시 코드가 된다.
		// return key.hashCode() % MAX_SIZE; 처럼 이미 구현된 해시 함수를 사용하는 것이 좋다.
  }

	//연결리스트에 저장할 원소 (Key-Value 쌍)
  private class KeyValue {
    public String key;
    public String value;
    public KeyValue(String key, String value) {
      this.key = key;
      this.value = value;
    }
  }
}

//테스트
public static void main(String[] args) {
  HashTable hashTable = new HashTable();
  hashTable.put("aaa", "A");
  hashTable.put("bb", "B");
	hashTable.put("eeeeeeeeeeeeeee", "E");
  hashTable.put("ccc", "C"); //충돌
  hashTable.put("dd", "D"); //충돌

  System.out.println("aaa : " + hashTable.get("aaa")); // A
  System.out.println("bb : " + hashTable.get("bb")); // B
  System.out.println("ccc : " + hashTable.get("ccc")); // C
  System.out.println("dd : " + hashTable.get("dd")); // D
	System.out.println("eeeeeeeeeeeeeee : " + hashTable.get("eeeeeeeeeeeeeee")); // E
}
```

<br/><br/>

## ArrayList와 가변 크기 배열

특정 언어에서는 배열의 크기를 자동으로 조절할 수 있습니다.

하지만 Java와 같은 언어에서는 배열의 길이가 고정되어 있습니다.  
이 경우, 배열을 만들 때 미리 배열의 크기를 설정해야 합니다.

동적 가변 크기 기능이 내재되어 있는 배열 자료구조를 원할 때는 보통 ArrayList를 사용합니다.

**ArrayList는 필요에 따라 크기를 변화시킬 수 있으면서도 `O(1)` 의 접근시간을 보장합니다.**

**통상적으로 배열이 가득 차는 순간, 배열의 크기를 두 배로 늘립니다.**

> Java의 경우, Vector는 2배로 늘리고 ArrayList는 1.5배로 늘립니다.
> 

**크기를 두 배로 늘리는 시간은 `O(n)` 이지만, 자주 발생하는 일이 아니기 때문에 값을 저장하고 조회하는 시간복잡도는 `O(1)` 입니다.**

<br/>

### ArrayList 구현

```java
public class ArrayList {
  public static final int BASE_SIZE = 2; //배열의 기본 크기
  public static final int APPEND_SIZE = 2; //배열 확장 계수
  private int size = 0; //현재 저장된 데이터 개수
  private int[] array; //배열

  public ArrayList() {
    this.array = new int[BASE_SIZE];
  }

	//추가 메서드
  public void add(int value) {
    if (size == array.length) appendArray(); //배열이 현재 꽉 찼다면, 배열 확장
    array[size++] = value; //마지막에 저장
  }

	//삽입 메서드
  public void add(int index, int value) {
    //TODO - IndexOutOfBound (size로 검사)
    if (size == 0) {
      add(value);
      return;
    }

    if (size == array.length) appendArray(); //배열이 현재 꽉 찼다면, 배열 확장

		//삽입할 위치의 뒷쪽 데이터들을 모두 한칸씩 뒤로 이동
    for (int i = size-1; i >= index; i--) {
      array[i+1] = array[i];
    }

    array[index] = value; //해당 인덱스에 저장
    size++;
  }

	//조회 메서드
  public int get(int index) {
    //TODO - IndexOutOfBound (size로 검사)
    return array[index];
  }

	//삭제 메서드
  public void remove(int index) {
    //TODO - IndexOutOfBound (size로 검사)
    if (index == size-1) {
      size--;
      return;
    }

		//삭제할 위치의 뒷쪽 데이터들을 모두 한칸씩 앞으로 이동
    for (int i = index; i < size; i++) {
      array[i] = array[i+1];
    }

    size--;
  }

	//배열 확장 메서드
  private void appendArray() {
		//확장된 새로운 배열 생성
    int[] newArray = new int[this.array.length * APPEND_SIZE];

		//기존 배열의 원소값 복사
    for (int i = 0; i < size; i++) {
      newArray[i] = this.array[i];
    }

		//대치
    array = newArray;
  }

  public int getSize() {
    return this.size;
  }
}

//테스트
public static void main(String[] args) {
  ArrayList arrayList = new ArrayList();
  arrayList.add(1);
  arrayList.add(2);
  arrayList.add(3);
  arrayList.add(4);

  System.out.println(arrayList.get(0)); // 1
  System.out.println(arrayList.get(1)); // 2
  System.out.println(arrayList.get(2)); // 3
  System.out.println(arrayList.get(3)); // 4

  System.out.println(arrayList.getSize()); // 4
}
```

<br/><br/>

## 연결리스트

연결리스트는 차례로 연결된 노드를 표현해주는 자료구조입니다.

**단방향 연결리스트**의 노드는 다음 노드를 참조하고, **양방향 연결리스트**의 노드는 다음 노드와 이전 노드를 모두 참조합니다.

연결리스트의 중요한 특징은 **‘배열과 달리 연결리스트에서는 특정 인덱스를 상수 시간에 접근할 수 없다’** 는 것입니다.  
즉, 리스트에서 K번째 원소를 찾고 싶다면 처음부터 K번 루프를 돌아야 합니다.

> 양방향의 경우, K가 뒷쪽에 더 가깝다면 뒤에서부터 탐색하도록 할 수 있습니다.
> 

연결리스트의 장점은 **‘리스트의 시작 지점에 데이터를 추가·삭제하는 시간복잡도가 `O(1)` 이라는 것’** 입니다.  
ArrayList의 경우, 첫 원소를 제외한 모든 원소를 한 칸씩 이동시켜야 하기 때문에 `O(n)` 시간이 소요됩니다.

**면접에서 연결리스트에 대해 이야기할 때, 단방향 연결리스트와 관련있는지 양방향 연결리스트와 관련있는지 확인해야 합니다.**

<br/>

### 헤드 노드로만 접근 가능한 연결리스트

아래는 헤드 노드로만 접근할 수 있는 단방향 연결리스트의 코드입니다.

```java
class Node {
	Node next = null;
	int data;

	public Node(data) {
		this.data = data;
	}

	void appendToTail(int d) {
		Node end = new Node(d);
		Node n = this;

		//마지막 노드를 찾을 때까지 반복
		while (n.next != null) {
			n = n.next;
		}

		//찾은 마지막 노드의 뒤에 새 노드 추가
		n.next = end;
	}
}
```

따로 연결리스트 클래스 없이, 오직 노드 클래스만 존재합니다.

연결리스트는 모든 노드가 연결되어 있기 때문에, 하나의 노드(헤드 노드)만 알고 있어도 전체 리스트에 접근하는 것이 가능합니다.

**매우 단순한 구조이지만, 이렇게 헤드 노드로 연결리스트에 접근하게 하는 것은 좋지 않습니다.**

여러 곳에서 헤드 노드를 참고하고 있다고 가정해볼까요?  
그리고 어느 한 곳에서 연결리스트의 가장 앞 노드를 제거했다고 해봅시다.  
**연결리스트에서는 가장 앞에 있는 노드(헤드 노드)가 제거되고 기존의 두번째 노드가 새로운 헤더 노드가 되었지만, 다른 나머지 곳에서는 여전히 기존의 헤드 노드를 참조하고 있게 됩니다.**

1. 객체 A와 B가 `Head 노드 1` 을 참조 중…
2. 객체 A가 연결리스트의 가장 앞 노드 (`Head 노드 1`)을 제거
3. 연결리스트의 두번째 노드가 새로운 헤드 노드 (`Head 노드 2`)가 됨
4. 하지만 객체 B는 여전히 `Head 노드 1` 을 참조 중…

**이를 해결하기 위해선, 하나의 LinkedList 클래스를 만들고 그 안에 Inner Class로 Node 클래스를 선언하는 것이 좋습니다.**  
그리고 LinkedList 클래스의 필드변수로 하나의 head 노드를 정의해서, 이 문제를 해결할 수 있습니다.

<br/>

### 단방향 연결리스트 구현

```java
public class LinkedList {
  private int size = 0;
  private Node head = new Node();

  public void add(String value) {
    Node newNode = new Node(value);
    Node currentNode = head;
    while (currentNode.getRear() != null) {
      currentNode = currentNode.rear;
    }
    currentNode.setRear(newNode);
    size++;
  }

  public void add(int index, String value) {
    if (index == size) add(value);

    Node newNode = new Node(value);

    if (index == 0) {
      Node rearNode = head.getRear();
      newNode.setRear(rearNode);
      head.setRear(newNode);
    } else {
      Node frontNode = getNode(index-1);
      newNode.setRear(frontNode.getRear());
      frontNode.setRear(newNode);
    }

    size++;
  }

  public String get(int index) {
    return getNode(index).value;
  }

  public void remove(int index) {
    if (index == 0) {
      head.setRear(head.getRear().getRear());
    } else {
      Node front = getNode(index-1);
      front.setRear(front.getRear().getRear());
    }

    size--;
  }

  public Node getNode(int index) {
    //Todo - OutOfIndex
    Node node = head;
    for (int i = 0; i <= index; i++) {
      node = node.rear;
    }
    return node;
  }

  private class Node {
    private Node rear;
    private String value;

    public Node() {}
    public Node(String value) {
      this.value = value;
    }
    public Node getRear() {
      return rear;
    }
    public void setRear(Node node) {
      rear = node;
    }
  }
}

//테스트
public static void main(String[] args) {
  LinkedList linkedList = new LinkedList();
  linkedList.add("a");
  linkedList.add("b");
  linkedList.add("c");

  System.out.println("index 0 : " + linkedList.get(0)); // a
  System.out.println("index 1 : " + linkedList.get(1)); // b
  System.out.println("index 2 : " + linkedList.get(2)); // c

  linkedList.remove(1);

  System.out.println("index 1 : " + linkedList.get(1)); // c

  linkedList.add(1, "inserted");

  System.out.println("index 0 : " + linkedList.get(0)); // a
  System.out.println("index 1 : " + linkedList.get(1)); // inserted
  System.out.println("index 2 : " + linkedList.get(2)); // c
}
```

<br/>

### 양방향 연결리스트 구현

```java
public class LinkedList {
  private int size = 0;
  private Node head = new Node();
  private Node tail = head;

  public void add(String value) {
    Node newNode = new Node(value);
    newNode.setFront(tail);
    tail.setRear(newNode);
    tail = newNode;
    size++;
  }

  public void add(int index, String value) {
    if (index == size) add(value);

    Node newNode = new Node(value);
    Node currentNode = getNode(index);

    if (currentNode.getFront() == null) {
      newNode.setRear(currentNode);
      currentNode.setFront(newNode);
    } else {
      newNode.setFront(currentNode.getFront());
      newNode.setRear(currentNode);
      currentNode.getFront().setRear(newNode);
      currentNode.setFront(newNode);
    }

    size++;
  }

  public String get(int index) {
    return getNode(index).value;
  }

  public void remove(int index) {
    Node target = getNode(index);

    if (target.getFront() != null) {
      target.getFront().setRear(target.getRear());
    }

    if (target.getRear() != null)
      target.getRear().setFront(target.getFront());
    else
      tail = target.getFront();
    size--;
  }

  public Node getNode(int index) {
    //Todo - tail부터 탐색 시작 최적화, OutOfIndex
    Node node = head;
    for (int i = 0; i <= index; i++) {
      node = node.rear;
    }
    return node;
  }

  private class Node {
    private Node front, rear;
    private String value;

    public Node() {}
    public Node(String value) {
      this.value = value;
    }

    public Node getFront() {
      return front;
    }
    public Node getRear() {
      return rear;
    }
    public void setFront(Node node) {
      front = node;
    }
    public void setRear(Node node) {
      rear = node;
    }
  }
}

//테스트
public static void main(String[] args) {
  Tmp linkedList = new Tmp();
  linkedList.add("a");
  linkedList.add("b");
  linkedList.add("c");

  System.out.println("index 0 : " + linkedList.get(0)); // a
  System.out.println("index 1 : " + linkedList.get(1)); // b
  System.out.println("index 2 : " + linkedList.get(2)); // c

  linkedList.remove(1);

  System.out.println("index 1 : " + linkedList.get(1)); // c

  linkedList.add(1, "inserted");

  System.out.println("index 0 : " + linkedList.get(0)); // a
  System.out.println("index 1 : " + linkedList.get(1)); // inserted
  System.out.println("index 2 : " + linkedList.get(2)); // c
}
```

<br/><br/>

## 스택과 큐

### 스택

스택은 FIFO 형식으로 동작하는 자료구조입니다.

스택에는 아래 4가지의 연산이 존재합니다.

- `pop()` : 스택에서 가장 위에 있는 항목을 제거한다.
- `push(item)` : item을 스택의 가장 윗부분에 추가한다.
- `peek()` : 스택의 가장 위에 있는 항목을 반환한다.
- `isEmpty()` : 스택이 비어 있을 때 true를 반환한다.

스택은 i번째 항목에 접근할 때, `O(i)` 시간이 소요됩니다.

하지만 데이터를 추가하거나 삭제하는 연산은 상수 시간에 가능합니다.

<br/>

### 스택을 언제 사용해야 하나요?

**스택이 유용한 경우는 재귀 알고리즘을 사용하는 경우입니다.**

- 재귀적으로 함수를 호출해야 하는 경우 → 임시 데이터를 스택에 Push
- 재귀 함수를 빠져 나와 퇴각 검색(backtrack)을 하는 경우 → 임시 데이터를 스택으로부터 Pop

<br/>

### 스택 구현 (연결리스트 활용)

스택은 연결리스트를 활용해서 간단히 구현할 수 있습니다.

```java
public class Stack {
	private StackNode top;
	
	public void push(String data) {
		StackNode newNode = new StackNode(data);
		if (isEmpty()) {
			top = newNode;
		} else {
			newNode.setNext(top);
			top = newNode;
		}
	}

	public String pop() {
		if (isEmpty()) throw new RuntimeException("stack is empty");
		String data = top.getData();
		top = top.next;
		return data;
	}

	public boolean isEmpty() {
		return top == null;
	}

	public String peek() {
		if (isEmpty()) throw new RuntimeException("stack is empty");
		return top.getData();
	}
	
	private static class StackNode {
		private String data;
		private StackNode next;
		public StackNode() {}
		public StackNode(String data) {
			this.data = data;
		}

		public StackNode getNext() {
			return this.next;
		}
		public void setNext(StackNode next) {
			this.next = next;
		}
	}
	public String getData() {
    return this.data;
  }
  public void setData(String data) {
    this.data = data;
  }
}

//테스트
public static void main(String[] args) {
  Tmp stack = new Tmp();

  stack.push("a");
  stack.push("b");
  stack.push("c");

  System.out.println(stack.pop()); //c
  System.out.println(stack.pop()); //b
  System.out.println(stack.pop()); //a

  stack.push("C");
  stack.push("B");
  stack.push("A");

  System.out.println(stack.pop()); //A
  System.out.println(stack.pop()); //B
  System.out.println(stack.pop()); //C
}
```

<br/>

### 큐

큐는 FIFO 구조로 동작합니다.

큐 연산의 종류는 아래와 같습니다.

- `enqueue(item)` : item을 리스트의 끝부분에 추가한다.
- `dequeue()` : 리스트의 첫번째 항목을 제거한다.
- `peek()` : 큐에서 가장 위에 있는 항목을 반환한다.
- `isEmpty()` : 큐가 비어 있을 때 true를 반환한다.

<br/>

### 큐는 언제 사용하면 좋나요?

**큐는 너비 우선 탐색(BFS)나 캐시를 구현하는 경우에 사용될 수 있습니다.**

<br/>

### 큐 구현 (연결리스트 활용)

스택과 마찬가지로 연결리스트를 활용해서 큐를 간단히 구현할 수 있습니다.

```java
public class Queue {
	private QueueNode front;
	private QueueNode rear;

	public void enqueue(String data) {
		QueueNode newNode = new QueueNode(data);
		if (isEmpty())
			front = newNode;
		else
			rear.setNext(newNode);
		rear = newNode;
	}

	public String dequeue() {
		if (isEmpty()) throw new RuntimeException("queue is empty");

		String data = front.getData();
		front = front.getNext();

		if (front == null)
			rear = null;

		return data;
	}

	public String peak() {
		if (isEmpty()) throw new RuntimeException("queue is empty");
		return front.getData();
	}

	public boolean isEmpty() {
		return front == null || rear == null;
	}

	private static class QueueNode {
		private String data;
		private QueueNode next;
		public QueueNode(String data) {
			this.data = data;
		}
		public String getData() {
			return this.data;
		}
		public void setData(String data) {
			this.data = data;
		}
		public QueueNode getNext() {
			return this.next;
		}
		public void setNext(QueueNode next) {
			this.next = next;
		}
	}
}

//테스트
public static void main(String[] args) {
  Queue queue = new Queue();

  queue.enqueue("a");
  queue.enqueue("b");
  queue.enqueue("c");

  System.out.println(queue.dequeue()); // a
  System.out.println(queue.dequeue()); // b
  System.out.println(queue.dequeue()); // c

  queue.enqueue("C");
  queue.enqueue("B");
  queue.enqueue("A");

  System.out.println(queue.dequeue()); // C
  System.out.println(queue.dequeue()); // B
  System.out.println(queue.dequeue()); // A
}
```

<br/><br/>

## 트리와 그래프

면접 지원자들이 가장 까다로워 하는 문제 중 하나가 바로 트리나 그래프 문제라고 저자가 말합니다.

트리·그래프의 동작방식이나 구현, 탐색 연산 등을 잘 알고 있어야 합니다.

> 이 포스팅만으로 트리·그래프의 상세한 이론적 부분에 대해서 알기엔 어렵습니다. (생략된 내용이 많습니다.)  
따라서 자세한 것은 본 블로그의 트리, 그래프 관련 포스팅을 참고하실 추천드립니다!
> 

<br/>

### 트리란?

트리는 재귀적인 구조로 이루어져 있습니다. 따라서 재귀적 설명법을 통해 트리를 정의하면 아래와 같습니다.

- 트리는 하나의 루트 노드를 갖는다.
- 루트 노드는 0개 이상의 자식 노드를 갖고 있다.
- 자식 노드 또한 0개 이상의 자식 노드를 갖고 있고, 이는 반복적으로 정의된다.

하나의 노드를 루트 노드로 삼는 서브트리가 존재할 수 있고, 동일한 노드는 상위 서브트리의 자식 노드로 취급될 수 있습니다. 이러한 특성 때문에, 재귀적인 구조를 갖는다고 이야기할 수 있습니다.

<br/>

### 트리의 특징

- 트리에는 사이클이 존재할 수 없다.
- 노드들은 특정 순서로 나열될 수도 있고, 그렇지 않을 수도 있다.
- 각 노드는 어떤 자료형으로도 표현 가능하다.
- 각 노드는 부모 노드로의 연결이 있을 수도 있고, 없을 수도 있다.

<br/>

### 노드(Node) 구현 예시

```java
class Node {
	public String data;
	public Node[] children;
}
```

- `data` : 노드의 데이터를 저장합니다.
- `Node[] children` : 해당 노드의 자식 노드들을 저장하는 포인터입니다.

<br/>

### 면접에서 출제되는 트리·그래프 문제를 풀 때, 유의해야 하는 부분

면접에서 마주치는 트리 혹은 그래프 문제들은 **대부분 세부사항이 모호하거나 가정 자체가 틀린 경우가 많다**고 저자가 이야기합니다.

따라서 아래 이슈들에 대해 유의하고, 필요하다면 명확하게 설명해달라고 요구하는 것이 좋습니다.

- 트리 vs 이진 트리
    - 이진 트리는 **각 노드가 최대 두 개의 자식을 갖는 트리**를 말합니다. 단순히 모든 노드의 자식 노드 개수가 2개 이하라면 이진 트리로 판단할 수 있습니다.
- 이진 트리 vs 이진 탐색 트리
    - 이진 탐색 트리는 **’모든 왼쪽 자식들의 값 ≤ n < 모든 오른쪽 자식들의 값’ 조건을 만족하는 이진 트리**입니다. 이진 트리를 기반으로 이 조건까지 만족시켜야 합니다.
    - 부모 노드의 값과 자식 노드의 값이 같을 때, 이것을 어떻게 처리할 것인지는 크게 상관하지 않아도 됩니다.  
    이진 탐색 트리가 같은 값을 가지면 안된다고 주장하는 곳도 있고, 아닌 경우도 존재합니다. 모두 이진 탐색 트리의 정의가 맞습니다.  
    **중요한 것은 면접관에게 같은 값을 어떻게 처리하겠다고 명확하게 이야기하는 것입니다.**
    - 저자는 많은 지원자들이 트리 문제를 만났을 때, 이진 탐색 트리일 것이라고 가정해버린다고 이야기합니다.  
    **따라서 이진 탐색 트리인지 면접관에게 확실히 물어보는 것이 필요합니다.**
- 균형 vs 비균형
    - 균형을 잡는다는 것이 **왼쪽과 오른쪽 부분 트리의 크기가 완전히 같게 만드는 것을 의미하지는 않습**니다.  
    `O(logN)` 시간에 삽입과 조회를 할 수 있을 정도로 균형을 맞춘다고 이해해도 좋습니다.
- 완전 이진 트리
    - 완전 이진 트리는 **말단 레벨을 제외한 모든 레벨의 노드가 꽉 채워져야 한다는 것을 의미**합니다.  
    **말단 레벨은 노드가 일부 비어있어도 되지만, 반드시 왼쪽에서 오른쪽으로 채워져**야 합니다.
- 전 이진 트리
    - 전 이진 트리는 **모든 노드의 자식이 없거나 정확히 두 개 있는 경우**를 말합니다.  
    즉, **자식이 하나만 있는 노드가 존재하면 안됩**니다.
- 포화 이진 트리
    - 포화 이진 트리는 **전 이진 트리면서, 완전 이진 트리인 경우**를 말합니다.  
    **모든 말단 노드는 같은 높이에 있어야 하고, 마지막 레벨의 노드 개수가 최대가 되어야 합니다.**
    - 포화 이진 트리의 조건이 까다롭기 때문에, 면접에서 자주 등장하지는 않습니다.

<br/>

### 이진 트리 순회

이진 트리를 순회하는 방법에는 세 가지가 있습니다.

- 중위 순회 : 왼쪽 → 본인 → 오른쪽
- 전위 순회 : 본인 → 왼쪽 → 오른쪽
- 후위 순회 : 왼쪽 → 오른쪽 → 본인

각 순회 방법에 대해 정리해보도록 하겠습니다.

> 각 순회 방식에 대해 상세한 정보가 필요하다면, [이 포스팅](https://taegyunwoo.github.io/interview/CS_Tree#%ED%8A%B8%EB%A6%AC-%EC%88%9C%ED%9A%8C-%EB%B0%A9%EC%8B%9D)을 참고하시기 바랍니다!
> 

- 중위 순회
    - ‘왼쪽 가지 → 현재 노드 → 오른쪽 가지’ 순서로 노드를 방문하고 출력하는 방법입니다.
    - **이진 탐색 트리를 이 방식으로 순회한다면, 오름차순으로 방문하게 됩니다.**
    
    ```java
    void traversal(Node node) {
    	if (node == null) return;
    
    	traversal(node.left); //왼쪽 재귀 호출
    	visit(node);          //본인 방문
    	traversal(node.right);//오른쪽 재귀 호출
    }
    ```
    
- 전위 순회
    - ‘현재 노드 → 왼쪽 가지→ 오른쪽 가지’ 순서로 노드를 방문하고 출력하는 방법입니다.
    - **가장 먼저 방문하게 될 노드는 언제나 루트노드입니다.**
    
    ```java
    void traversal(Node node) {
    	if (node == null) return;
    
    	visit(node);          //본인 방문
    	traversal(node.left); //왼쪽 재귀 호출
    	traversal(node.right);//오른쪽 재귀 호출
    }
    ```
    
- 후위 순회
    - ‘왼쪽 가지 → 오른쪽 가지 → 현재 노드’ 순서로 노드를 방문하고 출력하는 방법입니다.
    - **가장 마지막에 방문하게 될 노드는 언제나 루트노드입니다.**
    
    ```java
    void traversal(Node node) {
    	if (node == null) return;
    
    	traversal(node.left); //왼쪽 재귀 호출
    	traversal(node.right);//오른쪽 재귀 호출
    	visit(node);          //본인 방문
    }
    ```
    
<br/>

### 이진 힙(최소힙과 최대힙)

> 책에서 이진힙으로 최소힙만 다루기 때문에, 저 역시 최소힙만 이야기할 것입니다. 최소힙과 최대힙 모두 오름차순·내림차순 정렬에만 차이가 있고, 동작 원리는 동일하므로 참고하시기 바랍니다.
> 

최소힙(Heap)은 **각 노드의 원소가 자식들의 원소보다 작다는 특성**이 있습니다. 따라서 **루트 노드는 트리 전체에서 가장 작은 원소**가 됩니다.

핵심 연산으로는 아래 두 가지가 존재합니다.

- `insert` : 삽입
    1. 트리의 마지막 레벨의 왼쪽부터 새 노드를 삽입한다.
    2. 삽입된 새 노드를 부모 노드와 비교하고, 새 노드의 값이 부모 노드보다 더 작다면 서로 교체한다. (최소힙의 경우)
    3. 루트 노드에 도달하거나, 삽입된 새 노드가 부모 노드보다 더 크다면 교체작업을 중단한다.
    - 시간복잡도는 `O(logN) (N : 힙의 노드 개수)` 입니다.
- `extract_min` : 최소값 추출
    1. 루트 노드를 출력하고, 가장 마지막 노드(마지막 레벨 가장 우측 노드)를 루트 노드로 가져온다.
    2. ‘해당 루트 노드 값 < 좌측·우측 자식 노드 값’인 경우 : 좌측·우측 자식 노드 값 중 더 작은 노드와 교체  
    ‘해당 루트 노드 값 < 좌측 자식 노드 값’인 경우 : 좌측 자식 노드와 교체  
    ‘해당 루트 노드 값 < 우측 자식 노드 값’인 경우 : 우측 자식 노드와 교체
    3. 더이상 불가능할 때까지 2번을 반복한다.
    - 시간복잡도는 위와 동일하게 `O(logN) (N : 힙의 노드 개수)` 입니다.

<br/>

### 최소힙 구현 코드

힙은 완전 이진 트리의 형태이기 때문에, 단순히 배열로 구현할 수 있습니다.

- 노드 i의 부모 노드 Index : `i/2`
- 노드 i의 왼쪽 자식 노드 Index : `i*2`
- 노드 i의 오른쪽 자식 노드 Index : `i*2 + 1`

```java
public class MinHeap {
  public static final int MAX_SIZE = 50;
  private Integer[] nodeArray = new Integer[MAX_SIZE + 1]; //완전 이진 트리 배열 (index 0은 사용X)
  public int currentSize = 0;

  public void insert(int value) {
    nodeArray[++currentSize] = value; //일단 마지막에 새 노드 추가

    int childIdx = currentSize;
    int parentIdx = getParentIdx(childIdx); //새 노드의 부모 노드 index 구하기

    if (nodeArray[parentIdx] == null) return;

    //부모 노드와 교체
    while (nodeArray[parentIdx] > nodeArray[childIdx]) {
      int tmp = nodeArray[parentIdx];
      nodeArray[parentIdx] = nodeArray[childIdx];
      nodeArray[childIdx] = tmp;

      childIdx = parentIdx;
      parentIdx = getParentIdx(childIdx);
      if (nodeArray[parentIdx] == null) break;
    }
  }

  public int extractMin() {
    if (currentSize == 0) throw new RuntimeException("empty");

    int result = nodeArray[1];

    nodeArray[1] = nodeArray[currentSize]; //마지막 노드로 루트 노드 교체
    nodeArray[currentSize--] = null; //마지막 노드 제거

    int parentIdx = 1;
    int leftChildIdx = getLeftChildIdx(parentIdx);
    int rightChildIdx = getRightChildIdx(parentIdx);
    int minChildIdx = getMinChildIdx(leftChildIdx, rightChildIdx);

    if (minChildIdx == -1) return result;

    //자식 노드와 교체
    while (nodeArray[parentIdx] > nodeArray[minChildIdx]) {
      int tmp = nodeArray[parentIdx];
      nodeArray[parentIdx] = nodeArray[minChildIdx];
      nodeArray[minChildIdx] = tmp;

      parentIdx = minChildIdx;
      leftChildIdx = getLeftChildIdx(parentIdx);
      rightChildIdx = getRightChildIdx(parentIdx);
      minChildIdx = getMinChildIdx(leftChildIdx, rightChildIdx);
      if (minChildIdx == -1) break;
    }

    return result;
  }

  private int getLeftChildIdx(int currentIdx) {
    return currentIdx * 2;
  }

  private int getRightChildIdx(int currentIdx) {
    return currentIdx * 2 + 1;
  }

  private int getParentIdx(int currentIdx) {
    return currentIdx / 2;
  }

  private int getMinChildIdx(int leftChildIdx, int rightChildIdx) {
    int minChildIdx;

    if (nodeArray[leftChildIdx] != null && nodeArray[rightChildIdx] != null)
      minChildIdx = (nodeArray[leftChildIdx] <= nodeArray[rightChildIdx]) ? leftChildIdx : rightChildIdx;
    else if (nodeArray[leftChildIdx] != null)
      minChildIdx = leftChildIdx;
    else if (nodeArray[rightChildIdx] != null)
      minChildIdx = rightChildIdx;
    else
      return -1;

    return minChildIdx;
  }
}

//테스트
public static void main(String[] args) {
  MinHeap minHeap = new MinHeap();

  minHeap.insert(10);
  minHeap.insert(5);
  minHeap.insert(8);
  minHeap.insert(12);

  System.out.println(minHeap.extractMin()); // 5
  System.out.println(minHeap.extractMin()); // 8
  System.out.println(minHeap.extractMin()); // 10
  System.out.println(minHeap.extractMin()); // 12
}
```

<br/>

### 트라이

트라이는 n-차 트리의 변종으로, 각 노드에 문자를 저장하는 자료구조입니다.

**‘\* 노드’** 는 널 노드라고도 불리며, 단어의 끝을 나타낼때 사용됩니다.  
예를 들어, a → b → c → \* 이라면 ‘abc’라는 단어가 존재하는 것을 알 수 있습니다.

‘\* 노드’ 을 사용하는 것 대신에, **각 노드에 boolean 필드를 선언**해두고 이 필드값이 true일 때 하나의 단어가 있다고 표현할 수도 있습니다.

트라이에서 각 노드는 **‘1’ ~ ‘ALPHABET_SIZE + 1’ 개의 자식 노드**를 가질 수 있습니다.  
즉, 각 노드는 **‘알파벳’ + ‘\* 노드’ 만큼의 자식**을 가질 수 있습니다.

트라이는 접두사를 빠르게 찾기 위한 자료구조입니다.  
트라이 구조에서 원하는 접두사를 찾는 시간복잡도는 `O(k) (k : 찾을 접두사의 길이)` 입니다.

> 트라이에 대한 자세한 내용은 [이 포스팅](https://taegyunwoo.github.io/interview/CS_Trie)을 참고하세요!
> 

<br/>

### 트라이를 언제 사용하면 좋을까?

트라이는 접두사나 단어를 찾을 때, 매우 유용합니다.

따라서 **단어 집합을 이용하는 문제에서 트라이를 사용해 최적화**할 수 있습니다.

<br/>

### 그래프

트리는 그래프의 한 종류입니다.

- 트리 : 사이클이 없는 하나의 연결 그래프
- 그래프 : 노드와 간선을 하나로 모아 놓은 것

<br/>

### 그래프의 특성

그래프는 아래와 같은 특징을 갖습니다.

- 그래프의 간선은 방향성이 있을 수도 있고, 없을 수도 있다.  
(있는 경우 → 단방향) (없는 경우 → 양방향)
- 여러 개의 고립된 부분 그래프로 구성될 수 있다.  
모든 노드끼리 연결된 그래프는 **‘연결 그래프’** 라고 한다.
- 그래프에는 사이클이 존재할 수도 있고, 없을 수도 있다.  
사이클이 없는 그래프는 **‘비순환 그래프’** 라고 한다.

<br/>

### 그래프 구현 방식

그래프 구현 방식에는 두 가지가 있습니다.

- **인접 리스트**
    - 가장 일반적인 그래프 구현 방식
    - 각 노드가 어떤 노드와 인접한지 저장
    - 트리와는 달리 Graph 라는 클래스가 필요하다.  
    모든 노드가 연결되어 있다는 것을 보장할 수 없기 때문이다.
    
    ```java
    //그래프 클래스 : 그래프의 모든 노드 저장
    class Graph {
    	public Node[] nodeAry; //nodeAry[i] : i번째 노드의 데이터와 인접한 노드 정보
    }
    
    //노드 클래스 : 자신과 인접한 모든 노드 정보를 가지고 있음
    class Node {
    	public String data;
    	public Node[] nearNodes;
    }
    ```
    
- **인접 행렬**
    - 2차원 배열을 통해 표현
    - `graph[i][j] == true` 라면, `노드 i → 노드 j` 간선이 존재한다는 의미
    - 무방향 그래프(혹은 양방향 그래프)라면, 대칭행렬이 된다.

<br/>

### 인접 리스트 vs 인접 행렬

- 인접 리스트 : 특정 노드에 인접한 노드를 찾기 쉽다.  
(`nodeAry[i] : 노드i와 인접한 노드 정보만 담겨 있음`)
- 인접 행렬 : 특정 노드에 인접한 노드를 찾기 위해, 한 배열의 모든 원소를 찾아봐야 한다.  
(`graph[i] : 노드 i와 인접하다면 true 아니면 false이기 때문에, 모든 원소를 찾아봐야함`)

<br/>

### 그래프 탐색

그래프를 탐색하는 일반적인 방법 두 가지로, **‘깊이 우선 탐색(DFS)’** 과 **‘너비 우선 탐색(BFS)’** 가 존재합니다.

- DFS : 특정 노드에서 시작해서 한 분기를 완전히 탐색하고, 다음 분기로 넘어가는 방식
- BFS : 특정 노드에서 시작해서 인접한 노드를 먼저 탐색하는 방식

**만약 두 노드 사이의 최단 경로나 임의의 경로를 찾고 싶다면 BFS를 사용하는 것이 일반적입니다.**

```
[문제]
지구상에 존재하는 모든 친구 관계를 그래프로 표현한 뒤,
'Ash'와 'Vanessa' 사이에 존재하는 최단 경로를 찾아라.

[DFS로 해결시]
Ash -> Brian -> Carleton -> Davis -> Eric -> ...
위 케이스처럼 특정 분기를 계속 파고들기 때문에, 존재하는 모든 경로를 탐색해야할 수 있다.

[BFS로 해결시]
Ash와 인접한 사람부터 탐색해나간다.
역시 많은 친구들을 순회하겠지만, 반드시 필요한 경우가 아닌 경우 더 먼 관계에 있는 친구들을 살펴보진 않을 것이다.
즉 DFS에 비해 더 빠르게 찾을 수 있다.
```

<br/>

### DFS 구현

위에서 살펴본 **트리 순회(중위·전위·후위)는 모두 DFS의 한 종류**입니다.

트리 순회와 DFS의 가장 큰 차이점은 **‘방문한 노드를 체크해야 한다는 것’**입니다.

```java
void dfs(Node currentNode) {
	visit(currentNode); //노드 방문
	currentNode.visited = true; //방문 체크

	for (Node nearNode : currentNode.nearNodes) {
		if (nearNode.visit) continue; //방문한 노드라면 방문 X
		dfs(nearNode); //재귀를 통해 DFS 구현
	}
}

class Node {
	public String data;
	public boolean visited;
	public Node[] nearNodes;
}
```

<br/>

### BFS 구현

BFS는 큐를 활용하면 간단히 구현할 수 있습니다.

```java
void bfs(Node startNode) {
	Queue<Node> queue = new Queue<>();

	visit(startNode); //시작노드 방문
	startNode.visited = true; //시작노드 방문 처리
	queue.enqueue(startNode); //큐에 저장

	while (!queue.isEmpty()) {
		Node currentNode = queue.dequeue(); //큐에서 반환
		for (Node nearNode : currentNode.nearNodes) {
			if (nearNode.visited) continue; //방문한 노드라면 방문X
			visit(nearNode); //인접노드 방문
			nearNode.visited = true; //인접노드 방문 처리
			queue.enqueue(nearNode); //큐에 저장
		}
	}
}
```

<br/><br/>

## 정리하며...
지금까지 면접에서 자주 마주치게 되는 자료구조에 대해 알아봤습니다.

직접 자료구조를 종이에 코딩하여 구현해보고 테스트하며 학습하면 효과적이라고 저자가 설명했습니다.  
다소 시간이 걸리더라도, 직접 해본 것과 아닌 것의 이해도 차이가 분명할 것 같습니다!

면접을 대비하기 위해 자료구조를 확실히 알아두는 것도 좋지만, 개발의 기본적인 지식이고 활용되는 경우가 많으니 튼튼한 기초 체력을 키워봅시다.