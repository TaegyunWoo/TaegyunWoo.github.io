---
category: TS
tags: [TS]
title: "input 태그에 disabled 속성 설정시 값이 전달되지 않는 문제"
date:   2021-09-06 15:08:00 
lastmod : 2021-09-06 15:08:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 문제 발생 개발기록
  - [CRUD Web 개발일지: 2021-09-06](https://taegyunwoo.github.io/CRUD_Web/2021-09-06)

<br/>

# 문제 배경

- `<form>` 태그를 통해, 사용자에게 값을 전달받고자 했다.
- **특정 필드는 사용자에게 값을 보여주기만 하고 수정은 할 수 없도록 의도하고 싶었다.**
- 따라서, `disabled` 속성을 `<input>` 태그에 지정했다.

- 하지만 HTML의 `<input>` 태그의 속성 `disabled`를 사용하면, 해당 필드의 값이 전달되지 않는 문제가 발생하였다.
- 예시  
  ```html
  <form>
    <input type="text" name="field1" value="data1" disabled>
    <input type="text" name="field2" value="data2">
  </form>
  ```
  위와 같이, `disabled` 를 적용했을때 서버가 전달받는 값은 아래와 같다.
  ```properties
  field2=data2
  ```

<br><br>

# 문제 해결 탐색과정
- 비슷한 경험을 겪은 과거를 떠올리고 구글을 통해 조사하였다.

<br><br>

# 문제 해결
## 문제 원인

- `disabled` 속성을 지정했을 때, 서버로 값이 전달되지 않는 것은 정상적인 결과이다.
- 따라서, 다른 속성을 사용해야한다.

<br><br>

## 해결방법

- 특정 필드를 수정할 수 없도록 막고자한다면, `disabled` 속성 대신 **`readonly`** 속성을 사용해야한다.
- 예시
  ```html
  <form>
    <input type="text" name="field1" value="data1" readonly>
    <input type="text" name="field2" value="data2" disabled>
  </form>
  ```
  서버가 전달받은 값은 아래와 같다.
  ```properties
  field1=data1
  ```