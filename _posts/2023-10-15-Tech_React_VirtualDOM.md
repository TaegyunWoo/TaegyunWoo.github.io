---
category: React
tags: [React]
title: "[React] 가상 DOM 객체"
date: 2023-10-15 20:10:00 +0900
lastmod: 2023-10-15 20:10:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이번 시간에는 React의 가상 DOM 객체, 일명 Virtual DOM에 대해 알아보겠습니다.

가상 DOM을 이해하기 위해선, 기존에는 어떤 문제가 있었는지 아는 것이 도움됩니다.

먼저 어떤 문제가 있었는지 알아봅시다.

<br/><br/>

## 기존 문제

DOM 조작은 현대 웹에서 필수적인 요소입니다. 이를 통해서, 다양한 UI 컴포넌트를 조작하며, 다이나믹한 웹 서비스를 구성할 수 있습니다.

하지만 DOM 조작은 **느리다는 단점**이 있습니다.

<br/>

### 문제 예시

왜 DOM 조작이 느리다고 하는 것일까요?

**결론부터 이야기한다면, 한 문서의 일부분만 수정하더라도 문서의 모든 부분들을 다시 렌더링하기 때문입니다.**

예를 들어보겠습니다.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled.png)

DOM 모델은 위와 같이 **트리 구조**로 구성됩니다.

만약 사용자가 `Remove` 버튼을 클릭한다면, 아래와 같이 관련 `<li>` 태그가 제거된다고 가정해보겠습니다.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled%201.png)

사용자가 `Ruby` 항목을 제거한다면, 아래와 같이 화면이 다시 렌더링됩니다.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled%202.png)

**이때, 수정되지 않은 `<h1>` 태그와 나머지 `<li>` 태그까지 모두 다시 렌더링을 수행합니다.**

따라서 성능저하가 발생하게 되는 것이죠.

이 문제를 해결하고자 Virtual DOM이 등장했습니다.

<br/><br/>

## 가상 DOM

React에는, 한 문서의 **모든 DOM 객체를 담는 '가상 DOM 객체'**가 있습니다. 가상 DOM 개체는 기존의 DOM 객체의 경량본이라고 생각하면 됩니다.

즉, 한 HTML 문서의 DOM 객체를 하나의 JS 객체로 표현한 것을 Virtual DOM 객체라고 합니다. 아래처럼 말이죠.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled%203.png)

그리고 DOM 객체가 조작된다면, React는 아래와 같이 동작하게 됩니다.

- **기존의 Virtual DOM 객체의 스냅샷을 찍어둔다.**
- **Virtual DOM 객체를 조작한다.**
- **기존 스냅샷과 현재 상태의 차이점을 분석하고, 변경된 부분만 다시 렌더링한다.**

만약 사용자가 이전 예제의 `<li>` 태그를 제거했다고 한다면, 아래와 같이 동작합니다.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled%204.png)

최종적으로 아래와 같이, **실제 변경사항만 반영하여 렌더링**합니다.

![Untitled](/assets/img/2023-10-15-Tech_React_VirtualDOM/Untitled%205.png)

**이렇게 변경된 사항을 파악해 부분적으로만 렌더링함으로써, 성능개선이 가능합니다.**

<br/><br/>

## 정리

지금까지 React에서 사용되는 Virtual DOM 객체에 대해 알아봤습니다.

React에서 DOM 객체를 수정하려고 할 때, 어떻게 동작하게 되는지 다시 정리해보면 아래와 같습니다.

1. DOM 객체 수정을 시도한다.
2. Virtual DOM 객체의 스냅샷을 찍어둔다.
3. Virtual DOM 객체를 업데이트한다.
4. Virtual DOM 객체의 현재 상태와 기존 스냅샷을 비교한다.  
   React는 어떤 객체가 변경되었는지 파악한다.
5. 변경된 객체에 해당하는 실제 DOM만 업데이트한다.  
   실제 DOM을 변경하면 화면이 변경된다.

다음 시간에는 JSX 내용 중, 심화 내용에 대해 다뤄보겠습니다.

감사합니다.

## References

- [https://www.codecademy.com](https://www.codecademy.com/)
- [https://www.youtube.com/watch?v=jwRAdGLUarw](https://www.youtube.com/watch?v=jwRAdGLUarw)
