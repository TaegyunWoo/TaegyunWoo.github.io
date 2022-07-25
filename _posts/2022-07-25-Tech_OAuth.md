---
category: Tech
tags: [Spring]
title: "OAuth 원리"
date:   2022-07-25 17:30:00 
lastmod : 2022-07-25 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# OAuth

## 개요

### OAuth 란

- OAuth는 제 3자 서비스에 접근할 수 있는 토큰을 발급받는 방법이다.
- [RFC 표준문서](https://datatracker.ietf.org/doc/html/rfc6749)

### OAuth 주요 용어

- **Resource Server**
    - 우리가 다루고자 하는 자원을 가지고 있는 서버
    - Ex) 구글, 페이스북
- **Resource Owner**
    - 자원의 소유자
    - 유저를 의미한다.
- **Client**
    - OAuth를 통해 발급받은 Access Token을 사용하는 주체
    - 쉽게 말해, 우리가 만든 서비스를 의미한다.
- **Authorization Server**
    - 인증·인가 서버
    - Ex) 구글의 인증서버
    
    > 공식 문서에서는 `Resource Server` 와 `Authorization Server` 를 구분하지만, 본 설명에서는 `Resource Server` 로 통일하겠다.

<br/>

## OAuth 의 전체 흐름과 원리

### 1. Client가 Resource Server 에 사전 등록을 한다.

- Client가 Resource Server를 이용하기 위해 OAuth를 사용하는데, 그 전에 미리 Resource Server에 승인을 받아두어야 한다.
- 등록시 필요한 주요 요소
    - **Client ID**
        - 우리가 만든 앱의 ID
    - **Client Secret**
        - 비밀번호
    - **Authorized Redirect URLs**
        - Authorization Code를 전달받을 주소
        - Client의 주소이다.
        
        > Authorization Code 에 대해선 나중에 설명한다.
- 예시 (네이버)
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled.png)
    

> 서비스마다 조금씩 방식이 다를 수는 있다.

### 2. Resource Owner 의 OAuth 사용 시작

- Resource Owner (유저)가 버튼을 클릭하는 등의 행위를 통해, OAuth 절차를 시작하게 된다.
- 예시
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%201.png)
    
- 요청 파라미터 종류
    - **`client_id`**
        - Client가 사전에 등록해둔 ID
    - **`scope`**
        - 권한 범위
        - 위 그림 예시에선 기능 B, C에 대해 권한을 요청하게 된다.
    - **`redirect_uri`**
        - Client가 사전에 등록해둔 URI (Authorized Redirect URLs)
- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%202.png)
    

### 3. Resource Owner 로그인 확인

- Resource Server가 ‘Resource Owner가 현재 로그인이 되어 있는지' 확인한다.
- 만약 로그인이 필요하다면 아래와 같은 화면을 출력하여, 로그인을 할 수 있도록 한다.

![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%203.png)

- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%204.png)
    

### 4. 요청 파라미터 값 확인

- Resource Server가 Resource Owner로부터 요청받은 파라미터 값을 확인한다.
    - 요청 URL: `https://resource.server/?client_id=1&scope=B,C&redirect_uri=https://client/callback`
- `client_id` , `redirect_uri` 가 사전에 등록된 정보인지 확인한다.
- 만약 등록된 정보라면 계속해서 다음 절차를 수행한다.

### 5. 권한 정보 출력 및 승인

- Resource Server가 권한 정보를 Resource Owner에게 출력해주고, 그에 대한 승인을 받는다.
    - 이 권한 정보는 요청 URL의 파라미터 `scope` 에 정의되어있다.
- 예시 화면
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%205.png)
    
- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%206.png)
    

### 6. Authorization Code 전달

- Resource Owner가 권한 승인을 하면, Resource Server가 Authorization Code를 Resource Owner에게 전달한다.
- 이때 `Location` 헤더를 이용하여 기존에 등록된 Authorized Redirect URL 과 함께 보낸다.
    - `Location: https://client/callback?code=Authorization_Code`
- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%207.png)
    

### 7. Authorized Redirect URL 리다이렉션

- Resource Owner가 `Location` 헤더로 전달받은 URL로 다시 요청을 보낸다.
- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%208.png)
    

### 8. Resource Server 접속

- Client 가 Resource Server에 직접 접속하게 된다. 즉 아래와 같은 URL과 파라미터를 Client가 호출한다.
    
    ```text
    https://resource.server/token?grant_type=authorization_code&code=3&redirect_uri=https://client/callback&client_id=1&client_secret=2
    ```
    
- 시각화
    
    ![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%209.png)
    

### 9. 요청 파라미터 값 확인

- Resource Server가 Client로부터 전달받은 파라미터(`code` , `redirect_uri` , `client_id` , `client_secret`)의 값을 확인한다.
- 모두 일치한다면, Resource Server가 갖고 있는 Authorization Code는 삭제된다.
    - Authorization Code 는 임시 코드이기 때문에 삭제된다.

### 10. Access Token 전달

- Resource Server가 Client에게 Access Token을 전달한다.
- Resource Server와 Client는 Access Token을 각자 저장한다.

### 11. Access Token 사용

- Client가 Resource Server로부터 전달받은 Access Token을 사용하여, Resource Server의 API를 호출하여 사용한다.
- 호출 예시
    - 파라미터 전달 방식
        - `https://API주소?access_token=<토큰값>`
    - 헤더 전달 방식
        - `Authorization: Bearer <토큰값>`

<br/>

## Refresh Token

- 위에서 OAuth에 대해 설명할 때, Refresh Token에 대해선 언급하지 않았다.
- 하지만 Refresh Token을 사용하는 경우가 많다.

### Refresh Token 이란

- Access Token 에는 사용 가능한 유효 기간이 있다.
- 이 유효 기간이 지난 경우, 아예 처음부터 다시 재발급 받는 방법이 있고, Refresh Token을 사용해서 재발급 받는 방법이 있다.

### OAuth에서의 Refresh Token

![Untitled](/assets/img/2022-07-25-Tech_OAuth/Untitled%2010.png)

<br/>

## 정리

- OAuth 절차를 통해, Resource Server로부터 유저(Resource Owner)의 식별 토큰을 받을 수 있다.
- Access Token의 본질은 제 3자의 API를 사용하기 위함이다.
- 이 토큰이 유저의 ID·PW를 대신할 수 있으므로, 이 토큰 기반의 로그인 기능을 만들 수 있다.
    - 이것이 소셜 로그인이다.
    - Federated Identity 라고도 불린다.