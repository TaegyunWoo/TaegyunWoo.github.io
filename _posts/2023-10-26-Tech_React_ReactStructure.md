---
category: React
tags: [React]
title: "[React] React 애플리케이션 시작하기"
date: 2023-10-26 19:00:00 +0900
lastmod: 2023-10-26 19:00:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

이번 포스팅에서는 실제로 React 프로젝트를 생성하고, 애플리케이션이 어떻게 구성되는지 알아보겠습니다.

이전 글은 아래를 참고하시면 됩니다.

1. **[[React] JSX 알아보기](https://taegyunwoo.github.io/react/Tech_React_JSX)**
2. **[[React] 가상 DOM 객체](https://taegyunwoo.github.io/react/Tech_React_VirtualDOM)**
3. **[[React] JSX 심화 사용방법](https://taegyunwoo.github.io/react/Tech_React_AdvJSX)**
4. **[[React] 기본 컴포넌트 알아보기](https://taegyunwoo.github.io/react/Tech_React_Basic_Component)**

<br/><br/>

## 프로젝트 생성하기

이전에 `create-react-app` 을 **글로벌로 설치**한 적이 있다면, **제거**하는 것이 좋습니다. 아래 명령어를 사용해서, 전역적으로 설치된 `create-react-app` 을 제거해주세요.

```bash
npm uninstall -g create-react-app
```

그리고 프로젝트를 생성할 폴더를 열어서, 아래 명령어를 입력해 새로운 React 프로젝트를 구성해봅시다.

```bash
npx create-react-app [프로젝트_이름]
```

> 프로젝트 이름에는 **소문자만** 와야함에 주의합시다.

위 명령어를 통해서, 자동으로 애플리케이션의 구조와 몇가지 개발 설정들을 구성할 수 있습니다.

<br/><br/>

## 프로젝트 구조

이제 프로젝트가 성공적으로 생성되었을텐데, 구조가 다소 복잡해보일 수 있습니다.

전체적인 구조를 나타내면 아래와 같습니다.

```bash
myfirstreactapp
├── node_modules
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
├── src
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   ├── logo.svg
│   ├── serviceWorker.js
│   └── setupTests.js
├── .gititgnore
├── package.json
├── package-lock.json
└── README.md
```

각 파일과 디렉토리별로 하나씩 살펴보겠습니다.

<br/>

### `package.json` 파일

![Untitled](/assets/img/2023-10-26-Tech_React_ReactStructure/Untitled.png)

- **`name`** 은 **애플리케이션의 이름**을 의미합니다.
- **`version`** 은 **현재 애플리케이션의 버전**을 말합니다.
- **`"private": true`** npm 생태계에서 앱을 실수로 **공개 패키지로 게시**하는 것을 **방지**합니다.
- **`dependencies`** 은 애플리케이션에 필요한 **모든 Node 모듈과 버전**을 포함하게 됩니다. 위의 그림에서는 여섯 개의 의존성(dependencies)을 볼 수 있습니다. 처음 세 개는 테스트를 위한 것이고, 다음 두 개는 JavaScript에서 **react**와 **react-dom**을 사용할 수 있게 해줍니다. 마지막으로, **react-scripts**는 React와 관련된 유용한 개발 스크립트를 제공합니다.  
  위의 스크린샷에서 지정된 react 버전은 **^16.13.1**입니다. 이는 npm이 16.x.x에 해당하는 가장 최신 주 버전을 설치할 것을 의미합니다. **즉, 해당 메이저 버전의 가장 최신 마이너 버전을 의미합니다.**
  반대로, **~1.2.3**과 같은 것도 볼 수 있는데, 이는 **1.2.x에 해당하는 가장 최신 버전을 의미**합니다.
- **`scripts`** 는 **react-scripts** 명령에 대한 별칭을 지정합니다. 예를 들어, 명령줄에서 **`npm test`** 를 실행하면 내부적으로 **`react-scripts test --env=jsdom`** 이 실행됩니다.
- 또한 두 개의 추가 속성 **`eslintConfig`** 와 **`browserslist`** 를 볼 수 있습니다. 이 두 가지는 각각 자체 값을 가지는 Node 모듈입니다. **`browserslist`** 는 **앱의 브라우저 호환성**에 대한 정보를 제공하며, **`eslintConfig`** 는 [code linting](https://stackoverflow.com/questions/8503559/what-is-linting)을 처리합니다.
  > code linting : 소스코드를 분석해서, 발생할 수 있는 오류를 찾아내는 것

<br/>

### **`node_modules` 디렉토리**

이 디렉토리에는 `package.json`에 지정된 패키지의 **종속성 및 하위 종속성**이 포함되어 있습니다.

이 폴더는 `.gitignore`에 자동으로 추가됩니다. 굳이 GitHub 같은 공유 레포지토리에 종속성까지 업로드할 필요가 없기 때문입니다.

<br/>

### `package-lock.json` 파일

`node_modules` 디렉토리에 설치된 **종속성에 대한 세부 내용**이 담겨 있습니다. 이를 통해, **개발팀이 동일한 버전의 종속성 및 하위 종속성을 갖도록** 하는 방법을 제공합니다.

의존하는 패키지들의 **버전 충돌을 방지**하고, 프로젝트를 다른 환경에서 **복제할 때 일관성을 유지**하는 데 도움이 됩니다.

또한 package.json에 대한 **변경 내역이 포함**되어 있으므로 종속성 변경 사항을 빠르게 되돌아볼 수 있습니다.

<br/>

### `public` 디렉토리

이 디렉터리에는 직접적으로 제공되는 **정적 asset**이 포함되어 있습니다. 대표적인 파일은 아래와 같습니다.

- `index.html` : 웹서비스의 진입지점이 됩니다. 이를 React가 렌더링해줍니다.
- `robots.txt` : 봇(Bot)에 대한 정책 내용을 담습니다.

<br/>

### `src` 디렉토리

여기에는 JavaScript가 포함되어 있으며 **React 앱의 핵심**이 되는 JS 파일이 담기게 됩니다.

`App.js`, `App.css` 및 테스트 도구 모음(`App.test.js`)이 존재합니다.

`index.js` 및 `index.css`은 **앱에 대한 진입지점**을 제공하고, `RegisterServiceWorker.js`를 시작합니다.

`RegisterServiceWorker.js` 는 **사용자를 위한 파일 캐싱 및 업데이트**를 담당합니다. 이를 통해, 초기 방문 후 **오프라인 기능**과 **더 빠른 페이지 로드**가 가능합니다.

> 자세한 내용은 [https://developer.chrome.com/docs/workbox/service-worker-overview/](https://developer.chrome.com/docs/workbox/service-worker-overview/) 을 참고하세요.

React 앱이 커져감에 따라, 컴포넌트를 모아둘 `component` 디렉터리를 추가하고, React의 뷰를 모아둘 `views` 디렉터리를 추가하는 것이 관례적입니다.

<br/><br/>

## React 애플리케이션 실행하기

아래 명령어로 애플리케이션을 로컬에서 실행할 수 있습니다.

```bash
npm start
```

`http://localhost:3000` 에 접속하면 아래와 같은 화면을 볼 수 있습니다.

![Untitled](/assets/img/2023-10-26-Tech_React_ReactStructure/Untitled%201.png)

축하합니다! React 애플리케이션을 성공적으로 실행했습니다.

다음 시간에는 React의 Hook에 대해 다뤄보겠습니다.

이번 글에서 잘못된 내용이 있다면 알려주세요. 감사합니다.

<br/><br/>

## References

- [https://www.codecademy.com](https://www.codecademy.com/courses/react-101/articles/how-to-create-a-react-app)
- [https://developer.chrome.com/docs/workbox/service-worker-overview/](https://developer.chrome.com/docs/workbox/service-worker-overview/)
