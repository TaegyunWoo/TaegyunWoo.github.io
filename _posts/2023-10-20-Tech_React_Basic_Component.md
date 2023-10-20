---
category: React
tags: [React]
title: "[React] 기본 컴포넌트 알아보기"
date: 2023-10-20 21:00:00 +0900
lastmod: 2023-10-20 21:00:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이전 글에서 React를 학습하기 위한 사전 지식으로 JSX와 가상 돔 객체에 대해 알아봤습니다.

이전 글은 아래를 참고해주세요.

- **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
- **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
- **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**

이번 시간에는 본격적으로 React에 대해서 알아보도록 하겠습니다. 물론 몇 개의 시리즈로 포스팅을 할 예정이고, 이번 글에서는 기본적인 React 컴포넌트에 대해서 설명하겠습니다.

<br/><br/>

## React 컴포넌트

하나의 React 애플리케이션은 **여러 컴포넌트들로 구성**됩니다.

컴포넌트란, **하나의 작업을 담당하는 작고 재사용 가능한 코드 덩어리**라고 생각하시면 됩니다. 여기서 작업에는 아래와 같은 것들이 포함됩니다.

- HTML 요소 렌더링
- 특정 데이터가 업데이트되었을 때, 재 렌더링

그 예시 코드는 아래와 같습니다.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";

function MyComponent() {
  return <h1>Hello world</h1>;
}

ReactDOM.createRoot(document.getElementById("app")).render(<MyComponent />);
```

굉장히 낯선 코드죠? 이제부터 이 코드를 이해할 수 있도록, 하나씩 개념을 잡아가봅시다.

<br/><br/>

## React 애플리케이션 구조

먼저 React 애플리케이션에 어떤 구조를 갖고 있는지부터 알아봅시다.

대표적인 아래 3가지 파일에 대해서 설명하겠습니다.

- `public/index.html`
- `src/index.js`
- `src/App.js`

<br/>

### `public/index.html`

`index.html` 은 **React가 동적으로 페이지를 렌더링할 때, 기본이 되는 HTML 파일**입니다.

즉, **동적으로 렌더링된 컴포넌트들이 담길 틀**이라고 생각하면 됩니다.

아래 `index.html` 코드를 볼까요?

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- 생략 -->
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--이 부분이 중요-->
    <!--
      생략
    -->
  </body>
</html>
```

일반적으로 `index.html` 은 위처럼 생겼습니다. (별로 특별한게 없죠?)

React는 이 `index.html` 파일의 코드 중, `<div id="root"></div>` 부분을 개발자가 원하는 화면으로 치환(렌더링)하게 됩니다.

> 기본적으로 React는 싱글 페이지 애플리케이션 (SPA) 방식으로 동작하는데, 이때 사용될 HTML이 바로 `index.html` 입니다.

<br/>

### `src/index.js`

`index.js` 는 앱의 진입지점이 됩니다.

이어서 아래 코드를 보겠습니다.

```jsx
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import * as serviceWorker from "./serviceWorker";
ReactDOM.render(
  <React.StrictMode>
    <App /> {/* 중요 */}
  </React.StrictMode>,
  document.getElementById("root") //중요
);
// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

위 코드에서 집중할 내용은 아래와 같습니다.

```jsx
ReactDOM.render(
  <React.StrictMode>
    <App /> //중요
  </React.StrictMode>,
  document.getElementById("root") //중요
);
```

위 코드를 통해, `index.html` 의 `<div id="root"></div>` 부분에 `App.js` 를 가져와 `<App />` JSX 요소로 동적 렌더링한다는 것을 알 수 있습니다.

정리하자면 아래와 같습니다.

- `src/index.js` 는 애플리케이션의 초기 설정을 수행하고, 루트 컴포넌트를 렌더링합니다.
- 한 번 `src/index.js`가 실행되고 루트 컴포넌트가 렌더링되면, 이후의 애플리케이션 상태와 동작은 사용자 상호작용 및 이벤트 처리에 따라 변경됩니다.  
  즉, 애플리케이션은 이후에 사용자와의 상호작용에 따라 다양한 이벤트를 처리하고 상태를 업데이트합니다.

<br/>

### `src/App.js`

이 파일이 바로, 우리가 정의할 컴포넌트가 위치하는 곳입니다.

```jsx
import React from "react";
import "bootstrap/dist/css/bootstrap.min.css";
import "./App.css";

function App() {
  return <div className="App">{/* 개발자 작성 */}</div>;
}
export default App;
```

<br/><br/>

## 모듈 Import

### `react` 모듈

우리는 React 애플리케이션을 만들기 위해서, React 라이브러리를 추가했었습니다.

그럼 이 라이브러리(모듈)을 사용하기 위해선, 아래처럼 작성해야 합니다.

```jsx
import React from "react";
```

`'react'` 라는 패키지에서 여러 메서드들을 `React` 라는 객체에 담는다는 뜻입니다.

그럼 이 `React` 객체를 통해서, 원하는 React 작업에 접근할 수 있겠죠?

<br/>

### `react-dom/client` 모듈

이번에는 React 내부에서 DOM 객체를 조작하기 위해서, `react-dom/client` 모듈을 가져오겠습니다.

```jsx
import ReactDOM from "react-dom/client";
```

그렇다면, 위에서 살펴본 `react` 와 `react-dom/client` 는 어떤 차이가 있을까요? 이는 아래와 같습니다.

- `react`
  - 컴포넌트 생성, JSX 작성 등을 수행할 수 있습니다.
  - DOM을 처리하는 관련 기능은 전혀 없습니다.
- `react-dom/client`
  - DOM을 조작하기 위한 작업을 수행할 수 있습니다.

즉, `react` 가 DOM을 조작하기 위해서는 관련 라이브러리를 반드시 가져와야 합니다.

<br/><br/>

## 컴포넌트 작성

컴포넌트는 React 애플리케이션을 구성하는 데 필요한 작은 블록들이라고 설명했었습니다.

그래서 여러 컴포넌트들을 적절히 활용해서, 하나의 React 애플리케이션을 만들 수 있는 것이죠.

그리고 보통 컴포넌트는 **JS의 함수를 사용해서 작성**합니다. 이것을 **함수형 컴포넌트**라고 합니다.

함수형 컴포넌트 형식으로 코드를 작성하는 것이 현대 React 애플리케이션의 표준이라고 할 수 있습니다.

함수로 컴포넌트를 작성하고 이 함수를 다양한 곳에서 사용함으로써, **동일한 모양의 컴포넌트를 쉽게 생성**할 수 있습니다.

함수형 컴포넌트는 아래처럼 작성하면 됩니다.

```jsx
function MyComponent() {
  return <h1>Hello world</h1>;
}
```

`return` 문에 우리가 이전에 배웠던 JSX 문법이 사용되었습니다.

즉, 해당 코드를 실행하면 해당 컴포넌트가 반환되게 됩니다.

<br/>

### 함수형 컴포넌트 이름 규칙

위 예시에서 `MyComponent` 함수의 첫글자가 대문자인 것을 확인할 수 있습니다.

**함수형 컴포넌트의 함수 이름은 반드시 대문자로 시작해야 합니다.** 또한 관례적으로, **파스칼 케이스를 사용**합니다.

> 파스칼 케이스: 첫문자를 포함한 중간 글자들이 대문자인 것  
> Ex) funtional component → FunctionalComponent

함수 이름의 시작이 대문자여야만, JSX 컴파일러가 React 컴포넌트임을 알 수 있습니다. 만약 소문자로 시작한다면, JSX 컴파일러는 해당 코드가 HTML 태그인 것으로 인식하게 됩니다.

```jsx
function MyComponent() {
  return <h1>Hello world</h1>;
}
function myComponent() {
  return <h1>Hello world</h1>;
}

<MyComponent /> //JSX 컴파일러가 <h1>Hello world</h1>로 번역한다.
<myComponent /> //JSX 컴파일러는 HTML 태그인 줄 알고, 번역하지 않는다.
```

<br/>

### 함수형 컴포넌트 본문

함수형 컴포넌트의 본문에는 **‘어떻게 JSX 요소를 만들어낼지’**에 대한 내용이 담깁니다.

그리고 반드시 **함수의 반환값은 JSX 요소**가 되어야 합니다.

<br/><br/>

## 컴포넌트 사용

이번에는 우리가 작성한 컴포넌트를 어떻게 사용해야 하는지에 대해 알아보겠습니다.

함수형 컴포넌트를 사용하려면, `<함수명 />` 형식으로 작성하면 됩니다.

`App.js` 파일이 아래와 같다고 가정해봅시다.

```jsx
//App.js
import React from "react";

function MyComponent() {
  return <h1>Hello world</h1>;
}

export default MyComponent;
```

위 컴포넌트를 생성하고 렌더링하려면, `index.js` 에서 `App.js` 를 import 하고 아래처럼 작성해야 합니다.

```jsx
//index.js
import React from "react";
import ReactDOM from "react-dom/client";

import MyComponent from "./App";

ReactDOM.createRoot(document.getElementById("app")).render(<MyComponent />);
```

`ReactDOM.createRoot(document.getElementById('app')).render(<MyComponent />);` 코드가 무엇을 뜻하는지 자세히 알아보겠습니다.

- `document.getElementById('app')`
  - `index.html` 에 정의된 id가 `app` 인 태그의 DOM 객체를 가져온다.
- `ReactDOM.createRoot(..)`
  - 첫 Parameter로 들어온 DOM 객체를 위한 Root 객체를 생성합니다. 이것을 루트 컨테이너(Root Container)라고 합니다.
  - 즉, 가상 DOM 객체의 Root 부분을 생성해 반환합니다.
- `ReactDOM.createRoot(..).render(<MyComponent />)`
  - `MyComponent` 컴포넌트를 생성하고, 해당 컴포넌트를 Root에 렌더링합니다. 실제로 화면에 해당 컴포넌트를 표시하게 됩니다.
  - 즉, `MyComponent` 컴포넌트를 생성해서 `ReactDOM.createRoot(..)` 으로 생성된 Root 객체의 하위 컴포넌트로 추가합니다. 그리고 Root 객체를 렌더링해서, 변경사항을 디스플레이에 반영합니다.

정리하자면, 위의 코드는 'app'이라는 id를 가진 DOM 요소를 찾아서, 그 위치에 `<MyComponent />` 컴포넌트를 렌더링한다는 것을 의미합니다. 이를 통해 React 애플리케이션의 시작점을 설정하고, 해당 컴포넌트가 화면에 나타나게 됩니다.

<br/><br/>

## 정리

다시 글 초반에 봤던 코드를 볼까요?

```jsx
import React from "react";
import ReactDOM from "react-dom/client";

function MyComponent() {
  return <h1>Hello world</h1>;
}

ReactDOM.createRoot(document.getElementById("app")).render(<MyComponent />);
```

분명 지금까지 다뤘던 내용을 잘 이해하셨다면, 위 코드는 그다지 어려워보이지 않을겁니다.

다시 배운 내용을 정리하자면, 아래와 같습니다.

- React 애플리케이션은 **여러 컴포넌트로 구성**됩니다.
- 컴포넌트는 **사용자 인터페이스의 일부를 렌더링**하는 역할을 담당합니다.
- 컴포넌트를 생성하고 렌더링하려면 **React** 및 **ReactDOM**을 가져와야 합니다.
- React 컴포넌트는 **Javascript 함수로 정의**하여, **함수형 컴포넌트**로 만들 수 있습니다.
- 함수형 컴포넌트의 이름은 반드시 **대문자로 시작**해야 하며, **파스칼 표기법**이 주로 사용됩니다.
- 함수형 컴포넌트는 **JSX 구문**으로 React 요소를 반환해야 합니다.
- React 컴포넌트는 **‘HTML의 닫는 태그가 없는 태그 (`<br />` 같은)’로 ‘컴포넌트 이름’을 호출**하여 사용할 수 있습니다.
- React 컴포넌트를 렌더링하려면 `.createRoot()`를 사용하여 루트 컨테이너를 지정하고, 해당 컨테이너에서 `.render()` 메서드를 호출해야 합니다.

꽤 많은 내용이었지만, 이해하시는 데 도움이 되었으면 좋겠습니다.

잘못된 내용이 있다면 알려주세요. 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
- [https://developer.mozilla.org/ko/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/React_getting_started](https://developer.mozilla.org/ko/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/React_getting_started)
- [https://react.dev/reference/react-dom/render](https://react.dev/reference/react-dom/render)
