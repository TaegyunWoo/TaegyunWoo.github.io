---
category: TS
tags: [트러블슈팅, 스프링, 쿠키]
title: "스프링에서 쿠키가 추가되지 않는 문제와 해결법"
date:   2021-08-15 16:30:00 
lastmod : 2021-08-15 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 문제 배경
- `response.addCookie(cookie)` 를 통해 쿠키 응답을 시도했다.

- 하지만 클라이언트 측에서 쿠키를 받을 수 없었다.

- 문제 코드
  ```java
  Cookie cookie = new Cookie(쿠키이름, 쿠키값);
  response.addCookie(cookie);
  ```

<br><br>

# 문제 해결 탐색과정

- 구글링을 통해 조사를 해본바, 본인과 동일한 문제를 찾을 수 있었다.

- 따라서, 해당 해결방법을 참고하여 문제를 해결했다.

> [관련 링크](https://brocess.tistory.com/252)


<br><br>

# 문제 해결
## 문제 원인

- 서버로부터 전달받은 **쿠키를 어디에 저장해야할지 설정을 안해서**, 클라이언트가 쿠키를 받지 못한다.

- 따라서, 전달할 쿠키의 경로를 설정하여 문제를 해결했다.

<br><br>

## 해결방법

### `cookie.setPath("/")`

- 해당 코드를 추가하여 경로를 설정했다.

- 즉, 아래 코드와 같이 변경하였다.

```java
  Cookie cookie = new Cookie(쿠키이름, 쿠키값);
  cookie.setPath("/");
  response.addCookie(cookie);
```

- 이후, 클라이언트가 쿠키를 정상적으로 받고, 서버에 요청할 때마다 쿠키정보를 보내는 것을 확인했다.