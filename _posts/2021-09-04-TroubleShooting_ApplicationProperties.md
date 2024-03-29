---
category: TS
tags: [TS]
title: "스프팅부트에서 application.properties 파일을 나눠서 관리하기"
date:   2021-09-04 12:05:00 
lastmod : 2021-09-04 12:05:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 문제 발생 개발기록
  - [CRUD Web 개발일지: 2021-08-29](https://taegyunwoo.github.io/CRUD_Web/2021-08-29)

<br/>

# 문제 배경

- DB 설정 관련 내용을 `application.properties` 가 아닌, 다른 파일에 작성하고 싶다.
- DB 관련 설정은 보안에 민감한 사항이기 때문에, 관련 내용을 다른 파일로 관리하고 해당 파일이 푸쉬되지 않도록 `.gitignore` 에 등록하고 싶다.
- **따라서, `application-testdb.properties` 라는 파일을 새로 생성하여 작성했으나, 해당 파일이 적용되지 않았다.**

<br><br>

# 문제 해결 탐색과정
- 이전 강의에서 다뤘었던 주제로 기억하여, 이전 강의자료를 찾아보았다.
- 추가적으로 구글링을 통해 원인을 조사하고 문제를 해결하였다.

<br><br>

# 문제 해결
## 문제 원인

- **`application.properties` 에 분리된 `properties` 파일을 알려주어야한다.**
- 필자가 과거에 관련 내용을 공부할 때, 단순히 `application-{설정이름}.properties` 라는 이름 형식을 갖는 파일을 스프링부트가 자동으로 등록한다고 기억했다.
- 하지만 그것 뿐만 아니라, `application.properties` 에 해당 설정파일을 알려주어야 한다.

<br><br>

## 해결방법

1. `src/main/resources` 경로에 `application-임의이름.properties` 파일을 생성한다.
2. `application-임의이름.properties` 파일에 원하는 설정 내용을 작성한다.
3. `application.properties` 파일에 아래 내용을 작성한다.
    ```properties
    spring.profiles.active=임의이름
    ```