---
category: React
tags: [React]
title: "[React] 스타일 적용하기"
date: 2023-11-09 17:50:00 +0900
lastmod: 2023-11-09 17:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이번에는 React에서 스타일을 사용하는 방법에 대해 알아보겠습니다.

지금까지 다뤘던 내용은 아래와 같습니다.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**
5. **[[React] React 애플리케이션 시작하기](https://taegyunwoo.github.io/react/Tech_React_ReactStructure)**
6. **[[React] 컴포넌트 상호작용 (props)](https://taegyunwoo.github.io/react/Tech_React_Props)**
7. **[[React] Hook - State](https://taegyunwoo.github.io/react/Tech_React_StateHook)**
8. **[[React] Hook - Effect](https://taegyunwoo.github.io/react/Tech_React_EffectHook)**
9. **[[React] 컨테이너, 프레젠테이셔널 컴포넌트](https://taegyunwoo.github.io/react/Tech_React_Container_Presentional)**

React에서 스타일을 사용하는 방법에는 여러 가지가 존재합니다. 이번 글에서는 아래 3가지에 대해 알아보겠습니다.

- 인라인 스타일, 스타일 객체 변수
- 스타일 문법
- CSS 모듈

그다지 어렵지 않은 내용이기 때문에, 겁먹으실 필요는 없습니다! 바로 알아볼까요?

<br/><br/>

## 인라인 스타일과 스타일 객체 변수

가장 먼저 인라인 스타일과 스타일 객체에 대해 알아볼까요?

인라인 스타일은 아래와 같이, 태그의 속성으로 작성된 것을 의미합니다.

```jsx
<h1 style={{ color: "red" }}>Hello world</h1>
```

`style` 속성을 보면, 가장 바깥 중괄호로 JS 코드임을 표현하고, 안쪽 중괄호로 JS 객체를 전달했음을 알 수 있습니다.

만약, 여러 스타일을 적용한다면 아래와 같이 매우 지저분한 코드가 됩니다.

```jsx
<h1 style={{ color: 'white', background: 'black', ... }}>Hello world</h1>
```

이 경우, 아래와 같이 변수를 통해 전달하는 것이 가독성 측면에서 효과적입니다.

```jsx
//스타일 변수
const darkMode = {
  color: 'white',
  background: 'black';
};

//변수 사용
<h1 style={darkMode}>Hello world</h1>
```

어떤가요? 간단한 내용이라 이해에 큰 어려움을 없을 것입니다.

<br/><br/>

## 스타일 문법

스타일을 적용할 때, 주의해야하는 것이 있습니다. 바로 네이밍 규칙인데요. React에서는 `margin-top` 과 같은 CSS 속성을 `marginTop` 처럼 **카멜 케이스**로 작성합니다. (JSX에서 속성 이름을 카멜 케이스로 작성하는 것과 동일하죠?)

즉, 아래처럼 표현합니다.

```jsx
const styles = {
  marginTop: "20px",
  backgroundColor: "green",
};
```

추가적으로, 스타일 값의 타입은 대부분 문자열입니다. 스타일 값이 숫자형인 경우에도, **단위를 지정할 수 있도록 문자열로 작성**해야 합니다. 예를 들어 `'450px'` 또는 `'20%'`라고 씁니다.

만약 아래처럼 스타일 값으로 숫자형을 사용한다면, 자동으로 단위가 `'px'` 로 설정됩니다.

```jsx
{
  fontSize: 30;
} //30px
```

<br/><br/>

## CSS 모듈 적용

지금까지 살펴본 방식을 사용한다면, 애플리케이션이 커질수록 스타일 디버깅이 어려워질 수 있습니다.

스타일을 모듈화하고 재사용할 수 있도록 만들기 위해서, **컴포넌트별 스타일시트를 만들어 적용**할 수 있습니다. 바로 예시 코드를 볼까요?

### `MyStyle.module.css`

```css
.myClassName {
  background: pink;
  height: 175px;
  display: block;
}
```

위와 같이 CSS 파일을 정의할 수 있습니다.

다만 **파일 이름을 `[파일이름].module.css` 형식으로 정의**해야 합니다. 이 규칙을 따라야지만, 해당 CSS 파일을 **하나의 CSS 모듈로 처리**할 수 있게 됩니다. 즉, **다른 React 코드에서 스타일 모듈로서 import 하는 것이 가능**해집니다.

### `MyComponent.js`

```jsx
import React from 'react';
import Styles from './MyStyle.module.css';

function MyComponent() {
	return (
		<div className={ Styles.myClassName }>
			<...>
		</div>
	);
}
```

방금 정의한 CSS 모듈을 위와 같이 사용할 수 있습니다.

가장 먼저 `import` 를 사용해서, CSS 모듈을 가져옵니다. 그리고 React(JSX)에서 `class` 속성을 지정하기 위해, `className` 속성을 사용합니다. 해당 속성의 값으로 **CSS 모듈의 클래스 이름을 `CSS_모듈_객체명.클래스_이름` 형식**으로 적용하면 됩니다.

<br/>

### 고유 클래스명

동일한 클래스명을 가진 여러 스타일시트가 있는 경우, 클래스명이 충돌하여 스타일이 덮어씌워질 수 있습니다.

CSS 모듈은 이 문제를 해결합니다. 스타일을 **특정 컴포넌트에만 적용**할 수 있습니다. 왜냐하면, **각 CSS 모듈에 대해 고유한 클래스 이름을 자동으로 생성해서 적용**하기 때문입니다. 따라서 여러 CSS 파일에 걸쳐서 클래스 이름을 일일이 관리할 필요가 없습니다.

예를 들어, 아래와 같이 동작할 수 있습니다.

```css
/* MyStyle.module.css */

.myClassName {
  /* 이 클래스명이 MyStyle_myClassName_abc123 로 변환됨 */
  background: pink;
  height: 175px;
  display: block;
}
```

```jsx
// MyComponent.js

import React from "react";
import Styles from "./MyStyle.module.css";

function MyComponent() {
  return <div className="MyStyle_myClassName_abc123">{/* 내용 */}</div>;
}
```

<br/><br/>

## 정리

- React 컴포넌트는 인라인 스타일, 객체 변수 스타일, 스타일시트, CSS 모듈 등 다양한 방법으로 스타일을 지정할 수 있습니다.
- 인라인 스타일 지정은 `style` 속성에 스타일 객체를 전달해서 수행할 수 있습니다.
- 항상 CSS 속성 이름은 카멜 케이스로 작성합니다.
- React에서 숫자형 스타일 값은 `px` 단위를 사용하게 됩니다.
- 스타일을 분리해서 CSS 모듈 파일에 저장할 수 있습니다. 특정 컴포넌트의 `className` 속성을 통해 사용할 수 있습니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
