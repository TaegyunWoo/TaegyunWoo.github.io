---
category: React
tags: [React]
title: "[React] JSX 심화 사용방법"
date: 2023-10-16 20:10:00 +0900
lastmod: 2023-10-16 20:10:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이번 포스팅에서는 지난 포스팅에 이어서, JSX의 사용방법에 대해 좀 더 깊게 알아보겠습니다.

JSX가 무엇이고, 기본적인 사용방법은 어떻게 되는지를 알아보려면, [이전 게시글 ‘[React] JSX 알아보기’](https://taegyunwoo.github.io/react/Tech_React_JSX)를 참고해주시면 되겠습니다.

<br/><br/>

## `class` 속성 사용방법

일반적인 HTML에서는 `class` 라는 속성을 아래처럼 사용하곤 합니다.

```html
<div class="myClass">...</div>
```

**하지만 JSX에서는 `class` 속성 대신 `className` 속성을 사용해야 합니다.**

왜냐하면 이미 JS에는 `class` 라는 예약어가 존재하기 때문입니다.

이 부분에 유의하도록 합시다!

```jsx
const myDiv = <div className="big">I AM A BIG DIV</div>;
```

<br/><br/>

## Self-closing 태그 사용방법

HTML에는 두 가지 종류의 태그가 존재합니다.

```html
<div></div>
<input />
<br />
```

위 `<div></div>` 처럼 반드시 2개의 태그로 둘러싸는 경우가 존재하고, `<input>` 이나 `<br>` 처럼 단독 태그만으로도 사용할 수 있는 태그가 존재하죠.

이때 단독 태그를 Self-closing Tag라고 합니다.

이런 Self-closing Tag는 아래 두 가지 방법으로 사용 가능합니다.

- `<br/>`
- `<br>`

즉, 마지막에 `/` 문자가 있어도 되고, 생략해도 문제되지 않죠.

**하지만 JSX에서 Self-closing Tag를 사용하는 경우라면, 반드시 `/` 문자가 포함되어야 합니다.**

```jsx
/*아래는 불가능*/
const profile = (
  <div>
    <h1>John Smith</h1>
    <img src="images/john.png">
    <article>
      My name is John Smith.
      <br>
      I am a software developer.
      <br>
      I specialize in creating React applications.
    </article>
  </div>
);

/*수정*/
const profile = (
  <div>
    <h1>John Smith</h1>
    <img src="images/john.png" />
    <article>
      My name is John Smith.
      <br />
      I am a software developer.
      <br />
      I specialize in creating React applications.
    </article>
  </div>
);
```

<br/><br/>

## HTML 태그 안에 JS 코드 삽입 (JS 표현식)

아래 코드는 어떻게 동작하게 될까요?

```jsx
let myVar = <h1>2 + 3</h1>;
```

- `<h1>5</h1>`
- `<h1>2 + 3</h1>`

위 두가지 중 어떤 값이 `myVar`에 담기게 될까요?

**정답은 두번째 값 `<h1>2 + 3</h1>` 입니다.**

`<h1>5</h1>` 처럼, 태그 안의 JS 코드를 실제로 실행하려면 아래처럼 **중괄호**로 묶으면 됩니다.

```jsx
let myVar = <h1>{2 + 3}</h1>;
```

중괄호로 묶음으로써, JSX 컴파일러에게 해당 코드는 일반 String이 아니고 실제 코드라고 알려줄 수 있습니다.

아래처럼 활용할 수도 있습니다.

```jsx
const goose =
  "https://content.codecademy.com/courses/React/react_photo-goose.jpg";
const gooseImg = <img src={goose} />;
```

<br/><br/>

## 이벤트 리스너 등록

중괄호를 사용한 JS 표현식으로 이벤트 리스너 함수를 전달해서, 특정 태그의 이벤트 리스너를 등록할 수 있습니다.

```jsx
function clickAlert() {
  alert("You clicked this image!");
}

<img onClick={clickAlert} />;
```

참고로 HTML에서는 `onClick` 과 같은 속성의 이름이 모두 소문자로 구성됩니다. 하지만 JSX에서는 카멜 케이스로 작성하면 됩니다.

<br/><br/>

## JS 표현식에서 조건문 사용

위에서 중괄호를 사용해 JS 코드를 삽입할 수 있다고 했습니다.

**하지만 중괄호 안에서 `if` 문을 사용하는 것은 불가능합니다.**

따라서 아래 코드는 오류가 발생합니다.

```jsx
(
  <h1>
    {
      if (purchase.complete) {
        'Thank you for placing an order!'
      }
    }
  </h1>
)
```

조건문을 다루는 몇가지 방법에 대해 설명하겠습니다.

<br/>

### 중괄호 없이 조건문 사용하기

아래처럼 `{}` 없이 사용할 수 있습니다.

```jsx
let message;

if (user.age >= drinkingAge) {
  message = <h1>Hey, check out this alcoholic beverage!</h1>;
} else {
  message = <h1>Hey, check out these earrings I got at Claire's!</h1>;
}
```

<br/>

### 삼항 연산자 활용하기

아래처럼 삼항 연산자를 활용하는 것도 가능합니다.

```jsx
const headline = <h1>{age >= drinkingAge ? "Buy Drink" : "Do Teen Stuff"}</h1>;
```

<br/>

### `&&` 연산자 활용하기

JS에서 제공되는 `&&` 연산자를 활용해도 좋습니다.

참고로 `&&` 연산자는 아래와 같이 동작합니다.

> 논리적 AND (`&&`)은 피연산자를 왼쪽에서 오른쪽으로 평가하면서 첫 [거짓 같은](https://developer.mozilla.org/ko/docs/Glossary/Falsy) 피연산자를 만나면 즉시 그 값을 반환합니다.  
> **만약 모든 값이 [참 같은 값](https://developer.mozilla.org/ko/docs/Glossary/Truthy)이라면 마지막 피연산자의 값이 반환됩니다.**  
> 만약 어떤 값이 `true`로 변환 가능하다면 그 값은 소위 [참 같은 값(truthy)](https://developer.mozilla.org/ko/docs/Glossary/Truthy)이라 합니다.  
> 만약 어떤 값이 `false`로 변환 가능하다면 그 값은 소위 [거짓 같은 값(falsy)](https://developer.mozilla.org/ko/docs/Glossary/Falsy) 이라고 합니다.  
> <br/> [출처: MDN]

`&&` 연산자는 모든 피연산자의 값이 true 와 같다면, **마지막 피연산자를 반환한다**는 특성을 활용하는 것입니다.

```jsx
const tasty = (
  <ul>
    <li>Applesauce</li>
    {!baby && <li>Pizza</li>}
    {age > 15 && <li>Brussels Sprouts</li>}
    {age > 20 && <li>Oysters</li>}
    {age > 25 && <li>Grappa</li>}
  </ul>
);
```

위 코드의 결과는 아래와 같습니다.

- `!baby` 가 true라면, `<li>Pizza</li>`
- `age > 15` 가 true라면, `<li>Brussels Sprouts</li>`
- `age > 20` 가 true라면, `<li>Oysters</li>`
- `age > 25` 가 true라면, `<li>Grappa</li>`

<br/><br/>

## Map을 활용해, 여러 태그 만들기

이번에는 `map` 함수를 활용해서, 간편하게 다중 태그를 생성해보겠습니다.

```jsx
const ary = ["Home", "Shop", "About Me"];

const listItems = ary.map((item) => <li>{item}</li>);

<ul>{listItems}</ul>;
```

위 코드의 경우, `listItems` 변수에 아래 값이 담기게 됩니다.

```jsx
[<li>'Home'</li>, <li>'Shop'</li>, <li>'About Me'</li>];
```

그리고 이 값을 `<ul> </ul>` 태그 사이에 넣게 되는 것이죠.

<br/><br/>

## `key` 속성

JSX 에서는 각 태그마다 `key` 라는 속성을 지원합니다.

아래는 예시 코드입니다.

```jsx
<ul>
  <li key="li-01">Example1</li>
  <li key="li-02">Example2</li>
  <li key="li-03">Example3</li>
</ul>
```

<br/>

### `key` 속성이란?

`key` 속성은 **리스트를 만들 때 포함해야 하는 특수한 속성**입니다.

이 속성을 통해, React가 어떤 항목을 변경, 추가 또는 삭제할지 식별하는 것을 돕습니다. 따라서 `key` 속성의 값은 **유니크한 값**이어야 합니다.

**React가 렌더링을 진행할 때, 리스트의 경우 `key` 속성을 사용해서 좀 더 효율적으로 DOM 객체를 조작할 수 있습니다.**

<br/>

### `key` 속성을 왜 사용해야하나요?

그 이유는 **리스트의 효율적인 DOM 객체 조작**을 위해서입니다.

아래 코드를 봅시다.

```jsx
//기존 DOM
<ul>
  <li>first</li>
  <li>second</li>
</ul>

//수정된 DOM
<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

만약 위처럼 다시 DOM을 렌더링해야 한다면, React는 `<ul>` 의 가장 마지막 부분에 `<li>third</li>` 를 추가하고 렌더링을 끝냅니다.

`<li>first</li>` 가 일치하는지 확인하고, `<li>second</li>` 가 일치하는지 확인한 뒤, 마지막으로 `<li>third</li>` 을 추가하고 마무리됩니다.

이 경우엔, 매우 속도가 빠르겠죠? 마지막 요소만 추가하면 되니까요.

하지만 아래처럼 수정된다면 어떨까요?

```jsx
//기존 DOM
<ul>
  <li>first</li>
  <li>second</li>
</ul>

//수정된 DOM
<ul>
  <li>second</li>
  <li>third</li>
  <li>first</li>
</ul>
```

이 경우엔, 모든 요소를 수정해줘야 합니다. (순서가 모두 달라졌기 때문이죠.)

`<li>first</li>` 을 `<li>second</li>` 로 수정하고, `<li>second</li>` 을 `<li>third</li>` 로 수정하고, 마지막으로 `<li>first</li>` 을 추가하고 렌더링이 수행됩니다.

따라서 이렇게 순서가 뒤바뀐 경우엔 매우 비효율적으로 동작하게 됩니다.

**이때 `key` 속성을 사용해서 문제를 해결할 수 있습니다.**

**만약 항목들이 key를 가지고 있다면, React는 “key를 통해” 기존 DOM과 이후 DOM의 항목들이 일치하는지 확인합니다.**

따라서 매우 효율적으로 DOM 객체를 수정할 수 있게 됩니다.

`key` 사용 예시는 아래와 같습니다.

```jsx
//기존 DOM
<ul>
  <li key="id_0">first</li>
  <li key="id_1">second</li>
</ul>

//수정된 DOM
<ul>
  <li key="id_1">second</li>
  <li key="id_2">third</li>
  <li key="id_0">first</li>
</ul>
```

보다 자세한 내용은 [공식 문서 ‘자식에 대한 재귀적 처리’](https://ko.legacy.reactjs.org/docs/reconciliation.html#recursing-on-children) 를 참고하세요.

<br/><br/>

## **`React.createElement`**

JSX 없이도 React 코드를 작성할 수 있습니다.

> 물론 대부분 JSX를 사용해서 개발하겠지만, 이 내용에 대해서도 어느정도 이해해두는 것이 도움이 될 것 같습니다.

```jsx
const h1 = <h1>Hello world</h1>;
```

JSX를 사용한 위 코드는, JSX 없이 아래처럼 작성할 수 있습니다.

```jsx
const h1 = React.createElement("h1", null, "Hello world");
```

그리고 이것이 바로, **JSX 컴파일러가 동작하는 방식**입니다.

우리가 JSX 코드로 작성하면, JSX 컴파일러는 `React.createElement` 을 사용하는 코드로 번역해줍니다.

`React.createElement` 가 어떻게 동작하는지가 궁금하시다면, [공식 문서 ‘createElement’](https://react.dev/reference/react/createElement) 를 참고해주세요.

<br/><br/>

## References

- [https://www.codecademy.com](https://www.codecademy.com/)
- [https://react.dev](https://react.dev)
