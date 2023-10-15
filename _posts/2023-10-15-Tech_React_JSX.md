---
category: Tech
tags: [Tech, React]
title: "[React] JSX 알아보기"
date: 2023-10-15 19:40:00 +0900
lastmod: 2023-10-15 19:40:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

앞으로 몇 개의 포스팅 시리즈로 React 라이브러리에 대해 살펴볼 예정입니다.

프론트엔드 애플리케이션을 구현할 때, 가장 많이 사용되는 라이브러리 중 하나인 리액트에 대해 알아보겠습니다.

리액트는 JS 라이브러리로, 페이스북 엔지니어들에 의해 개발되었습니다.

리액트가 주장하는 장점은 아래와 같습니다.

- 빠릅니다. React로 만들어진 앱은 복잡한 업데이트를 처리하면서도, 빠르고 반응성이 좋습니다.
- 모듈화가 잘 되어 있습니다. 크고 밀집된 코드 파일을 작성하는 대신, 재사용 가능한 더 작은 파일을 많이 작성할 수 있습니다. React의 모듈성은 JavaScript의 유지 관리 문제에 대해 유용합니다.
- 확장 가능합니다. 자주 변화하는 데이터를 표시하는 데 React가 사용될 수 있습니다.
- 유연합니다. 웹 앱 제작과 관련이 없는 흥미로운 프로젝트에서도 React를 사용할 수 있습니다.
- 인기가 있습니다. React를 이해하면 취업 가능성이 더 높아진다는 것입니다.

리액트에 대해 학습하기 위해선, 리액트가 사용하는 **JSX라는 기술에 대해 알아두어야 합니다.**

이번 포스팅에서는 JSX가 무엇이고, 사용방법에 대해 다뤄보겠습니다.

<br/><br/>

## JSX 시작하기

JSX는 **JS의 문법 확장자**로, **JS 파일에서도 HTML 문법을 사용**할 수 있도록 지원합니다.

따라서 아래와 같은 코드가 가능합니다.

```jsx
const h1 = <h1>Hello world</h1>;
```

하지만 알아두어야 할 것이 있습니다.

**JSX는 JS의 공식적인 스펙이 아닙니다.** 따라서 위 코드를 웹브라우저에서 실행할 수는 없습니다.

JSX로 작성된 코드는 반드시 **JSX 컴파일러로 컴파일**되어야 합니다. JSX 컴파일러를 통해, 일반적인 JS 코드로 컴파일될 수 있습니다.

<br/>

### 컴파일 예시

예를 들어, 아래 JSX 코드가 있다고 해보겠습니다.

```jsx
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

위 코드를 JSX 컴파일러는 아래처럼 번역하게 됩니다.

```jsx
React.createElement(MyButton, { color: "blue", shadowSize: 2 }, "Click Me");
```

> 컴파일된 결과를 보면, `React` 의 코드를 호출하는 것을 알 수 있습니다.  
> 따라서 JSX는 반드시 `React` 와 함께 사용되어야 합니다.  
> 보다 자세한 것은 나중에 설명할 예정이니, 그렇구나~ 정도로 생각해주시면 되겠습니다.

<br/>

### 다양한 사용 예시

아래처럼 다양하게 사용할 수 있습니다.

```jsx
let navBar = <nav>I am a nav bar</nav>; //변수에 저장
const myTeam = {
  //객체로 저장
  center: <li>Benzo Walli</li>,
  powerForward: <li>Rasha Loa</li>,
  smallForward: <li>Tayshaun Dasmoto</li>,
  shootingGuard: <li>Colmar Cumberbatch</li>,
  pointGuard: <li>Femi Billon</li>,
};
```

또한, 아래처럼 속성을 갖는 태그를 만드는 것 역시 가능합니다.

```jsx
const title = <h1 id="title">Introduction to React.js</h1>;
```

만약 여러 줄로 작성한 HTML 태그라면, 아래처럼 소괄호를 사용해 작성할 수 있습니다.

```jsx
const theExample = (
  <a href="https://www.example.com">
    <h1>Click me!</h1>
  </a>
);
```

<br/>

### 반드시 하나의 태그로 묶여야 한다.

JSX에서 작성되는 태그는 모두 하나의 태그로 묶여야 합니다.

따라서 아래와 같은 코드는 오류가 발생합니다.

```jsx
//불가능한 코드
const paragraphs = (
  <p>I am a paragraph.</p>
  <p>I, too, am a paragraph.</p>
);
```

이를 아래처럼 수정할 수 있습니다.

```jsx
//수정된 코드
const paragraphs = (
  <div>
    <p>I am a paragraph.</p>
    <p>I, too, am a paragraph.</p>
  </div>
);
```

<br/><br/>

## 정리

React 라이브러리에 대한 포스팅 시리즈의 첫 번째로 JSX에 대해 알아보았습니다.

매우 간략한 내용이었지만, 정리하자면 아래와 같습니다.

- JSX는 JavaScript 파일에서 HTML 문법을 사용할 수 있도록 지원하는 JS의 문법 확장자입니다.
- JSX 코드는 JSX 컴파일러를 통해 일반적인 JavaScript 코드로 컴파일되어야 합니다.

다음 시간에는 React의 핵심적인 Virtual DOM 객체에 대해 알아보겠습니다.

감사합니다.

<br/><br/>

## References

- [www.codecademy.com](www.codecademy.com)
- [https://ko.legacy.reactjs.org/docs/jsx-in-depth.html](https://ko.legacy.reactjs.org/docs/jsx-in-depth.html)
