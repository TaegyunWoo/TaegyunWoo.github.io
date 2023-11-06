---
category: React
tags: [React]
title: "[React] 컴포넌트 상호작용 (props)"
date: 2023-10-27 22:50:00 +0900
lastmod: 2023-10-27 22:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이번 시간에는 React의 컴포넌트들이 어떻게 상호작용하는지에 대해 알아보겠습니다.

지금까지 살펴본 내용은 아래와 같습니다.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**
5. **[[React] React 애플리케이션 시작하기](https://taegyunwoo.github.io/react/Tech_React_ReactStructure)**

<br/><br/>

## React 컴포넌트 개요

![Untitled](/assets/img/2023-10-27-Tech_React_Props/Untitled.png)

React에서 컴포넌트들은 위와 같이, **트리 구조**를 갖게 됩니다. 이를 참고해주세요!

이제부터 본격적인 내용을 다뤄봅시다.

<br/><br/>

## 컴포넌트 호출

### 다른 컴포넌트에서 컴포넌트 호출하기

```jsx
function PurchaseButton() {
  return (
    <button
      onClick={() => {
        alert("Purchase Complete");
      }}
    >
      Purchase
    </button>
  );
}

function ItemBox() {
  return (
    <div>
      <h1>50% Sale</h1>
      <h2>Item: Small Shirt</h2>
      <PurchaseButton />
    </div>
  );
}
```

위 코드를 보면, `ItemBox` 컴포넌트가 `PurchaseButton` 컴포넌트를 호출하는 것을 알 수 있습니다.

이렇게 특정 컴포넌트에서 다른 컴포넌트를 호출할 수 있다는 것을 알아둡시다.

<br/>

### 다른 js 파일의 컴포넌트 호출하기

물론 **다른 js 파일에 작성된 컴포넌트도 호출**할 수 있습니다. 이때 `import` 와 `export` 를 사용하면 됩니다.

예시 코드는 아래와 같습니다.

```jsx
// Component1.js
import React from "react";

function Component1() {
  return <h1>Component1</h1>;
}

export default Component1;

// Component2.js
import React from "react";
import Component1 from "./Component1.js";

function Component2() {
  return (
    <div>
      <Component1 />
    </div>
  );
}
```

<br/><br/>

## 컴포넌트 props

### props란?

React에서 컴포넌트가 다른 컴포넌트와 통신을 할 수 있습니다. 즉, **컴포넌트가 다른 컴포넌트에게 정보를 전달**할 수 있습니다.

그리고 **한 컴포넌트에서 다른 컴포넌트로 전달되는 정보**를 **props**라고 합니다. (props는 properties의 약자입니다)

**상위 컴포넌트는 props를 하위 컴포넌트에 전달**할 수 있지만, **그 반대의 경우는 불가능**합니다.

또한 props는 아래 데이터타입을 포함한 여러 타입이 될 수 있습니다.

- Numbers
- Strings
- Functions
- Objects

예시 코드는 아래와 같습니다.

```jsx
import React from 'react';

class ParentComponent extends React.Component {
  render() {
    return <ChildComponent prop1="Mike" prop2="piza">
  }
}

function ChildComponent(props) {
  return <h2>This is prop1: {props.prop1}. This is prop2: {props.prop2}.</h2>
}
```

`ParentComponent` 에서 `ChildComponent` 를 사용할 때, 속성으로 `prop1="Mike"` 와 `prop2="piza"` 를 넘겨줬습니다.

이렇게 넘어간 **prop 값들은 `ChildComponent` 함수의 파라미터에 아래와 같은 객체 타입으로 전달**됩니다.

```json
{
  "prop1": "Mike",
  "prop2": "piza"
}
```

이를 전달받은 `ChildComponent` 는 해당 값을 활용해, 본인의 컴포넌트를 구현할 수 있습니다.

<br/>

또한 props를 객체로 전달받기 때문에, 아래와 같이 구조분해 역시 가능합니다.

```jsx
import React from 'react';

class ParentComponent extends React.Component {
  render() {
    return <ChildComponent prop1="Mike" prop2="piza">
  }
}

function ChildComponent({prop1, prop2}) {
  return <h2>This is prop1: {prop1}. This is prop2: {prop2}.</h2>
}
```

**모든 컴포넌트에는 props가 존재**합니다.

컴포넌트의 props는 **객체**로, 해당 컴포넌트에 대한 정보를 보유합니다.

<br/>

추가로 알아두어야 하는 것은 **문자열이 아닌 값들은 모두 중괄호로 묶어서 전달해야 한다는 것**입니다.

```jsx
<Greeting myInfo={['top', 'secret', 'lol']} /> //배열을 전달하는 경우
<Greeting name="Frarthur" town="Flundon" age={2} haunted={false} />
```

<br/>

마지막으로 알아두어야 하는 것은 **React에서 일반적으로 props는 불변(Read only)으로 취급된다는 것**입니다.

> [setter 메서드에 의해서 변경되어야 하는 이유]  
> React의 컴포넌트는 아래 조건에서 재 렌더링됩니다.
>
> 1. props가 바뀔 때
> 2. state가 바뀔 때
> 3. 부모 컴포넌트가 리렌더링 될 때
> 4. this.forceUpdate로 강제로 렌더링을 트리거할 때

따라서 아래 코드는 에러가 발생합니다.

```jsx
//MyComponent 컴포넌트 객체의 props 필드값으로 {"text":"old"} 객체를 설정한다.
const myComponent = <MyComponent text="old" />;
myComponent.props.text = "new";
```

`TypeError: Cannot assign to read only property 'name' of object '#<Object>'` 오류를 뱉으며 프로그램이 죽게 됩니다.

참고로, `Object.getOwnPropertyDescriptor()` 함수로 해당 객체에 대한 정보를 확인해보면, `writable:  false` 로 설정되어 있는 것을 알 수 있습니다.

이제 props에 대한 기본적인 개념은 충분히 설명했습니다.

한번 실제로 사용해볼까요?

<br/>

### 이벤트 핸들러 함수를 props로 전달하기

props를 통해서 단순한 값을 전달할 수도 있지만, **특정 컴포넌트의 이벤트 핸들러를 외부에서 설정**할 때도 사용할 수 있습니다.

```jsx
// Button.js
import React from "react";

function Button(props) {
  return (
    <button onClick={props.onClick}>
      {" "}
      {/* props로 이벤트 핸들러 등록 */}
      Click me!
    </button>
  );
}

export default Button;
```

```jsx
// Talker.js
import React from "react";
import Button from "./Button";

function Talker() {
  function handleClick() {
    let speech = "";
    for (let i = 0; i < 10000; i++) {
      speech += "blah ";
    }
    alert(speech);
  }
  return <Button onClick={handleClick} />; //이벤트 핸들러로 talk 콜백 함수 전달
}

export default Talker;
```

이를 통해서, 좀 더 확장성있는 컴포넌트를 설계할 수 있습니다.

> [이름 규칙]  
> 관례적으로 이벤트 핸들러 콜백 함수는 `handle...` 로 설정하며, 이벤트 핸들러 콜백 함수가 담길 props 필드명은 `on...` 으로 설정한다는 것을 알아둡시다.

<br/>

### `props.children` 속성

모든 컴포넌트의 props 객체에는 `children`이라는 속성이 있습니다.

`props.children`은 **컴포넌트 열기 태그와 닫기 태그 사이에 있는 모든 것을 반환**합니다. 즉, `<MyComponent>` 와 `</MyComponent>` 사이에 존재하는 모든 정보를 반환하게 됩니다.

```jsx
// Example 1
<BigButton>
  I am a child of BigButton.
</BigButton>

// Example 2
<BigButton>
  <LilButton />
</BigButton>

// Example 3
<BigButton />

// Example 4
<BigButton>
	<LilButton1 />
	<LilButton2 />
</BigButton>

//BigButton
function BigButton(props) {
	console.log(props.children);
	return <div>props.children</div>;
}
```

위 코드에서 `BigButton` 컴포넌트의 `props.children` 은 아래와 같은 값을 갖게 됩니다.

- Example 1 : `I am a child of BigButton.`
- Example 2 : `<LilButton />`
- Example 3 : `undefined`
- Example 4 : `[<LilButton1 />, <LilButton2 />]`

컴포넌트에 **둘 이상의 자식**이 있는 경우, `props.children`은 **해당 자식을 배열로 반환**합니다. 그러나 **자식이 하나**만 있는 경우, `props.children`은 배열로 래핑되지 않고 **단일 자식을 반환**합니다.

<br/>

### `props` 기본값 설정하기

필요에 따라, 속성값이 넘어오지 않았을 때 사용할 기본값을 지정할 수도 있습니다.

- **방법 1**

  ```jsx
  function Example(props) {
    return <h1>{props.text}</h1>;
  }

  Example.defaultProps = {
    text: "This is default text",
  };
  ```

- **방법 2**
  ```jsx
  function Example({ text = "This is default text" }) {
    return <h1>{text}</h1>;
  }
  ```
- **방법 3**
  ```jsx
  function Example(props) {
    const { text = "This is default text" } = props;
    return <h1>{text}</h1>;
  }
  ```

위 3가지 방법 모두 `text` 필드가 비어있다면, 기본값 `This is default text` 를 가지게 됩니다.

<br/><br/>

## 정리

지금까지 살펴본 내용을 정리해보면 아래와 같습니다.

- 컴포넌트 인스턴스에 `attribute`을 제공하여 props을 전달하는 방법에 대해 알아봤습니다.
- `props.prop이름` 을 통해 전달된 `prop` 에 접근할 수 있습니다.
- 콜백함수를 props로 전달하고, 이벤트 핸들러로 등록할 수 있습니다.
- `props.children` 를 통해, 하위 정보에 접근할 수 있습니다.
- `props` 에 기본값을 할당할 수 있습니다.

약간은 복잡할 수 있지만, 차근차근 따라오면 충분히 이해할 수 있습니다.

혹시 잘못된 내용이 있다면 알려주세요! 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
