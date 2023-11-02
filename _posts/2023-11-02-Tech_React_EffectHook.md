---
category: React
tags: [React]
title: "[React] Hook - Effect"
date: 2023-10-30 13:50:00 +0900
lastmod: 2023-10-30 13:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

지난 포스팅에서 React Hook이 무엇이고, React가 기본적으로 제공하는 Hook 중 State에 대해 알아봤습니다. 이번에는 또다른 Hook 중 하나인 Effect에 대해 알아보겠습니다.

지금까지 살펴본 내용은 아래를 참고해주세요.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**
5. **[[React] React 애플리케이션 시작하기](https://taegyunwoo.github.io/react/Tech_React_ReactStructure)**
6. **[[React] 컴포넌트 상호작용 (props)](https://taegyunwoo.github.io/react/Tech_React_Props)**
7. **[[React] Hook - State](https://taegyunwoo.github.io/react/Tech_React_StateHook)**

<br/><br/>

## Effect Hook

### Effect Hook 이란?

Effect Hook은 **컴포넌트가 렌더링된 이후에 특정 작업을 수행하도록 지시**합니다. State와 함께 Effect Hook을 사용하면, 웹 페이지를 동적으로 변화시킬 수 있습니다.

<br/>

### Effect Hook 사용 예시

Effect Hook이 어떻게 사용되는지, 바로 예시를 통해서 알아볼까요?

사용자가 우리의 웹페이지에 입력을 할 때마다, ‘웹 페이지 탭의 제목’을 변경할 수 있도록 하려고 한다고 가정해 보겠습니다. 다음과 같이 Effect Hook(`useEffect()`)을 사용하여 이를 구현할 수 있습니다.

```jsx
import React, { useState, useEffect } from "react";

export default function PageTitle() {
  const [name, setName] = useState("");

  useEffect(() => {
    document.title = `Hi, ${name}`;
  }, [name]);

  return (
    <div>
      <p>Use {name} input field below to rename this page!</p>
      <input
        onChange={({ target }) => setName(target.value)}
        value={name}
        type="text"
      />
    </div>
  );
}
```

위 예제 코드를 하나씩 뜯어봅시다.

<br/>

### `useEffect()`

가장 먼저 Effect Hook을 사용하기 위해선, 아래처럼 `useEffect` 함수를 import 해야 합니다.

```jsx
import { useEffect } from "react";
```

`useEffect()` 함수는 **반환 값이 없**습니다. `useEffect()` 함수의 파라미터로 전달된 **콜백함수**나 **Effect**가 **컴포넌트가 렌더링된 후 실행**됩니다.

위에서 살펴본 예시에서는 `PageTitle` 컴포넌트가 렌더링된 이후 마다, `useEffect()` 함수의 파라미터로 전달된 콜백함수 `() => {document.title = `Hi, ${name}`; }` 가 실행됩니다.

전체적인 동작 순서를 보면, 아래와 같습니다.

1. 사용자가 웹페이지의 `<input>` 태그에 **입력**한다.
2. 입력할 때마다, `onChange` 에 등록된 **이벤트 핸들러 `({target}) => setName(target.value)` 가 실행**된다.
3. 이벤트 핸들러에 의해 **State Setter 함수(`setName`)이 실행**된다.  
   따라서, State(`name`)이 `target.value` 로 **업데이트**된다.
4. State가 업데이트되었으므로, React는 컴포넌트 `PageTitle` 을 **재렌더링**한다.
5. 재렌더링이 끝난 후, `useEffect` 함수로 **등록한 콜백함수 `() => {document.title = `Hi, ${name}`;}` 가 실행**된다.

   > 매개변수로 제공한 `[name]` 에 대해선 나중에 다룰게요.

6. 따라서 해당 웹 페이지의 **탭 제목이 `"Hi, "` 뒤에 현재 `name` 값으로 변경**된다.
7. 최종적으로 **실제 DOM에 ‘가상 DOM의 변경사항’을 반영**한다.

<br/>

### Effect 메모리 누수

구현된 Effect 로직에 따라 **메모리에서 정리(Clean-Up)** 가 필요한 경우가 있습니다.

아래 코드를 볼까요?

```jsx
import React, { useState, useEffect } from "react";

export default function Counter() {
  const [clickCount, setClickCount] = useState(0);

  const increment = () => {
    setClickCount((prevClickCount) => prevClickCount + 1);
  };

  useEffect(() => {
    document.addEventListener("mousedown", increment);
  });

  return <h1>Document Clicks: {clickCount}</h1>;
}
```

위 코드를 실행하게 되면, 아래와 같이 동작하게 됩니다. 차근히 따라와볼까요?

1.  `Counter` 컴포넌트가 처음 렌더링됩니다.
2.  렌더링이 끝난 후, `useEffect` 에 의해 아래 콜백함수가 실행됩니다.

    ```jsx
    //effect 처리 함수
    () => {
      //전체 문서의 mouseDown 이벤트 리스너에 increment 이벤트 핸들러 등록
      document.addEventListener("mousedown", increment);
    };
    ```

3.  사용자가 처음 보게 되는 화면은 아래와 같습니다.

    ![Untitled](/assets/img/2023-11-02-Tech_React_EffectHook/Untitled.png)

4.  사용자가 문서를 한번 클릭합니다.
5.  등록된 이벤트핸들러 `increment` 에 의해, 아래 콜백함수가 실행됩니다.

    ```jsx
    //이벤트 핸들러 함수
    () => {
      setClickCount((prevClickCount) => prevClickCount + 1);
    };
    ```

6.  State Setter 함수가 실행되었으므로 `clickCount` State가 업데이트(`0+1`로)되고, `Counter` 컴포넌트가 다시 렌더링 됩니다.
7.  다시 렌더링이 되었으므로, **또다시 `useEffect` 에 전달된 아래 콜백함수가 실행**됩니다.  
    따라서 **동일한 이벤트 핸들러가 다시 등록**됩니다. 총 **2개의 `increment` 이벤트 핸들러**를 갖게 됩니다.
    ```jsx
    //effect 처리 함수
    () => {
    	//increment 이벤트 핸들러를 또다시 등록
      document.addEventListener("mousedown", increment);
    }
    ```
8.  이제 사용자는 아래 화면을 보게 됩니다.

    ![Untitled](/assets/img/2023-11-02-Tech_React_EffectHook/Untitled%201.png)

9.  또다시 사용자가 문서를 클릭합니다.
10. 현재 등록된 **2개의 `increment` 이벤트 핸들러**에 의해, **아래 콜백함수가 총 2번 호출**됩니다.

    ```jsx
    //이벤트핸들러 함수가 2번 호출됨.
    () => {
      setClickCount((prevClickCount) => prevClickCount + 1);
    };
    ```

11. `setClickCount()` 가 **2번 호출**되었기 때문에, `clickCount` State는 `1+1+1` 로 업데이트됩니다.  
    즉 `clickCount` 는 **2가 아닌 3**이 됩니다.
    그리고 컴포넌트를 **총 2번 렌더링(`setClickCount()`를 2번 호출했기에)** 합니다.
12. **2번 렌더링**이 되었으므로, 또다시 `useEffect` 에 전달된 아래 콜백함수가 **2번 실행**됩니다.  
    따라서 **총 4개의 `increment` 이벤트 핸들러**를 갖게 됩니다.
    ```jsx
    //effect 처리 함수
    //렌더링이 2번 진행되었기 때문에, 이 effect 처리 함수도 2번 실행됨.
    () => {
    	//increment 이벤트 핸들러를 또다시 등록
      document.addEventListener("mousedown", increment);
    }
    ```
13. 사용자는 아래와 같은 화면을 보게 됩니다.

    ![Untitled](/assets/img/2023-11-02-Tech_React_EffectHook/Untitled%202.png)

    의도와는 다르게 동작한 것을 알 수 있습니다.

14. 최종적으로 `Counter` 컴포넌트는 중복된 이벤트 핸들러 `setClickCount` 를 4개 갖게 됩니다.  
    **그리고 사용자가 클릭을 하면 할수록, 그 수가 2배씩 증가하게 됩니다.**

이 코드는 **렌더링이 수행될 때**마다 **이벤트 핸들러가 새로 등록**되며, **각 이벤트 핸들러가 State를 업데이트**하면 **렌더링이 다시 수행**됩니다. 이로 인해 **새로운 이벤트 핸들러가 또다시 추가**됩니다. 즉, 아래와 같이 정리할 수 있습니다.

렌더링 수행 → 이벤트 핸들러 등록 → 이벤트 핸들러가 State 업데이트 → 렌더링 수행 → 다시 이벤트 핸들러 등록 → …

이로 인해서, **우리의 의도와는 다르게 사용자 화면이 변경된 것도 문제**이긴 하지만, 더 큰 문제가 있습니다.

바로, **새로운 이벤트 핸들러가 사용자의 클릭마다 기하급수적으로 메모리에 추가된다는 것**이죠!

따라서 사용자의 웹브라우저는 시간이 갈수록 성능이 저하될 것입니다.

<br/>

이 문제를 해결하기 위해서, **Effect 콜백함수 안에서 return 문으로 ‘Clean-Up’을 수행하는 콜백함수’를 반환**할 수 있습니다. 아래처럼 말이죠.

```jsx
useEffect(() => {
  document.addEventListener("keydown", handleKeyPress);

  //Clean-Up 함수 반환
  return () => {
    document.removeEventListener("keydown", handleKeyPress); // 해당 이벤트 핸들러를 제거
  };
});
```

이렇게 Effect 콜백함수의 반환값으로 또다른 콜백함수를 반환하면, React는 해당 콜백 함수를 **‘컴포넌트 재렌더링 이전’과 ‘컴포넌트 언마운트 이전’에 호출**합니다.

> 언마운트 : DOM에서 특정 컴포넌트를 제거하는 것

이전에 살펴본 상황에서 Effect 함수가 Clean-Up 함수를 반환하도록 위처럼 수정하면, 이벤트 핸들러 함수에 의해 재렌더링이 수행되기 전에 **미리 현재 등록된 이벤트 핸들러 함수를 제거할 수 있습니다.**

Effect 로직에 따라, Clean-Up 함수를 반환해야 하는 것을 잊지않는 것이 중요하다는 사실을 기억해주세요!

<br/>

### 컴포넌트 첫 렌더링에서만 Effect 실행하기 (Dependency Array)

바로 위에서 성능 문제나 기타 버그가 발생하지 않도록, Clean-Up 함수를 반환하는 방법을 배웠습니다. 하지만 이런 방식 대신, 아예 **재렌더링시 발생하는 Effect 실행**을 건너뛰고 싶을 수 있습니다. 그리고 보통 함수형 컴포넌트를 정의할 때, 컴포넌트가 마운트(첫 렌더링)될 때만 Effect를 실행하고, **다시 렌더링될 때는 Effect를 실행하지 않는 것이 일반적**입니다. 이번에는 그 방법에 대해 알아보겠습니다.

**`useEffect()` 함수에 빈 배열을 전달하면, 우리가 원하는대로 첫 렌더링시에만 Effect가 실행되도록 처리할 수 있습니다.** 첫 번째 렌더링이 수행된 후에만 Effect를 호출하고 재렌더링시에는 호출하지 않으려면, **빈 배열을 `useEffect()`의 두 번째 파라미터로 전달**합니다. 이 두 번째 인수를 **Dependency Array**라고 합니다.

그 예시코드는 아래와 같습니다.

```jsx
useEffect(() => {
  alert("component rendered for the first time");
  return () => {
    alert("component is being removed from the DOM");
  };
}, []); //빈 배열(Dependency Array) 전달
```

Dependency Array는 **‘Effect를 호출할 시기’와 ‘건너뛸 시기’를 `useEffect()` 메서드에 알려주는 데 사용**됩니다. Dependency Array를 `useEffect()` 에 전달하면, **‘Dependency Array 요소의 값’이 렌더링 도중 변경되었을 때만, Effect가 호출**됩니다. (첫 렌더링시에도 Effect가 호출됨)

Dependency Array로 Effect를 실행하는 조건을 정리하자면 아래와 같습니다.

- **컴포넌트가 화면에 첫 렌더링될 때**
- **Dependency Array의 value값이 바뀔 때**

따라서 모든 state가 아닌 **특정 state에 관해 Side effect를 실행**시킬 수 있습니다.

> Side Effect : 컴포넌트의 상태(State)가 변경될 때 발생하는 작업, 주로 컴포넌트의 렌더링 이후에 발생함

만약 지금까지 그랬던 것처럼 아래와 같이, Dependency Array 없이 `useEffect()` 를 호출하면 컴포넌트가 렌더링된 이후 마다 Effect를 호출하게 됩니다.

```jsx
useEffect(() => {
  alert("component rendered for the first time");
  return () => {
    alert("component is being removed from the DOM");
  };
}); //Dependency Array 전달 X
```

<br/><br/>

## React Hook의 규칙

우리는 이전 글부터 지금까지 React Hook(State, Effect)에 대해 알아봤습니다.

이 React Hook에도 몇가지 규칙이 존재하는데, 이는 아래와 같습니다.

- **Hook은 컴포넌트의 최상위 레벨에서 호출해야한다. early return이 실행되기 전에, React 함수의 최상위(at the top level)에서 Hook을 호출해야 합니다.**
  - 또한 **조건문**, **반복문**, **중첩 함수** 내에서 **Hook을 호출하지 않**아야 한다.
  - 이 규칙을 지킴으로써, 컴포넌트가 렌더링 될 때마다 **항상 동일한 순서로 Hook이 호출**되는 것이 보장할 수 있습니다.
- **Hook은 함수형 컴포넌트 내에서만 호출할 수 있다.**
  - **클래스 컴포넌트**나 **일반 JavaScript 함수**에서 **Hook을 사용하지 말**아야 한다.

<br/>

### 왜 최상위 레벨에서 Hook을 호출해야 할까?

왜냐하면, Hook의 동작을 예측 가능하고 안정적으로 유지하기 위함입니다. React가 State나 Effect를 처리하는 방식을 살펴볼까요?

한 컴포넌트에서 State나 Effect Hook을 여러 개 사용할 수 있습니다.

```jsx
function Form() {
  // 1. name이라는 state 변수를 사용하세요.
  const [name, setName] = useState("Mary");

  // 2. Effect를 사용해 폼 데이터를 저장하세요.
  useEffect(function persistForm() {
    localStorage.setItem("formData", name);
  });

  // 3. surname이라는 state 변수를 사용하세요.
  const [surname, setSurname] = useState("Poppins");

  // 4. Effect를 사용해서 제목을 업데이트합니다.
  useEffect(function updateTitle() {
    document.title = name + " " + surname;
  });

  // ...
}
```

이렇게 여러 State와 Effect를 사용하는 경우, **React는 Hook이 호출되는 순서에 의존**해서, **특정 state나 Effect가 어떤 `useState` 나 `useEffect` 호출에 해당하는지 추적**하게 됩니다.

```jsx
// ------------
// 첫 번째 렌더링
// ------------
useState("Mary"); // 1. 'Mary'라는 name state 변수를 선언합니다.
useEffect(persistForm); // 2. '폼 데이터를 저장하기 위한 effect'를 추가합니다.
useState("Poppins"); // 3. 'Poppins'라는 surname state 변수를 선언합니다.
useEffect(updateTitle); // 4. '제목을 업데이트하기 위한 effect'를 추가합니다.

// -------------
// 두 번째 렌더링
// -------------
useState("Mary"); // 1. name state 변수를 읽습니다.(인자는 무시됩니다)
useEffect(persistForm); // 2. '폼 데이터를 저장하기 위한 effect'가 대체됩니다.
useState("Poppins"); // 3. surname state 변수를 읽습니다.(인자는 무시됩니다)
useEffect(updateTitle); // 4. '제목을 업데이트하기 위한 effect'가 대체됩니다.

// ...
```

즉 두 번째 렌더링 시, 아래와 같이 동작합니다.

1. 첫 번째 렌더링에서 ‘가장 처음 호출된 `name` state 변수’를 읽는다. (재렌더링이므로, 파라미터로 전달된 ‘Mary’는 무시)
2. 첫 번째 렌더링에서 추가된 ‘두번째로 호출된 effect(폼 데이터 저장용)’를 ‘두 번째 렌더링의 마지막 `useEffect` 함수의 `persistForm` 파라미터 값’으로 대체한다.
3. 첫 번째 렌더링에서 ‘세번째로 호출된 `surname` state 변수’를 읽는다. (재렌더링이므로, 파라미터로 전달된 ‘Poppins’는 무시)
4. 첫 번째 렌더링에서 추가된 ‘마지막으로 호출된 effect(제목 업데이트용)’를 ‘두 번째 렌더링의 마지막 `useEffect` 함수의 `persistForm` 파라미터 값’으로 대체한다.

이런 과정을 통해서, **컴포넌트 재렌더링 시 해당 컴포넌트의 ‘기존 state’와 ‘기존 effect’를 추적**할 수 있게 됩니다.

> React 내부적으로 state나 effect들을 배열로 관리하고, 순서에 따라 접근하게 됩니다.

<br/>

하지만 아래처럼 조건문에 `useEffect` 를 사용하면 어떻게 될까요?

```jsx
// 🔴 조건문에 Hook을 사용함으로써 첫 번째 규칙을 깼습니다
if (name !== "") {
  useEffect(function persistForm() {
    localStorage.setItem("formData", name);
  });
}
```

아마 **첫 렌더링** 시에는 Form의 `name` 값이 빈 문자열이기 때문에, 조건문이 `true` 가 되어 위와 동일하게 아래 순서로 Hook이 실행될겁니다.

```jsx
// ------------
// 첫 번째 렌더링
// ------------
useState("Mary"); // 1. 'Mary'라는 name state 변수를 선언합니다.
useEffect(persistForm); // 2. '폼 데이터를 저장하기 위한 effect'를 추가합니다.
useState("Poppins"); // 3. 'Poppins'라는 surname state 변수를 선언합니다.
useEffect(updateTitle); // 4. '제목을 업데이트하기 위한 effect'를 추가합니다.
```

하지만 사용자가 Form을 사용해 `name` 값을 변경했다면, 조건문이 `false` 가 되고 **두 번째 렌더링**은 아래와 같이 동작하게 됩니다.

```jsx
// -------------
// 두 번째 렌더링
// -------------
useState("Mary"); // 1. name state 변수를 읽습니다. (인자는 무시됩니다)
// useEffect(persistForm)  // 🔴 Hook을 건너뛰었습니다!
useState("Poppins"); // 🔴 2 (3이었던). surname state 변수를 읽는 데 실패했습니다.
useEffect(updateTitle); // 🔴 3 (4였던). 제목을 업데이트하기 위한 effect가 대체되는 데 실패했습니다.
```

이 경우, **첫 렌더링에서 호출된 Hook의 순서와 두 번째 렌더링의 순서가 달라**집니다. React는 **이전 렌더링 때처럼, 재렌더링되었을 때의 ‘두 번째 Hook 호출’이 `persistForm` effect와 일치할 것이라 예상했지만 그렇지 않았**습니다. 따라서 그 시점부터 건너뛴 Hook 다음에 호출되는 **Hook이 순서가 하나씩 밀리면서 버그**를 발생시키게 됩니다.

**이것이 컴포넌트 최상위(the top of level)에서 Hook이 호출되어야만 하는 이유입니다.**

만약 조건부로 effect를 실행하기를 원한다면, 아래처럼 **조건문을 Hook 내부**에 넣을 수 있습니다.

```jsx
// 👍 더 이상 첫 번째 규칙을 어기지 않습니다
useEffect(function persistForm() {
  if (name !== "") {
    localStorage.setItem("formData", name);
  }
});
```

<br/><br/>

## 정리하며…

- 'react' 라이브러리에서 `useEffect()` 함수를 가져와서 함수형 컴포넌트에서 호출할 수 있습니다.
- **Effect**는 **`useEffect()` 함수의 첫 번째 파라미터로 전달하는 함수**를 나타냅니다. 기본적으로 Effect Hook은 **컴포넌트의 매 렌더링 후에 이 효과를 호출** 합니다.
- **Clean-Up 함수**는 Effect의 return 값으로 설정할 수 있습니다. 메모리 누수나 ‘중복에 의한 버그’를 방지하기 위해 정리해야 하는 작업을 수행하는 경우 사용합니다. 컴포넌트가 **마운트 해제될 때뿐만 아니라, Effect를 다시 호출하기 전**에 이 **Clean-Up 함수를 호출**합니다.
- **Dependency Array**를 통해 **원하는 State가 변경되었을 때만, Effect가 동작하도록 제어**할 수 있습니다. Dependency Array는 `useEffect()` 함수를 호출할 때 두 번째 파라미터로 결정할 수 있습니다.
- 모든 Hook은 **함수형 컴포넌트 내에서만 호출**될 수 있고, **조건문이나 반복문 등이 아닌, 컴포넌트의 최상위 레벨에서만 호출**해야 합니다.

꽤 많은 이야기를 했습니다만, React를 잘 다루려면 반드시 짚고 넘어가야하는 것이라 생각합니다.

혹시 잘못된 내용이 있다면, 알려주세요! 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
- [https://ko.legacy.reactjs.org/docs/hooks-rules.html](https://ko.legacy.reactjs.org/docs/hooks-rules.html)
