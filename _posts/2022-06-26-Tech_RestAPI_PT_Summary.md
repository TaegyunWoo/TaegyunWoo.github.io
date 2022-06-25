---
category: Tech
tags: [API]
title: "Naver D2 - '그런 REST API로 괜찮은가' PT 정리"
date:   2022-06-26 02:10:00 
lastmod : 2022-06-26 02:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# ‘그런 REST API로 괜찮은가’ 컨퍼런스 정리

> [Naver D2 - '그런 REST API로 괜찮은가' 발표 영상](https://www.youtube.com/watch?v=RP_f5dMoHFc)

<br/>

## REST의 탄생 배경

### Web이란

정보들을 하이퍼텍스트로 연결하여, 인터넷에서 정보 공유를 하는 방식이다.

### Web의 핵심요소

- 표현 형식
    - HTML
- 식별자
    - URI
- 전송 방법
    - HTTP

### Roy T. Fielding 가라사대…

- “이미 전세계에 보급된 Web을 망가뜨리지 않고, HTTP를 어떻게 더 발전시킬 수 있을까?”이라는 의문
- 그것이 바로 **HTTP Object Model** 이다.
- HTTP Object Model 을 **REST라는 이름으로 논문을 발표**했다.
- Flickr 라는 서비스가 REST 방식의 API를 제공하기 시작했다.
- 기존 방식인 SOAP보다 쉽고 편리하여, REST가 빠르게 발전하게 됐다.
    > SOAP: HTTP, HTTPS, SMTP 등을 통해 XML 기반의 메시지를 컴퓨터 네트워크 상에서 교환하는 프로토콜

<br/>

## REST

### REST란

- 분산 하이퍼미디어 시스템 (Ex. Web) 을 위한 **아키텍처 스타일**
- REST는 하이브리드 아키텍처 스타일로, 여러가지 아키텍처 스타일이 합쳐져 하나의 아키텍처 스타일이 된 것이다.
- 아키텍처 스타일?
    - 제약조건의 집합

### REST API란

- REST라는 아키텍처 스타일이 적용된 API
- **REST 아키텍처 스타일은 API뿐만 아니라, 여러 곳에도 적용이 가능하다.**
    - Ex. 서버와 웹브라우저 등

> 즉 REST는 일종의 방법론, REST API는 그 방법론이 적용된 API

### REST를 구성하는 스타일

- client - server
- stateless
- cache
- layered system
- code-on-demand
    - 서버에서 보낸 코드를 클라이언트가 실행할 수 있는 구조 (JavaScript)
- **uniform interface**
- ‘client - server’, ‘stateless’, ‘cache’, ‘layered system’, ‘code-on-demand’ 는 HTTP를 잘 활용했을 때 자연스럽게 지켜진다.
- **하지만 ‘uniform interface’는 잘 지켜지기 어렵다.**

<br/>

## Uniform Interface 아키텍처 스타일

### Uniform Interface 아키텍처 스타일의 제약조건

- Identification of Resources
    - URI로 자원을 식별할 수 있어야 한다.
    - 이미 대부분이 만족하는 조건
- Manipulation of Resources Through Representations
    - ‘자원 추가, 삭제, 수정 등의 작업에 대한 표현’이 HTTP 메시지에 포함되어야 한다.
    - 이미 대부분이 만족하는 조건
- **Self-Descriptive Message**
    - 메시지 자체가 스스로의 의미를 잘 나타내야 한다.
- **HyperMedia as the Engine Of Application State (HATEOAS)**
    - 애플리케이션의 상태는 HyperLink를 이용해 전달되어야 한다.

> 가장 중요한 것은 ‘Self-Descriptive Message’와 ‘HATEOAS’임.  
하나씩 자세히 설명할 예정임.

### **Self-Descriptive Message**

메시지 자체가 스스로의 의미를 잘 나타내야한다는 제약조건이다.

- 예시 1
    
    ```text
    GET / HTTP/1.1
    ```
    
    위 HTTP 요청 MSG.는 **목적지 정보가 빠져있어서 해당 제약조건을 만족시키기 못한다.**
    
    ```text
    GET / HTTP/1.1
    Host: www.example.org
    ```
    
    위와 같이 수정되어야 한다.
    
- 예시 2
    
    ```text
    HTTP/1.1 200 OK
    [{
      "op": "remove",
      "path": "/a/b/c"
    }]
    ```
    
    위 HTTP 응답 MSG.는 **어떤 문법으로 파싱해야 하는지 나타나지 않아서 해당 제약조건을 만족시키지 못한다.**
    
    ```text
    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [{
      "op": "remove",
      "path": "/a/b/c"
    }]
    ```
    
    `[` , `{` , `”` , `:` 의 의미를 알 수 있어, 파싱이 가능하다.
    
    **하지만 `op` , `path` 의 의미는 여전히 모른다.**
    
    ```text
    HTTP/1.1 200 OK
    Content-Type: application/json-patch+json
    
    [{
      "op": "remove",
      "path": "/a/b/c"
    }]
    ```
    
    `application/json-patch+json` 이라는 Content-Type 표준에, `op` 와 `path` 의 의미가 정의되어 있다.
    
    따라서, 모든 내용을 정확하게 해석할 수 있다.
    
<br/>

### **HyperMedia as the Engine Of Application State (HATEOAS)**

애플리케이션의 상태는 HyperLink를 통해 전달되어야 한다는 제약조건이다.

- 예시 1
    
    ![Untitled](/assets/img/2022-06-26-Tech_RestAPI_PT_Summary/Untitled.png)
    
    **각 상태전이가 “해당 페이지에 있는 링크를 따라서 정의"되었기 때문에, 이것은 HATEOAS를 만족시킨다.**
    
- 예시 2
    
    ```text
    HTTP/1.1 200 OK
    Content-Type: test/html
    
    <html>
    <head></head>
    <body>
      <a href="/test">test</a>
    </body>
    </html>
    ```
    
    **`<a>` 태그를 통해 하이퍼링크를 만들어 상태를 전이시키므로, 이것은 HATEOAS를 만족시킨다.**
    
- 예시 3
    
    ```text
    HTTP/1.1 200 OK
    Content-Type: test/html
    Link: </article/1>; rel="previous",
          </article/3>; rel="next"
    
    {
      "title": "The Second Article",
      "content": "Blah~ Blah~"
    }
    ```
    
    **`Link` 헤더를 통해 하이퍼링크로 다른 자원을 가르키므로, 이것 역시 HATEOAS를 만족시킨다.**
    
    > `Link` 헤더: 이 자원(응답MSG.)과 연결된 다른 자원을 하이퍼링크로 가르키는 헤더,  
    이미 HTTP 표준에 포함된 헤더임
    > 

### Uniform Interface 아키텍처 스타일의 필요성

- 지금까지 Uniform Interface 라는 아키텍처 스타일을 살펴보았다.
- 그렇다면 이러한 아키텍처 스타일이 왜 필요한 것일까?
- **그것은 바로, ‘독립적 진화’를 위해서 이다.**

<br/>

## 독립적 진화

### 독립적 진화란

- 서버와 클라이언트가 각각 독립적으로 진화한다는 의미이다.
- **즉 서버의 기능이 변경되어도, 클라이언트를 업데이트할 필요가 없다는 의미이다.**
    - Ex. API가 서버에서 삭제되거나 변경되어도, 웹브라우저는 전혀 변경될 필요가 없다.

### 독립적 진화와 REST

- 독립적 진화는 REST라는 아키텍처 스타일을 만들게 된 계기와 일맥상통한다.
    - REST 아키텍처 스타일을 만들게 된 계기: “이미 보급된 Web을 망가뜨리지 않고, HTTP를 업그레이드시키자.”
- **즉 REST의 목표가 ‘독립적인 진화'이므로, Uniform Interface가 반드시 지켜져야 REST를 제대로 적용했다고 할 수 있다.**

### Web에 적용된 REST

- 이런 REST가 잘 지켜지고 있는 것이 바로 웹이라고 할 수 있다. (REST API가 아님)
    - 웹페이지(Server)를 변경했다고, 웹브라우저(Client)를 업데이트할 필요는 없다.
    - 웹브라우저를 업데이트했다고, 웹페이지를 변경할 필요도 없다.
    - HTTP 명세가 변경되어도 웹은 잘 동작한다.
        - HTTP 2.0 이 출시되어도, 여전히 기존 웹브라우저로 잘 접속할 수 있다.
    - HTML 명세가 변경되어도 웹은 잘 동작한다.
        - HTML 5.0 이 출시돼 이것으로 작성된 웹페이지를 기존  웹브라우저로 잘 접속할 수 있다.
- **하지만 모바일 앱은 서버가 변경되면 업데이트가 필수적이다.**
    - 따라서 이 경우, REST 아키텍처 스타일이 아니라고 할 수 있다.

### Self-Descriptive와 HATEOAS가 독립적 진화에 있어서 중요한 이유

- Self-Descriptive
    - **서버나 클라이언트가 변경되더라도, 오고가는 메시지는 언제나 해석이 가능하다.**
    - 그 자체로 의미를 알 수 있으므로
- HATEOAS
    - 어디서 어디로 전이가 가능한지 미리 결정되지 않는다.
    - 어떤 상태로 전이가 완료되고나서야 그 다음 전이될 수 있는 상태가 결정된다.
    - **즉, 특정 페이지에 들어가고 나서야 그 다음 상태로 전이가 가능하다.**
    - **따라서, 서버가 링크를 자유롭게 수정할 수 있으므로 독립적 진화가 가능하다.**

<br/>

## REST API

### REST가 적용된 API?

- REST API는 REST 아키텍처 스타일을 따라야 한다.
- **하지만 오늘날 스스로 REST API라고 하는 API들은 대부분 REST 아키텍처 스타일을 따르지 않는다.**
- 당연하게도, REST의 모든 조건을 따라야 REST API이다.
    
    > REST API라고 부르려면 반드시 모두 지켜야 한다고, ‘Roy T. Fielding’가 말했다..!


### Web은 REST가 잘 적용되는데, 왜 API는 어려울까

- Web vs API
    
    
    |  | 흔한 웹 페이지 | HTTP API |
    | --- | --- | --- |
    | Protocol | HTTP | HTTP |
    | Communication | 사람·기계 | **기계·기계** |
    | Media Type | HTML | **JSON** |
    
    **왜냐하면 Media Type에 차이가 있기 때문이다.**
    
- HTML vs JSON
    
    
    |  | HTML | JSON |
    | --- | --- | --- |
    | Hyperlink | 가능 (a 태그 등) | 정의되어 있지 않음 |
    | Self-Descriptive | 가능 (HTML 명세) | 불완전* |
    - Hyperlink
        - HTML 에서는 a 태그 등을 통해, 하이퍼링크를 정의할 수 있다.
        - JSON 은 아예 지원하지 않는다.
    - Self-Descriptive
        - HTML 명세에는 각 태그가 어떤 의미인지 정의되어 있기 때문에, Self-Descriptive가 가능하다.
        - JSON 의 경우 문법해석은 가능하지만, 의미를 해석하려면 별도의 문서 (API 문서 등)이 필요하다.

<br/>

## HTML vs JSON

### HTML, JSON

아래는 HTML과 JSON을 비교하기 위해, 사용할 예시이다.

- HTML 예시
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: text/html
    
    <html>
    <body>
    <a href="https://todos/1">회사 가기</a>
    <a href="https://todos/2">집에 가기</a>
    </body>
    </html>
    ```
    
- JSON 예시
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: application/json
    
    [
      {"id": 1, "title": "회사 가기"},
      {"id": 2, "title": "집에 가기"}
    ]
    ```
    

### HTML

```text
GET /todos HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Content-Type: text/html

<html>
<body>
<a href="https://todos/1">회사 가기</a>
<a href="https://todos/2">집에 가기</a>
</body>
</html>
```

- **Self-Descriptive을 만족한다.**
    1. 응답 메시지의 Content-Type을 보고 Media Type이 `text/html` 임을 확인한다.
    2. HTTP 명세에 Media Type은 IANA에 등록되어있다고 하므로, IANA에서 `text/html` 의 설명을 찾는다.
    3. IANA에 따르면 `text/html` 의 명세는 ‘http://www.w3.org/TR/html’ 이므로 링크를 찾아가 명세를 해석한다.
    4. 명세에 모든 태그의 해석방법이 구체적으로 나와있으므로 이를 해석하여, 문서 저자가 사용자에게 해석 방법을 제공해 줄 수 있다.
- **HATEOAS를 만족한다.**
    - a 태그를 이용해, 표현된 링크를 통해 다음 상태로 전이될 수 있으므로 HATEOAS를 만족한다.

### JSON

```text
GET /todos HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {"id": 1, "title": "회사 가기"},
  {"id": 2, "title": "집에 가기"}
]
```

- **Self-Descriptive을 만족하지 못한다.**
    1. 응답 메시지의 Content-Type을 보고 Media Type이 `application/json` 임을 확인한다.
    2. HTTP 명세에 Media Type은 IANA에 등록되어있다고 하므로, IANA에서 `application/json` 의 설명을 찾는다.
    3. IANA에 따르면 `application/json` 의 명세는 draft-ietf-jsonbis-rfc7159bis-04 이므로, 링크를 찾아가 명세를 해석한다.
    4. 명세에 json 문서를 파싱하는 방법이 명시되어있으므로 성공적으로 파싱에 성공한다.  
    **그러나 “id”와 “title”이 무엇을 의미하는지 알 방법은 없다.**
- **HATEOAS를 만족하지 못한다.**
    - 다음 상태로 전이할 링크가 없다.

<br/>

## 진정한 REST API 로 만들기

### Self-Descriptive 만족시키기

- 방법 1: Media Type 활용
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: application/vnd.todos+json
    
    [
      {"id": 1, "title": "회사 가기"},
      {"id": 2, "title": "집에 가기"}
    ]
    ```
    
    미디어 타입이 변경되었다.
    
    1. 미디어 타입을 하나 정의한다.
    2. 미디어 타입 문서를 작성한다. 이 문서에 “id”와 “title”이 의미하는 바를 정의한다.
    3. **IANA에 미디어 타입을 등록한다. 이 때 만든 문서를 미디어 타입의 명세로 등록한다.**
    4. 이제 이 메시지를 보는 사람은 명세를 찾아갈 수 있으므로, 이 메시지의 의미를 온전히 해석할 수 있다.
    
    > 하지만 다소 복잡하다.  
    다음 방법을 보자.
    > 
- 방법 2: Profile
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: application/json
    Link: <http://example.org/docs/todos>; rel="profile"
    
    [
      {"id": 1, "title": "회사 가기"},
      {"id": 2, "title": "집에 가기"}
    ]
    ```
    
    `Link` 헤더가 추가되었다.
    
    1. “id”와 “title”이 의미하는 바를 정의한 명세를 작성한다.
    2. Link 헤더에 profile relation으로 해당 명세를 링크한다.
    3. 이제 이 메시지를 보는 사람은 명세를 찾아갈 수 있으므로, 이 메시지의 의미를 온전히 해석할 수 있다.
    - 단점
        - 클라이언트가 Link 헤더(RFC 5988)와 profile(RFC 6906)을 이해해야 한다.
        - Content Negotiation 불가능
            
            > Content Negotiation: 동일한 URI에서 다른 버전의 문서를 제공 할 수 있으므로 사용자 에이전트는 자신의 기능에 가장 적합한 버전을 지정하는 것  
            Ex. 동일한 URI이지만 `Accept-Language` 헤더에 따라, 다른 언어로 제공
            > 

### HATEOAS 만족시키기

- 방법 1: data 활용
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: application/json
    Link: <http://example.org/docs/todos>; rel="profile"
    
    [
      {
        "link": "https://example.org/todos/1",
        "title": "회사 가기"
      },
      {
        "link": "https://example.org/todos/2",
        "title": "집에 가기"
      }
    ]
    ```
    
    - “id” 대신 “link”라는 key가 들어가고, 관련 하이퍼링크를 value로 제공한다.
    
    아래와 같이, 표현할 수도 있다.
    
    ```text
    GET /todos HTTP/1.1
    Host: example.org
    
    HTTP/1.1 200 OK
    Content-Type: application/json
    Link: <http://example.org/docs/todos>; rel="profile"
    
    {
      "links": {
        "todo": "https://example.org/todos/{id}"
      },
      "data": [{
        "id": 1,
        "title": "회사 가기"
      },
      {
        "id": 2,
        "title": "집에 가기"
      }]
    }
    ```
    
    - “link”와 “data”를 따로 구분하고, URI를 템플릿 형태로 변경하였다.
- 방법 2: HTTP 헤더 활용
    
    ```text
    POST /todos HTTP/1.1
    Content-Type: application/json
    
    {
      "title": "점심 약속"
    }
    
    HTTP/1.1 204 No Content
    Location: /todos/1
    Link: </todos/>; rel="collection"
    ```
    
    - Link, Location 등의 헤더로 링크를 표현한다.

## 정리

- 오늘날 대부분의 “REST API”는 사실 REST를 따르지 않고 있다.
- REST의 제약조건 중에서 특히 **Self-Descriptive** 와 **HATEOAS** 를 잘 만족하지 못한다.
- REST는 긴 시간에 걸쳐(수십년) 진화하는 웹 애플리케이션을 위한 것이다.
- REST를 따를 것인지는 API를 설계하는 이들이 스스로 판단하여 결정해야 한다.
- REST를 따르겠다면, **Self-Descriptive** 와 **HATEOAS**를 만족시켜야 한다.
    - Self-Descriptive는 **custom media type**이나 **profile link relation** 등으로 만족시킬 수 있다.
    - HATEOAS는 HTTP 헤더나 본문에 **링크**를 담아 만족시킬 수 있다.
- REST를 따르지 않겠다면, “REST를 만족하지 않는 REST API”를 뭐라고 부를지 결정해야 할 것이다.
    - HTTP API라고 부를 수도 있고
    - 그냥 이대로 REST API라고 부를 수도 있다.
        
        > REST 창시자는 싫어하지만…
        > 

<br/>

### REST를 만족하지 않는 REST API 가이드

- 아래는 MicroSoft에서 제공하는 REST API 가이드이다.
- 하지만 이 가이드는 **‘REST를 만족하지 않는 REST API 가이드’** 이며, MicroSoft도 인정했다.
- 어떻게 API를 설계해야하는지에 대한 상세한 가이드이다.

[https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md)