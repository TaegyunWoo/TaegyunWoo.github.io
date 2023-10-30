---
category: React
tags: [React]
title: "[React] Hook - State"
date: 2023-10-30 13:50:00 +0900
lastmod: 2023-10-30 13:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

지금까지 살펴본 내용은 아래와 같습니다.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**
5. **[[React] React 애플리케이션 시작하기](https://taegyunwoo.github.io/react/Tech_React_ReactStructure)**
6. **[[React] 컴포넌트 상호작용 (props)](https://taegyunwoo.github.io/react/Tech_React_Props)**

함수형 컴포넌트에 상태를 추가하려면 어떻게 해야 할까요?

우리가 만든 애플리케이션이 데이터가 변화할 때 반응하기를 원한다면 Hook을 사용할 수 있습니다.

이번 시간에는 React Hook의 기본적인 내용에 대해 알아보고, **State Hook**에 대해 설명하겠습니다.

<br/><br/>

## React Hook이란?

React Hook은 **함수형 컴포넌트에 대한 여러 기능을 지원하는 기술**을 의미합니다. (대표적인 예로, 컴포넌트의 상태가 변할 때 다시 렌더링을 하도록 만들 수 있습니다.)

특히 State Hook을 사용하면 **상태(state)에 따라 사용자 인터페이스가 어떻게 보일지 선언**함으로써, 사용자에게 **무엇을 보여줄 것인지 결정**할 수 있습니다.

React는 다양한 내장 Hook을 제공합니다. 대표적으로 `useState()`, `useEffect()`, `useContext()`, `useReducer()` 및 `useRef()` 가 존재합니다.

모든 내장 Hook에 대해 살펴보려면, 아래 React 문서를 참고해도 좋습니다.

[https://react.dev/reference/react](https://react.dev/reference/react)

<br/>

### 이번 시간에 배울 내용

이 포스팅에서는 `state` 에 대해서 알아보겠습니다. 다룰 내용은 아래와 같습니다.

- 상태를 갖는 (stateful) 함수형 컴포넌트를 만들어봅니다.
- State Hook을 사용해봅니다.
- State를 초기화하고 설정해봅니다.
- 이벤트 핸들러를 정의합니다.
- State Setter 콜백 함수를 사용해봅니다.
- 배열 및 객체와 함께 state를 사용합니다.

이번 학습이 끝나면 우리는 아래 코드를 이해할 수 있게 될 겁니다.

```jsx
import React, { useState } from "react";
import NewTask from "../Presentational/NewTask";
import TasksList from "../Presentational/TasksList";

export default function AppFunction() {
  const [newTask, setNewTask] = useState({});
  const handleChange = ({ target }) => {
    const { name, value } = target;
    setNewTask((prev) => ({ ...prev, id: Date.now(), [name]: value }));
  };

  const [allTasks, setAllTasks] = useState([]);
  const handleSubmit = (event) => {
    event.preventDefault();
    if (!newTask.title) return;
    setAllTasks((prev) => [newTask, ...prev]);
    setNewTask({});
  };
  const handleDelete = (taskIdToRemove) => {
    setAllTasks((prev) => prev.filter((task) => task.id !== taskIdToRemove));
  };

  return (
    <main>
      <h1>Tasks</h1>
      <NewTask
        newTask={newTask}
        handleChange={handleChange}
        handleSubmit={handleSubmit}
      />
      <TasksList allTasks={allTasks} handleDelete={handleDelete} />
    </main>
  );
}
```

상당히 복잡해보이죠? 하지만 겁먹을 필요 없습니다! 지금부터 하나씩 살펴봅시다.

<br/><br/>

## State Hook

State Hook은 React 컴포넌트를 만들 때 가장 일반적으로 사용되는 Hook 중 하나입니다. State Hook은 아래처럼 구조분해를 사용해 가져올 수 있습니다.

```jsx
import React, { useState } from "react";
```

`react` 의 default export 객체를 `React` 로 할당받고, `React.useState` 함수를 `useState` 로 할당받는다는 의미입니다.

<br/>

`useState` 함수를 호출하면, 우리는 두 개의 값을 Return 받게 됩니다.

- **현재 상태(state)**
- **상태 Setter 함수**

이 두 가지 값을 사용해서, 상태를 추적하거나 변경할 수 있습니다. 이 값들은 보통 아래처럼 **배열 구조 분해 할당** 방식으로 전달받습니다.

```jsx
const [currentState, setCurrentState] = useState();
```

여기서 중요한 것은 **`currentState`** 변수를 **직접 조작하면 안된다**는 것입니다. 만약 **`currentState += 1`** 처럼 직접 변경하려고 한다면, **React에서는 해당 상태값이 변경되었음을 감지할 수 없**고, 따라서 **변경된 상태값이 정상적으로 애플리케이션에 반영되지 않을 것**입니다. 이것은 **React에서 ‘상태의 불변성’을 유지해야 함**을 의미합니다.

**따라서 반드시 Setter 함수를 사용해서, 간접적으로 상태를 업데이트해야 합니다.**

<br/>

### 예시

좀 더 실용적인 예시를 살펴보겠습니다.

```jsx
import React, { useState } from "react";

export default function ColorPicker() {
  const [color, setColor] = useState();
  const divStyle = { backgroundColor: color };

  return (
    <div style={divStyle}>
      <p>The color is {color}</p>
      <button onClick={() => setColor("Aquamarine")}>Aquamarine</button>
      <button onClick={() => setColor("BlueViolet")}>BlueViolet</button>
      <button onClick={() => setColor("Chartreuse")}>Chartreuse</button>
      <button onClick={() => setColor("CornflowerBlue")}>CornflowerBlue</button>
    </div>
  );
}
```

위 코드에서는 전달받은 ‘상태 Setter 함수 `setColor`’를 `onClick` **이벤트핸들러로 호출**합니다.

만약 버튼을 클릭해서 `toggle` **상태값이 변경된다면**, **해당 컴포넌트(`Toggle`)를 다시 렌더링**하게 됩니다.

1. 버튼을 클릭한다.
2. `onClick` 이벤트핸들러로 등록된 화살표 표현식이 실행된다.
3. 화살표 표현식에 의해, `setColor(...)` 함수가 실행된다.
4. `setColor(...)` 함수에 의해, `color` 상태가 변경된다.
5. `ColorPicker` 컴포넌트가 재렌더링된다.
   1. 변경된 `color` 상태의 값으로 `divStyle` 변수에 담긴 객체의 `backgroundColor` 프로퍼티의 값이 설정된다.
   2. 최종적으로 `div` 태그의 스타일 속성이 적용된다.

<br/>

### 다양한 타입의 State

State Hook을 사용하여, 문자열 같은 기본 데이터 유형의 값은 물론이고, **배열 및 객체와 같은 타입도 관리**할 수 있습니다.

아래 예시를 볼까요?

```jsx
function MyComponent() {
  const [bool, setBool] = useState();
  const [obj, setObj] = useState();
  return (
    <>
      <button onClick={() => setBool(true)}>Boolean</button>
      <button onClick={() => setObj({ myField: "hello" })}>Object</button>
    </>
  );
}
```

<br/>

### state 초기화

state를 초기화해서 사용하려면, 아래와 같이 함수를 호출하면 됩니다.

```jsx
const [bool, setBool] = useState(false); //false로 bool 초기화
const [obj, setObj] = useState({}); //빈객체로 obj 초기화
```

1. 첫 렌더링 시, 초기 state 값을 사용합니다.
2. 만약 setter 함수가 호출되면, state에 할당된 새로운 값을 사용합니다.
3. 만약 어떤 다른 이유로 컴포넌트가 다시 렌더링된다면, 이전 렌더링에서 사용된 state 값을 그대로 사용합니다.

`useState()`를 호출할 때 초기 값을 전달하지 않으면, 첫 번째 렌더링에서 state의 값은 `undefined` 가 됩니다.

코드를 실행하는 컴퓨터에는 문제될 것은 없지만, **코드를 읽는 사람에게는 명확하지 않**을 수 있습니다.

따라서 **state를 명시적으로 초기화**하는 것을 권장합니다.

만약 첫 번째 렌더링에서 필요한 값이 없으면, 명시적으로 **null을 전달**하는 것을 고려해야 합니다.

<br/>

### 함수형 업데이트

setter 함수를 실행해서 state를 업데이트하는 작업은 **비동기적으로 수행**됩니다.

즉, 상태를 변경하는 코드가 즉시 실행되지 않을 수 있고, **React가 업데이트 작업을 모으고 나중에 일괄적으로 처리**할 수 있습니다. (React는 이런 업데이트 방식을 채택해서 성능을 개선합니다. 자세한 것은 이어서 설명할게요.)

따라서 우리가 의도하지 않은 방식으로 동작할 수 있습니다. 아래 예제를 볼까요?

```jsx
const [count, setCount] = useState(0);

// 비동기 업데이트로 인한 문제
const increment = () => {
  setCount(count + 1); //A
  setCount(count + 2); //B
};
```

위 코드에서 최종적으로 **`count`**에 3을 더하려고 했지만, **React는 이러한 업데이트를 일괄적으로 처리하므로 결과적으로 `count`는 2만 증가**합니다. 즉, 가장 마지막 코드인 B만 적용됩니다.

**React는 상태 업데이트를 비동기적으로 처리하기 때문에, 작업 A(**`setCount(count + 1)`**)와 B(**`setCount(count + 2)`**)를 모아서 한번에 처리할 수 있습니다.**  
**상태 업데이트 작업을 모아 한번에 처리하는 과정**에서 **각 작업들이 병합**되고, 의도하지 않은 결과를 만들 수 있습니다.

<br/>

상태 업데이트 작업들이 병합되는 과정을 `Object.assign` 함수로 표현할 수 있는데, 그 예시는 아래와 같습니다.

```jsx
// 초기 상태
const initialState = {
  count: 0,
  name: "John",
};

// 첫 번째 업데이트
const update1 = { count: 1 };

// 두 번째 업데이트
const update2 = { name: "Doe" };

// Object.assign을 사용하여 업데이트 작업 병합
const newState = Object.assign({}, initialState, update1, update2);
// 최종적으로 { count: 1, name: "Doe" } 가 newState 변수에 담기게 됩니다

console.log(newState);
```

만약 객체의 동일한 속성 `count`를 Setter 함수로 수정한다면, 아래와 같이 덮어씌워지게 됩니다.

```jsx
// 초기 상태
const initialState = {
  count: 0,
};

// 첫 번째 업데이트
const update1 = { count: 1 };

// 두 번째 업데이트
const update2 = { count: 2 };

// Object.assign을 사용하여 업데이트 작업 병합
const newState = Object.assign({}, initialState, update1, update2);
// 최종적으로 { count: 2 } 가 newState 변수에 담기게 됩니다
console.log(newState);
```

따라서 방금 살펴본 예시에서 최종 상태값은 2가 됩니다.

이 문제를 해결하기 위해 **함수형 업데이트**를 사용할 수 있습니다. **함수형 업데이트**를 사용하면 **React는 각 업데이트를 ‘이전 상태에 기반’하여 처리**합니다.

따라서 이 코드를 아래처럼 수정할 수 있습니다.

```jsx
const [count, setCount] = useState(0);

// 함수형 업데이트 적용
const increment = () => {
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 2);
};
```

위 코드를 보면, setter 함수의 파라미터로 ‘변경될 값을 직접 전달’하는 것이 아니라, **변경 로직을 콜백 함수로써 전달**한 것을 확인할 수 있습니다.

이 경우, `setCount` 함수를 실행한 **순서대로 콜백함수가 큐에 적재**됩니다. 그리고 큐에서 차례대로 콜백함수가 실행되며 상태값이 업데이트됩니다. **따라서 비동기적으로 상태 업데이트가 진행되더라도, 병합되지 않고 ‘이전 상태’에 기반해서 작업할 수 있습니다.**

이처럼 콜백 함수를 setter 함수의 파라미터로 전달하면, **React가 ‘콜백 함수의 파라미터’에 ‘기존 상태값’을 전달**해줍니다. 그리고 **콜백 함수의 결과값으로 상태값을 업데이트하게 되는 것**이죠.

따라서, 상태값은 정상적으로 3이 됩니다.

> 참고로, Setter 함수의 콜백함수의 파라미터 이름은 **관례적으로 ‘prev상태명’로 명명**합니다.

<br/>

### 비동기적으로 동작하는 상태 업데이트

그렇다면 React는 왜 **상태 업데이트를 비동기적으로 처리**하는 것일까요?

그 이유는 **성능 최적화**와 **렌더링 효율성을 향상**시키기 위함입니다. 이를 통해, 다음과 같은 이점을 얻을 수 있습니다.

- **일괄 처리를 통한 성능 최적화**
  - React는 상태 업데이트를 비동기적으로 처리하여 **여러 상태 업데이트가 동시에 발생할 때 이를 한 번에 처리**합니다. 이렇게 함으로써 **불필요한 중간 렌더링을 줄일 수 있**으며, 성능을 최적화할 수 있습니다.
  - React는 **여러 상태 업데이트를 일괄 처리**함으로써 **컴포넌트의 렌더링 주기를 줄**입니다. 이전 상태와 새로운 상태를 비교하고, **변경된 부분만을 실제 DOM에 반영**하는 데에 더 효율적으로 작동합니다.
  > 상태를 A로 변경한 뒤 바로 B로 변경한다면, 굳이 상태값이 A인 경우로 렌더링할 이유가 없죠?  
  >  그래서 최종적으로 업데이트된 상태인 B인 경우로만 렌더링합니다.

<br/><br/>

## 다양한 예시

### 배열 타입 state 예시

이번에는 배열 타입의 state를 어떻게 다루는지 예시를 통해 알아보겠습니다.

```jsx
//상태 초기화
const [selected, setSelected] = useState([]); //빈 배열로 초기화

//이벤트핸들러
const toggleItem = ({ target }) => {
  const clickedItem = target.value;

  //상태 업데이트
  setSelected((prev) => {
    if (prev.includes(clickedTopping)) {
      //이미 선택된 아이템을 다시 선택하는 경우
      return prev.filter((t) => t !== clickedItem); //해당 아이템이 아닌 다른 아이템들을 담은 새 배열 객체 반환
    } else {
      //선택되지 않은 아이템을 선택하는 경우
      return [clickedItem, ...prev]; //해당 아이템까지 담은 새 배열 객체 반환
    }
  });
};
```

위 코드에서 이벤트핸들러 함수는 선택된 아이템을 `selected` 상태에 추가하고, 만약 이미 존재한다면 `selected` 상태에서 제거하는 로직을 수행합니다.

우리가 주목해야하는 부분은 return 문이 작성된 부분입니다. 모두 기존 배열을 수정하고 return 하는 것이 아니라, **새로운 배열을 만들어서 return한다는 점**입니다.

초반에 설명했듯이, 상태는 불변성을 갖기 때문에 Setter 함수를 통해서 **간접적으로 상태 업데이트를 수행**해야한다고 했습니다.

> `myState = 3` 과 같이, 직접 수정하면 안됩니다.

만약 아래와 같이 **기존 배열에 새로운 아이템을 `push` 한다**면, **상태를 직접 수정하는 것**이 되고, 이는 **상태 불변성을 위반**하는 코드가 됩니다.

```jsx
const clickedItem = "new item";

//상태 업데이트
setSelected((prev) => {
  prev.push(clickedItem); //기존 상태(배열 객체) 직접 수정
  return prev; //기존 배열 객체 반환
});
```

따라서 **새로운 배열 객체를 만들어 Return 해야 합니다.**

<br/>

### 객체 타입 state 예시

객체 타입의 경우 역시, 동일한 객체를 수정해서 업데이트해서는 안됩니다.

```jsx
const [formState, setFormState] = useState({});
const handleChange = ({ target }) => {
  const { name, value } = target;
  setFormState((prev) => ({
    //새로운 객체 반환
    ...prev, //기존 객체의 속성 추가
    [name]: value, //Computed Property Name
  }));
};
```

<br/>

### 여러 state 사용 예시

우리는 하나의 컴포넌트에 여러 state를 사용할 수도 있습니다.

물론 하나의 거대한 객체를 단독으로 state로서 활용할 순 있지만, 이 경우 가독성이 떨어지고 관리하기도 까다롭습니다.

```jsx
//거대한 단독 state
function Subject() {
  const [state, setState] = useState({
    currentGrade: 'B',
    classmates: ['Hasan', 'Sam', 'Emma'],
    classDetails: {topic: 'Math', teacher: 'Ms. Barry', room: 201},
    exams: [{unit: 1, score: 91}, {unit: 2, score: 88}])
  });
	// ...
}
```

위 코드를 아래처럼 개선할 수 있습니다.

```jsx
//여러 state 사용
function Subject() {
  const [currentGrade, setGrade] = useState("B");
  const [classmates, setClassmates] = useState(["Hasan", "Sam", "Emma"]);
  const [classDetails, setClassDetails] = useState({
    topic: "Math",
    teacher: "Ms. Barry",
    room: 201,
  });
  const [exams, setExams] = useState([
    { unit: 1, score: 91 },
    { unit: 2, score: 88 },
  ]);
  // ...
}
```

별도의 상태 변수를 사용하여 데이터를 관리하면 코드를 더욱 간단하게 작성하고, 읽고, 테스트하고, 컴포넌트 전체에서 재사용할 수 있는 등 많은 이점이 있습니다.

<br/><br/>

## 정리

지금까지 꽤 많은 내용을 살펴보았습니다. 다시 정리해보면, 아래와 같습니다.

- Hook을 통해서, **함수형 컴포넌트**를 **다양하게 조작**할 수 있다.
- React가 제공하는 Hook 중, **State Hook**이 존재한다. State Hook은 **컴포넌트의 상태를 관리**하고, **상태값의 변화**에 따라 **컴포넌트를 재구성 및 재렌더링**해준다.
- 컴포넌트의 상태는 **불변성**을 유지해야 한다. 따라서 직접 상태값을 업데이트할 순 없고, 반드시 **Setter 함수를 사용**해서 간접적으로 상태를 변경해야 한다.
- 성능 개선을 위해, **상태 업데이트**는 React에서 **비동기적으로 수행**된다. 그리고 같은 상태를 **여러번 업데이트**한다면, 모든 업데이트가 **하나로 병합**되어, 결국 **가장 마지막 업데이트 작업**만 실제로 **반영**된다.  
  만약 여러 업데이트 작업을 각각 수행하고 싶다면, **함수형 업데이트**를 사용할 수 있다.
- **상태값의 타입**으로 **배열**이나 **객체**를 사용할 수 있다. 다만 **상태는 불변성을 유지**해야 하기 때문에, **동일한 배열이나 객체를 수정**해서 **상태를 업데이트하는 작업은 피**해야 한다.

단순히 `useState` 함수를 사용하는 방법을 아는 것을 넘어서, 동작원리까지 아는 것이 좋겠죠? 다소 복잡하지만, 차근히 읽어보면 충분히 이해하실 수 있을겁니다.

혹시 잘못된 내용이 있다면 알려주세요. 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com/](https://www.codecademy.com/)
- [https://react.dev/reference/react](https://react.dev/reference/react)
