---
category: Interview
tags: [코딩인터뷰 완전분석]
title: "[코딩인터뷰 완전분석] 면접 대비 정리 - 객체 지향 설계"
date:   2023-04-29 18:30:00 +0900
lastmod : 2023-04-29 18:30:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

> 이 글은 게일 라크만 맥도웰 저자의 ‘코딩인터뷰 완전분석’을 읽으며, 깨달은 사실과 내용을 정리하는 글입니다.  
자세한 내용이 궁금하다면 [http://www.yes24.com/Product/Goods/44305533](http://www.yes24.com/Product/Goods/44305533) 에서 책을 구매하시면 좋겠습니다.
> 

면접에서 등장하는 객체 지향 설계 관련 질문은 ‘지원자가 깔끔하고, 유지보수가 가능한 객체 지향적 코드를 만드는 방법을 이해하고 있는지 확인’하기 위함입니다.

이런 질문들은 디자인 패턴들을 쏟아내도록 요구하는 것이 아님을 기억합시다!

그럼 이런 문제를 어떻게 접근하고 해결해야 하는지 알아볼까요?

<br/><br/>

## 접근법

### 1. 모호성 해소 : 요구사항을 파악하자

객체 지향 설계에 관한 질문을 받으면, **누가 그것을 사용할 것인지, 어떻게 사용할 것인지에 대한 질문을 던져야 합니다.**  
질문에 따라서, 육하원칙에 따른 질문을 던져야 할 때도 있습니다.

객체 지향 설계 (OOD) 관련 문제들은 **대개 고의적으로 모호성을 띄고 있다**고 저자가 말합니다.

왜냐하면, 면접관들은 지원자가 **스스로 가정을 만들어내고 질문을 통해, 명확히 해나가는 과정을 살펴보고 싶어하기 때문**입니다.

**따라서 질문을 통해, 문제의 모호성을 제거하는 것이 중요합니다.**

> [예시]  
질문 : 커피메이커를 설계해보세요.  
모호성 : 카페용인지 가정용인지? , 사용자가 누구인지?(장애인, 노인, 전문가, 일반인)
> 

<br/>

### 2. 핵심 객체 설계 : 시스템에 필요한 객체를 파악하자

위 단계에서 무엇을 설계할 것인지 이해했으면, 이제 **시스템에 넣을 ‘핵심 객체’가 무엇인지 생각해봐야 합니다.**

> [예시 - 식당을 객체 지향적으로 설계]  
핵심 객체 : Table, Guest, Host, Order, Menu 등
> 

<br/>

### 3. 객체간 관계 분석 : 핵심 객체간의 관계를 설정하자

핵심 객체를 어느정도 결정했다면, **이 핵심 객체 사이의 관계를 분석해야 합니다.**

- 어떤 객체가 어떤 객체에 속해 있는가?
- 다른 객체로부터 상속 받아야 하는 객체가 존재하는가?
- 다대다·일대다·일대일 관계인가?

> [예시 - 식당을 객체 지향적으로 설계]  
\- Host와 Server는 Employee 를 상속받는다.  
\- Party는 Guest 배열을 갖는다.  
\- 각 Table은 Party를 하나만 갖지만, Party는 여러 Table을 가질 수 있다.
> 

<br/>

### 4. 행동 분석 : 객체의 행동을 분석하고 정의하자

이제 설계의 골격은 어느정도 잡혔을 것입니다. 이제 남은 일은 **객체가 수행해야 하는 핵심 행동들에 대해서 생각하고, 이들이 어떻게 상호작용** 해야하는지 따져보는 것입니다.

이때 깜빡하고 잊은 객체를 추가할 수도 있고, 설계가 변경될 수도 있습니다.

> [예시 - 하나의 Party가 입장할 때, Host의 핵심 행동]  
\- Host는 식당에 자리가 있으면 해당 Party에게 Table을 배정한다.  
\- 자리가 없다면 대기팀 명단에 등록한다.  
\- 한 Party가 식사를 마치면, 대기팀 명단 최상단에 있는 Party에게 해당 Table을 배정한다.


<br/><br/>

## 디자인 패턴

가끔 면접에서 디자인 패턴과 관련된 질문이 나올 수도 있습니다.

하지만 면접관이 평가하는 것은 ‘지원자의 지식’이 아니라 ‘지원자의 능력’이기 때문에, **일반적인 경우 디자인 패턴은 보통 면접 범위 외로 칩니다.**

하지만 ‘싱글톤’이나 ‘팩토리 메서드’ 같은 디자인 패턴은 알아 두면, 면접에서 유용하게 사용될 수 있다고 합니다.

추가로 디자인 패턴 문제를 마주했을 때 유의해야 하는 점은, **‘해당 문제에 가장 적합한 디자인 패턴을 찾으려고 하는 행위’를 지양**해야 한다는 것입니다.  
즉, 이미 존재하는 디자인 패턴 중 그나마 적절한 것을 선택하는 것을 피해야 합니다.  
필요하다면 그 문제에 적합한 디자인 패턴을 직접 만들면 됩니다.

<br/>

### 싱글톤 패턴

싱글톤 패턴은 **어떤 클래스가 오직 하나의 객체만을 갖도록 하고, 프로그램 전반에 걸쳐 그 객체 하나만 사용되도록 보장하는 패턴**입니다.

```java
public class MyClass {
	private static MyClass instance; //사용될 객체가 저장될 static 변수 (private)

	//생성자를 감춰둔다.
	private MyClass() {...}

	//싱글톤 객체를 반환 메서드
	public static MyClass getInstance() {
		if (instance == null) {
			instance = new MyClass();
		}
		return instance;
	}
}
```

이러한 싱글톤 패턴의 단점은 ‘다형성 활용의 불가능’과 ‘단위 테스트의 어려움’이 있습니다.

> 보다 자세한 내용은 아래를 참고하시면 좋습니다.  
[Spring과 싱글톤] : [https://taegyunwoo.github.io/spring-core/SPRING_Singleton](https://taegyunwoo.github.io/spring-core/SPRING_Singleton)  
[싱글톤의 단점] : [https://zereight.tistory.com/1222](https://zereight.tistory.com/1222)
> 

<br/>

### 팩토리 메서드

팩토리 메서드는 **어떤 클래스의 객체를 생성하기 위한 인터페이스를 제공하되, 하위 클래스에서 어떤 클래스를 생성할지 결정할 수 있도록 도와주는 패턴**입니다.

```java
public abstract class CardGame {
	public abstract void play();

	//팩토리 메서드
	public static CardGame createCardGame(GameType type) {
		switch (type) {
			case GameType.Poker :
				return new PokerGame();
			case GameType.OneCard :
				return new OneCardGame();
			case GameType.BlackJack :
				return new BlackJackGame();
		}
	}
}

public class PokerGame extends CardGame {
	@Override
	public void play() {
		//...
	}
}

public class Main {
	public static void main(String[] args) {
		CardGame cardGame = CardGame.createCardGame(GameType.Poker);
		cardGame.play(); //PokerGame 클래스의 play 메서드 실행됨
	}
}
```

<br/><br/>

## 마무리하며
이번에는 면접에서 출제될 수 있는 객체 지향적 설계 문제를 어떻게 접근해서 해결해야하는지와, 알아두면 좋은 디자인 패턴 2가지에 대해 알아봤습니다.

저자가 디자인 패턴은 일반적인 경우 잘 출제되지 않는다고 했지만, 제출한 이력서와 자기소개서에 관련 내용이 있다면 자세히 알아볼 필요가 있을 것 같다고 생각합니다. (제가 그런 케이스입니다...)

OOD 관련 문제에 접근하는 절차를 잘 기억해두고, 몇개의 연습문제를 풀어서 연습하는 것이 좋겠습니다.