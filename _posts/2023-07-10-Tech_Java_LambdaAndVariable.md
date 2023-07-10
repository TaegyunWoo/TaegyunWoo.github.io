---
category: Tech
tags: [Tech]
title: "왜 자바 람다는 final 지역변수를 사용해야할까?"
date:   2023-07-10 18:40:00 +0900
lastmod : 2023-07-10 18:40:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

이번 포스팅에서는 자바의 람다식은 어째서 final 변수를 사용해야 하는지에 대해 정리해보고자 합니다.

얼마전 면접에서 관련 질문을 받게 되었는데 제대로 답변하지 못한 아쉬움이 있어, 이번 기회에 확실히 짚고 넘어가보려고 합니다.

<br/><br/>

## 결론부터 이야기하자면…

왜 자바에서 람다식은 final 변수를 사용해야하는지, 그 결론부터 이야기하자면 아래와 같습니다.

- 람다식의 종류 중, `Non-local Capturing Lambda` 에서 사용되는 **외부 지역변수**는 반드시 `final` 이거나, `Effectively final` 이어야 한다.
- 왜냐하면, **외부 지역변수를 동기화할 방법이 없기 때문**이다.

그럼 왜 이런 결론이 도출되는지, 지금부터 자세히 알아보겠습니다.

<br/><br/>

## 람다의 종류

먼저 람다의 종류부터 시작해보죠.

람다는 어떻게 사용되느냐에 따라서, 여러 종류로 구분할 수 있습니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled.png)

- `Capturing Lambda`
    - 외부 변수를 사용하는 람다식을 의미합니다.
    - 외부 변수란, 클래스·인스턴스·지역변수처럼 람다식 외부에 선언된 변수를 의미합니다.
- `Non-capturing Lambda`
    - 외부 변수를 사용하지 않는 람다식을 의미합니다.
- `Local Capturing Lambda`
    - **사용하는 외부 변수가 지역 변수**인 람다식을 의미합니다.
    - 이때 람다식은 **외부 지역변수의 값을 복사해와서 사용**합니다.
    - 사용된 외부 지역변수는 반드시 `final` 이거나 `Effectively final` 이어야 합니다.
- `Non-local Capturing Lambda`
    - 사용하는 외부 변수가 지역 변수가 아닌 람다식을 의미합니다.
    - 외부 변수의 값을 복사하지 않고, 그대로 참조하여 사용합니다.

위 그림과 설명으로, 람다식의 종류는 충분히 이해할 수 있을겁니다.

우리가 이번 포스팅에서 집중할 것은 바로 `Local Capturing Lambda` 입니다.

`Local Capturing Lambda` 는 외부 지역변수를 사용하는 람다식을 의미합니다. 그리고 여기에서 사용되는 외부 지역변수는 반드시 불변성을 가져야 합니다.

<br/><br/>

## `Local Capturing Lambda` 는 외부 지역변수의 값을 복사해서 사용한다.

그렇다면 다른 람다식, 특히 인스턴스 변수나 클래스 변수를 사용하는 `Non-local Capturing Lambda` 는 `final` 이 없는 변수를 사용해도 되는데, 왜 `Local Capturing Lambda` 만 불변하는 외부 지역변수를 사용해야할까요?

그 이유는, `Local Capturing Lambda` 의 특징에 있습니다.

**‘`Local Capturing Lambda` 는 외부 지역변수의 값을 복사해서 사용한다.’** 가 바로 그 특징입니다.

이런 특성 때문에, 어쩔 수 없이 불변성을 갖는 지역변수를 사용할 수 밖에 없습니다.  
(이에 대한 자세한 내용은 조금 있다가 나오니, 조금만 기다려주세요!)

그렇다면, 왜 굳이 값을 복사해서 사용하는지를 먼저 알아볼까요?

<br/>

### 이유 1) 지역변수는 Stack 영역에 임시로 관리된다.

**지역변수는 해당 메서드를 실행한 쓰레드의 Stack 영역에 할당**됩니다. 그래서 메서드 실행을 마치게 되면, **Stack 영역에서 지역변수가 해제**됩니다.

그래서 람다는 어쩔 수 없이, 외부 지역변수의 값을 복사해서 사용하게 됩니다.

만약, 어떤 메서드가 자신의 지역변수를 사용하는 람다식을 리턴한다고 가정해봅시다. 그리고 `Local Capturing Lambda` 가 외부 지역변수를 복사해서 사용하지 않고, 참조해서 사용한다고 가정해보겠습니다.

그러면 아래 그림과 같이, 나타날 것입니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%201.png)

쓰레드에 의해 실행된 Method 4가 람다식을 반환했고, 이 람다식은 JVM의 메모리 영역 중 Method Area에 적재되게 됩니다.  
왜냐하면, 자바 컴파일러에 의해서 람다식이 `private static` 메서드로 변환되기 때문입니다.

그리고 **해당 람다식이 Method 4의 지역변수를 참조**하게 됩니다.

Method 4가 람다식을 반환하고 실행을 마쳤으므로, **해당 쓰레드의 Stack 영역에서 해제**될 것입니다.

그럼 아래 그림처럼, **람다식이 해당 지역변수를 참조할 수 없는 문제**가 발생합니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%202.png)

위와 같은 문제로 인해서, `Local Capturing Lambda` 는 **외부 지역변수의 값을 복사**해서 사용합니다.

그럼 아래와 같이 문제를 해결할 수 있습니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%203.png)

그렇기 때문에, 내부적으로 `Local Capturing Lambda` 는 외부 지역변수의 값을 복사해서 사용합니다.

<br/>

### 이유 2) 지역변수를 관리하는 쓰레드와 람다를 실행하는 쓰레드가 다를 수 있다.

그리고 `Local Capturing Lambda` 가 외부 지역변수의 값을 복사해서 사용하는 또다른 이유가 있습니다.

그것은 바로, **‘지역변수를 관리하는 쓰레드’와 ‘람다를 실행하는 쓰레드’가 서로 다를 수 있다는 것**입니다.

**Stack 메모리 영역은 쓰레드 별로 독립적으로 가지고 있으며, 서로 다른 쓰레드가 각자의 Stack 영역을 참조할 수 없습니다.**

**따라서, ‘지역변수가 존재하는 쓰레드’와 ‘람다를 실행하는 쓰레드’가 서로 다른 경우, 지역변수를 참조할 수 없습니다.**

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%204.png)

위 그림을 보면, JVM의 메모리 영역 중 Method Area에 람다식이 저장되어 있게 됩니다.  
(다시 말하지만, 람다식은 컴파일러에 의해 `private static` 메서드로 변환됩니다. 그렇기 때문에, 람다식은 Method Area 영역에 할당됩니다.)

따라서 Method Area에 Thread B의 지역변수에 대한 참조값이 저장되게 되고, 이것을 Thread A가 사용하게 됩니다.

**이 말은 즉, Thread A가 Thread B의 Stack 영역을 참조하여 사용한다는 이야기이고, 이것은 다른 쓰레드의 Stack 영역을 참조할 수 없다는 원리를 위배하게 됩니다.**

따라서 아래 그림처럼 동작하게 됩니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%205.png)

즉 정리하자면, `Local Capturing Lambda` 는 아래와 같은 이유로 복사된 외부 지역변수의 값을 사용합니다.

- 지역변수는 Stack 메모리 영역에 존재하기 때문에, 람다식을 실행하는 시점에 외부 지역변수가 메모리에서 이미 해제되어 있을 수 있다.
- 서로 다른 쓰레드는 독립적인 Stack 메모리 영역을 갖고, 서로 참조할 수 없다.

그리고 바로 이런 특성 때문에, 불변성을 갖는 외부 지역변수만을 사용할 수 있습니다.

<br/><br/>

## 그럼 왜 불변하는 외부 지역변수만 사용할 수 있나?

이제 드디어 왜 불변 외부 지역변수만 사용할 수 있는지 이야기해볼 수 있겠군요.

그 이유는 **‘람다에서 사용되는 복사된 값과 실제 외부 지역변수의 값이 동일하다는 것을 보장할 수 없기 때문’** 입니다.

만약 ‘지역변수를 관리하는 쓰레드’와 ‘람다식을 실행하는 쓰레드’가 서로 다르고, 변경 가능한 외부 지역변수를 람다가 사용할 수 있다면, **람다에서 사용하는 복사된 값과 불일치가 발생**하게 됩니다.

불변하지 않는 외부 지역변수를 사용할 수 있다고 가정한 뒤, 아래 코드를 볼까요?

```java
public void myMethod() {
	int localVariable = 1; //지역변수 (불변X)

	//람다식
	Runnable lambda = () -> {
		System.out.println(localVariable); //변할 수 있는 외부 지역변수 사용
	};

	//새로운 쓰레드에서 람다 실행
	Thread newThread = new Thread(lambda);
	newThread.start();

	localVariable++; //Effectively final 제거 및 값 변경
}
```

위 코드를 그림으로 나타내면, 아래와 같습니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%206.png)

코드에서 람다식을 선언하고, `new Thread` 에서 람다식을 실행시키고 난 직후 상태는 위 그림과 같을 것입니다.

이때 그 다음 코드인 `localVariable++` 를 실행하면, 아래와 같이 상태가 변하게 됩니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%207.png)

만약 불변하지 않는 지역변수를 람다식에서 사용할 수 있다면, 위처럼 **값의 불일치가 발생**하게 됩니다.

**그러면 수행된 코드의 결과를 예측할 수 없어지게 되고, 결과를 알 수 없는 코드는 수행할 의미가 없어지게 되겠죠?**

그렇다면, 서로의 상태를 동기화시킬 방법은 없을까요?

네, 없습니다. 왜냐하면 위에서 설명했듯, **쓰레드는 각각 독립적인 Stack 영역을 가지고 있고, 다른 쓰레드의 Stack 영역을 참조하는 것은 불가능하기 때문**입니다.

그렇기 때문에, 아래처럼 외부 지역변수는 불변성을 가져야 합니다.

```java
//final을 명시하는 경우
public void myMethod() {
	final int localVariable = 1; //final 지역변수

	//람다식
	Runnable lambda = () -> {
		System.out.println(localVariable); //불변 외부 지역변수 사용
	};

	//새로운 쓰레드에서 람다 실행
	Thread newThread = new Thread(lambda);
	newThread.start();

	// localVariable++; 불가능
}

//Effectively final을 사용하는 경우
public void myMethod() {
	int localVariable = 1; //Effectively final 지역변수

	//람다식
	Runnable lambda = () -> {
		System.out.println(localVariable); //불변 외부 지역변수 사용
	};

	//새로운 쓰레드에서 람다 실행
	Thread newThread = new Thread(lambda);
	newThread.start();

	// localVariable++; 추가 조작이 없다면, final 변수로 취급 (Effectively final)
}
```

정리하자면, 불변성을 갖지 않는 외부 지역변수를 람다식에서 사용하는 경우 값의 불일치가 발생할 수 있고, 이 때문에 실행 결과를 예측할 수 없다고 할 수 있습니다.

그리고 서로 다른 쓰레드의 Stack 영역을 참조할 수 없기 때문에, 값의 불일치 문제를 해결할 수 있는 방법이 없어서, 무조건 람다에서 사용되는 외부 지역변수는 불변해야 합니다.

<br/><br/>

## 그렇다면, 람다식 내부에서 조작하는 경우에는?

`Local Capturing Lambda` 는 **외부 지역변수를 람다식 내부에서 조작할 수도 없**습니다.

람다식의 외부에서 조작을 하든, 내부에서 조작을 하든, 모두 **값의 불일치 문제가 발생하는 것은 마찬가지**입니다.

아래 코드를 볼까요?

```java
public void myMethod() {
	int localVariable = 1; //지역변수 (불변X)

	//람다식
	Runnable lambda = () -> {
		localVariable++; //외부 지역변수 값을 람다 내부에서 조작
		System.out.println(localVariable);
	};

	//새로운 쓰레드에서 람다 실행
	Thread newThread = new Thread(lambda);
	newThread.start();
}
```

위 코드를 실행시키면, 아래 그림과 같이 나타나게 됩니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%208.png)

이 경우, 복사된 값이 변경되었고 **값의 불일치 문제가 발생**하게 됩니다.

**그렇기 때문에, 람다의 내부·외부 모두 외부 지역변수의 값을 변경할 수 없습니다.**

<br/><br/>

## 그럼 왜 인스턴스 변수나 클래스 변수는 final이 아니더라도, 람다에서 사용할 수 있을까?

그 이유는 **람다식에서 클래스·인스턴스 변수를 사용할 때, 값을 복사해서 사용하지 않기 때문**입니다.

그렇기 때문에, 값의 불일치 문제를 해결할 수 있습니다.

아래 코드는 컴파일 에러가 발생하지 않고, 정상적으로 실행 가능합니다.

```java
static int classVar = 1; //클래스 변수
int instanceVar = 1; //인스턴스 변수

public void myMethod() {
	//인스턴스 변수를 조작하는 경우
  Runnable lambda1 = () -> {
    instanceVar++; //람다식 내부에서 인스턴스 변수 조작
    System.out.println("instanceVar : " + instanceVar);
  };
  instanceVar++; //람다식 외부에서 인스턴스 변수 조작

	//클래스 변수를 조작하는 경우
  Runnable lambda2 = () -> {
    classVar++; //람다식 내부에서 클래스 변수 조작
    System.out.println("classVar : " + classVar);
  };
  classVar++; //람다식 외부에서 클래스 변수 조작

  Thread threadA = new Thread(lambda1);
  Thread threadB = new Thread(lambda2);

  threadA.start(); //새 쓰레드에서 람다식1 실행
  threadB.start(); //새 쓰레드에서 람다식2 실행
}
```

위 코드를 실행하면, 아래와 같은 상태가 되게 됩니다.

![Untitled](/assets/img/2023-07-10-Tech_Java_LambdaAndVariable/Untitled%209.png)

**인스턴스 변수**는 **Heap 메모리 영역**에 할당되고, **클래스 변수**는 **Method Area 메모리 영역**에 할당됩니다.

여기서 Heap과 Method Area 영역은 모두 **서로 다른 쓰레드에서 접근할 수 있는 메모리 영역**입니다.

그리고 Stack 영역과는 다르게, **람다식을 수행할 때 해당 외부 변수들이 먼저 메모리에서 해제될 가능성이 없**습니다.

따라서 **여러 쓰레드에서 이 외부 변수들을 참조**를 할 수 있고, 그렇기 때문에 **람다식에서 값을 복사해서 사용할 필요가 없**습니다.

그러므로, 값의 불일치 문제를 해결할 수 있습니다. 물론 동시성 문제를 해결하기 위해서, `synchronized` 같은 키워드를 사용하긴 해야 하지만요.

<br/><br/>

## 정리하며

지금까지 람다에서 불변 외부 지역변수를 사용해야하는 이유에 대해 알아보았습니다.

다시 정리해보면 아래와 같습니다.

1. 람다 종류에는 `Local Capturing Lambda` 라는 것이 있고, 이것은 외부 지역변수를 사용하는 람다를 말한다.
2. `Local Capturing Lambda` 는 외부 지역변수의 값을 복사해와서 사용한다. 그 이유는 아래와 같다.
    - 지역변수는 쓰레드의 Stack 메모리 영역에 할당되기 때문이다.
    - 따라서 람다식을 반환하고 메서드가 종료되는 경우, 람다식이 참조하고 있는 지역변수가 Stack 영역에서 제거된다. 그러면 람다식이 해당 지역변수를 참조할 수 없다.
    - 그리고 쓰레드 간 서로의 Stack 영역을 참조할 수 없다.
3. 람다식이 외부 지역변수의 값을 복사해서 사용하기 때문에, 외부 지역변수가 불변하지 않는다면 값의 불일치 문제가 발생한다.
    - 그리고 이 문제를 해결할 수 있는 방법이 없다.  
    왜냐하면 쓰레드 간 서로의 Stack 영역을 참조할 수 없어서, 값을 동기화할 수 없기 때문이다.
4. 그렇기 때문에, 람다식이 사용하는 외부 지역변수는 무조건 불변해야한다.

이번 포스팅을 통해서, 람다식이 외부 지역변수를 사용할 때 왜 불변 변수만 사용할 수 있는지 이해하셨길 바라며, 글 마치도록 하겠습니다.

감사합니다.