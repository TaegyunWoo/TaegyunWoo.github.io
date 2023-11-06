---
category: React
tags: [React]
title: "[React] 컨테이너, 프레젠테이셔널 컴포넌트"
date: 2023-11-06 17:50:00 +0900
lastmod: 2023-11-06 17:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

React 애플리케이션을 구축하다 보면 하나의 컴포넌트에 책임이 너무 많아지게 되고, 유지보수가 어려워지는 시점이 옵니다. 이번 글에서는 React 코드를 구성하는 데 도움이 되는 **프로그래밍 패턴**에 대해 알아보겠습니다.

지금까지 React에 대해 알아본 내용은 아래 글을 참고하세요.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**
5. **[[React] React 애플리케이션 시작하기](https://taegyunwoo.github.io/react/Tech_React_ReactStructure)**
6. **[[React] 컴포넌트 상호작용 (props)](https://taegyunwoo.github.io/react/Tech_React_Props)**
7. **[[React] Hook - State](https://taegyunwoo.github.io/react/Tech_React_StateHook)**
8. **[[React] Hook - Effect](https://taegyunwoo.github.io/react/Tech_React_EffectHook)**

<br/><br/>

## 컨테이너 컴포넌트와 프레젠테이셔널 컴포넌트

컴포넌트의 복잡성을 줄이기 위해, 컴포넌트를 여러 개의 간단한 컴포넌트로 나눌 수 있습니다. 그렇다면 컴포넌트를 어떻게 분해해야 할까요?

복잡한 컴포넌트를 `상태 저장(컨테이너) 컴포넌트`와 `상태 비저장(프레젠테이셔널) 컴포넌트`로 분할할 수 있습니다. 여기서 `상태 저장 컴포넌트`는 **복잡한 State 또는 로직을 관리**하고, `상태 비저장 컴포넌트`는 **JSX만 렌더링**합니다.

다시 말해, **컴포넌트의 기능적인 부분들(State 유지, props를 기반으로 계산 수행 등)**은 `상태 저장 컴포넌트` 즉, `컨테이너 컴포넌트`로 분리될 수 있습니다.

이 `컨테이너 컴포넌트`는 **상태를 유지하고(생성 및 업데이트)**,이를 **props를 통해 `상태 비저장(프레젠테이셔널) 컴포넌트` 에 전달**하는 일을 담당합니다. 이를 통해, ‘로직을 수행하는 컴포넌트’와 ‘ui를 보여주는 컴포넌트’를 분리할 수 있습니다.

<br/>

- **프레젠테이셔널 컴포넌트**
  - 사용자가 직접 보고, 조작하는 컴포넌트  
    (ui와 관련 있습니다.)
  - **State를 직접 조작하지 않**으며, 컨테이너 컴포넌트가 내려준 prop의 함수에 의해 state를 변경합니다.  
    그에 따라 `useState`, `useCallback`, `dispatch`등 **State와 관련된 Hook은 존재하지 않**습니다.
  - State를 거의 가지지 않으며, State를 가진다면 데이터에 관한것이 아닌 ui 상태에 관한 것입니다.
  - 프레젠테이셔널 컴포넌트는 항상 **컨테이너 컴포넌트에 의해 렌더링**되므로, **자체적으로 렌더링되어서는 안 됩니다.**
- **컨테이너 컴포넌트**
  - **어떻게 동작**하는지, **어떤 로직**을 수행하는지에 관련있습니다.
  - 스타일을 사용하지 않습니다.
  - **프레젠테이셔널 컴포넌트를 호출함**으로써 실제 보여질 화면을 결정합니다.
  - 데이터와 데이터 조작에 관한 함수를 만들어서, **프레젠테이셔널 컴포넌트에 제공**합니다.  
    이때, **props를 통해 전달**합니다.

추가로 프레젠테이셔널 컴포넌트가 State를 유지하지 않는다고 해서, **동적으로 변경되지 않다는 의미는 아니라는 점**을 기억해주세요. State와 마찬가지로 **props의 변경(부모 컴포넌트로부터 새 props를 전달받는 경우)도 재렌더링을 수행한다는 점**을 알아둡시다.

> props에 대한 자세한 이야기는 [이전 글](https://taegyunwoo.github.io/react/Tech_React_Props#props%EB%9E%80)에서 확인할 수 있습니다.

<br/>

### 예시 코드

```jsx
//컨테이너
function Container() {
  const [isActive, setIsActive] = useState(false);

	return (
	  <>
	    <Presentational active={isActive} toggle={setIsActive}/>
	  </>
  );
}

//프레젠테이셔널
function Presentational({ active, toggle }) { //props로 전달받음
  return (
    <h1>Engines are {active}</h1>
    <button onClick={() => toggle(!active)}>Engine Toggle</button> //전달받은 state setter 함수 사용
  );
}
```

위 코드를 살펴보면 아래와 같습니다.

- **컨테이너 컴포넌트**에서 **State를 사용**했습니다.  
  그리고 **프레젠테이셔널 컴포넌트를 return**할 뿐, **UI를 위한 그 어떤 로직도 포함되지 않**습니다.
- 프레젠테이셔널 컴포넌트는 **props**를 통해, State와 Setter 함수를 전달받습니다.  
  **UI 외 구체적인 로직을 전혀 포함하고 있지 않**습니다.
- 만약 ‘컨테이너 컴포넌트의 State’를 ‘프레젠테이셔널 컴포넌트’에서 업데이트하고 싶다면, props로 전달받은 **Setter 함수를 사용**합니다.  
  (다만, Setter 함수를 직접 호출해서 사용하는 것이 아닌, 콜백함수 등으로 UI에 등록합니다.)

그리고 아래와 같이 동작하게 됩니다.

1. 만약 사용자가 버튼을 클릭하면, `() => toggle(!active)` 콜백함수가 실행됩니다.
2. 해당 콜백함수에 의해, **컨테이너 컴포넌트의 State가 변경**됩니다.
3. 컨테이너 컴포넌트의 State가 변경되었으므로, **컨테이너 컴포넌트가 재렌더링**됩니다.
4. 따라서 아래 코드가 재호출되고, **프레젠테이셔널 컴포넌트에 새로운 props를 전달**합니다.

   ```jsx
   return (
     <>
       <Presentational active={isActive} toggle={setIsActive} />
     </>
   );
   ```

5. 프레젠테이셔널 컴포넌트에 새로운 props가 전달됐기 때문에, **프레젠테이셔널 컴포넌트도 재렌더링**됩니다.

<br/><br/>

## 정리하며

지금까지 React에서 세부 로직과 UI 부분을 어떻게 분리해야 하는지 알아봤습니다.

내용을 정리해보면 아래와 같습니다.

- 애플리케이션의 복잡성을 줄이고 유지보수성을 향상시키기 위해, 하나의 복잡한 컴포넌트를 `컨테이너 컴포넌트` 와 `프레젠테이셔널 컴포넌트` 로 분리할 수 있습니다.
- `컨테이너 컴포넌트` 는 State를 갖고 세부로직을 담당합니다.
- `프레젠테이셔널 컴포넌트` 는 오직 UI에만 집중합니다.

이처럼 간단하지만 효과적인 프로그래밍 패턴을 사용해서, 더 나은 애플리케이션 구조를 갖을 수 있다는 사실을 기억하며 글 마치도록 하겠습니다.

혹시 잘못된 내용이 있다면 알려주세요. 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
- [https://kyounghwan01.github.io/blog/React/container-presenter-dessign-pattern/#gist-예제](https://kyounghwan01.github.io/blog/React/container-presenter-dessign-pattern/#gist-%E1%84%8B%E1%85%A8%E1%84%8C%E1%85%A6)
