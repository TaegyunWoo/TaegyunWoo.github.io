---
category: React
tags: [React]
title: "AsyncAPI : 비동기 API를 문서화하는 방법"
date: 2023-12-10 14:50:00 +0900
lastmod: 2023-12-10 14:50:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

최근 Rannect라는 랜덤 채팅 프로젝트를 진행하고 있습니다. 메인 기능인 채팅을 포함해서 대부분의 기능이 웹소켓 등을 기반으로 동작하게 됩니다.

프로젝트를 진행하던 중, 웹소켓 API를 어떻게 Documentation 할 수 있을지 고민하게 되었습니다. 기존 HTTP API의 경우에는 OpenAPI(Swagger)라는 아주 좋은 스펙과 문서화 도구를 사용하면 되는데, 웹소켓 API는 OpenAPI의 범위를 벗어납니다. 그래서 관련 자료를 찾아보던 중, AsyncAPI에 대해 알게 되었습니다.

AsyncAPI는 REST API처럼 비동기 API (Event-Driven Architecture)를 간편하게 다룰 수 있게(문서화) 하는 오픈소스입니다. AsyncAPI는 비동기 API의 표준처럼 사용됩니다. 이번 포스팅에서는 이 AsyncAPI에 대해 알아보고, 어떻게 사용하는지 알아보도록 하겠습니다.

공식 문서는 아래 링크를 통해 접속하실 수 있습니다.

[https://www.asyncapi.com/](https://www.asyncapi.com/)

<br/><br/>

## AsyncAPI란

공식 문서에서는 AsyncAPI를 아래와 같이 설명합니다.

> AsyncAPI는 EDA(Event-Driven Architecture)의 문제점을 개선하는 오픈 소스입니다. 우리의 최종적인 목표는 EDA를 REST API처럼 쉽게 사용할 수 있도록 하는 것입니다.

AsyncAPI가 제공하는 핵심은 아래와 같습니다.

- 문서화, 코드 생성의 자동화
- 검색
- 이벤트 관리

<br/><br/>

## Event-Driven Architecture

그럼 먼저, 여기서 계속 등장하는 단어 EDA(Event-Driven Architecture)란 무엇인지 알아볼까요?

우리는 REST API와 같은 동기 방식의 통신에 익숙합니다. 서버에 요청을 보내면, 응답이 올 때까지 기다리는거죠. 그리고 전통적인 Web이 바로, 이러한 동기 방식 API를 활용한 케이스이죠.

하지만 IT 기술이 발전함에 따라, 요청을 보내고 서버의 응답을 즉각적으로 받지 않는 것이 가능해졌습니다. 그리고 아래처럼 단순히 ‘어떤 일이 일어났다’는 사실(이벤트)만을 전달하고 싶은 경우가 있죠.

- 한 사용자가 조금 전에 로그인한 경우
- 새로운 팔로워가 생긴 경우

EDA는 이러한 이벤트를 기반으로 서비스 간 통신을 수행하는 아키텍처를 의미합니다.

<br/>

### 핵심 개념

![Untitled](/assets/img/2023-12-10-Tech_AsyncAPI/Untitled.png)

EDA의 대부분의 경우, 위처럼 메시지 브로커가 통신을 중계하게 됩니다. 주요 구성 요소를 자세히 설명하면 아래와 같습니다.

- **Message Broker**
  - 메시지를 수신받고, 그것을 수신받고 싶어하는 다른 서비스에 전달하는 역할을 수행합니다.
  - 메시지가 전달될 때까지 메시지를 저장하고 있기 때문에, 장애 대처 능력이 있습니다.
  - 메시지 브로커의 대표적인 예시로는, RabbitMQ, Apache Kafka, Solace 등이 있습니다.
- **Publisher/Subscriber**
  - Publisher(생산자)는 메시지 브로커에게 메시지를 전달하는 애플리케이션을 지칭합니다.
  - Subscriber(소비자)는 메시지 브로커로부터 ‘특정 유형의 메시지를 받고 싶다’고 알리고, 관련 메시지를 브로커로부터 Push 받는 애플리케이션입니다.
  - 하나의 애플리케이션이 생산자이자 소비자가 될 수도 있습니다.
- **Message**
  - 정보의 한 조각을 의미하며, Publisher가 이것을 Broker에게 전달하고 Subscriber가 받게 됩니다.
  - 어떤 정보든 포함될 수 있습니다. 대부분의 경우 이벤트나 수행할 명령을 갖게 됩니다.
    - 이벤트: 어떤 일이 발생한 사실
    - 명령: ‘구독자들에게 무엇을 하라’고 지시
- **Channel**
  - 메시지 브로커는 여러 채널로 통신할 수 있도록 지원합니다. 채널을 통해, Subscriber는 원하는 유형의 정보만을 수신받을 수 있습니다.
  - 특정 채널로 하나의 목적을 갖는 메시지만 처리하는 것이 좋습니다.

자 지금까지 EDA에 대해 간략하게나마 설명했으니, 다시 AsyncAPI에 대해 알아볼까요?

<br/><br/>

## AsyncAPI의 스펙

기본적으로 AsyncAPI는 **OpenAPI의 스펙을 차용**하여 만들어졌습니다.

![Untitled](/assets/img/2023-12-10-Tech_AsyncAPI/Untitled%201.png)

위 그림의 왼쪽이 OpenAPI 3.0의 주요 구성 요소이고, 오른쪽은 AsyncAPI 2.0(현재는 3.0까지 나왔습니다.)의 주요 구성 요소입니다. 물론 외우실 필요는 없고, 간략하게 어떤 부분이 어떻게 치환됐구나~ 정도로만 보셔도 되겠습니다.

비동기 통신에 맞게끔 바뀐 주요한 차이점을 살펴보면 아래와 같습니다.

- `Paths` → `Channel`
- `Operation(Request, Response)` → `Message(Headers, Payload)`

중요한 것은 대부분 요소가 매우 유사하다는 것입니다. 이를 통해서, 개발자들이 좀 더 익숙하게 AsyncAPI를 사용할 수 있도록 유도한 것 같습니다.

백문의 불여일견! 바로, AsyncAPI 문서를 만들어보겠습니다.

<br/><br/>

## AsyncAPI 문서 만들기

```yaml
asyncapi: 3.0.0
info:
  title: Hello world application
  version: "0.1.0"
channels:
  hello:
    address: "hello"
    messages:
      sayHelloMessage:
        payload:
          type: string
          pattern: "^hello .+$"
operations:
  receiveHello:
    action: "receive"
    channel:
      $ref: "#/channels/hello"
```

`YAML` 파일로 작성된 AsyncAPI 문서입니다. `"hello {name}"` 이라는 메시지를 받는 API를 설명했는데요. 이걸 토대로, 각 요소에 대해 하나씩 알아봅시다.

<br/>

### `aysncapi`

```yaml
asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
#channels:
#  hello:
#    address: 'hello'
#    messages:
#      sayHelloMessage:
#        payload:
#          type: string
#          pattern: '^hello .+$'
#operations:
#  receiveHello:
#    action: 'receive'
#    channel:
#      $ref: '#/channels/hello'
```

첫번째 줄을 먼저 볼까요? `asyncapi` 라는 키를 통해서 문서의 타입이 AsyncAPI라는 것을 나타내고, 값인 `3.0.0` 을 사용했습니다.

이것이 꼭 첫번째 줄에 들어가진 않아도 되지만, 첫번째 줄에 작성하는 것을 권장한다고 공식문서에서 설명합니다. 그러니 웬만하면 첫번째 줄에 작성해줍시다.

<br/>

### `info`

```yaml
#asyncapi: 3.0.0
info:
  title: Hello world application
  version: "0.1.0"
#channels:
#  hello:
#    address: 'hello'
#    messages:
#      sayHelloMessage:
#        payload:
#          type: string
#          pattern: '^hello .+$'
#operations:
#  receiveHello:
#    action: 'receive'
#    channel:
#      $ref: '#/channels/hello'
```

`info` 는 애플리케이션에 대한 최소한의 정보를 갖습니다. 하위 속성으로는 `title` 과 `version` 을 갖는데, 각각 ‘애플리케이션의 이름’과 ‘애플리케이션의 버전’을 나타냅니다.

다만 `version`의 경우, API가 변경될 때마다 변경해주는 것을 권장합니다.

<br/>

### `channels`

```yaml
#asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
channels:
  hello:
    address: "hello"
    messages:
      sayHelloMessage:
        payload:
          type: string
          pattern: "^hello .+$"
#operations:
#  receiveHello:
#    action: 'receive'
#    channel:
#      $ref: '#/channels/hello'
```

`channels` 에는 각 채널에 대한 정보가 기입됩니다. 채널의 이름, 주소, 메시지의 형식 등의 정보가 담깁니다.

- `hello` : 채널 이름
- `address` : 채널 주소
- `sayHelloMessage` : 메시지 이름
- `payload` : 메시지에 담길 데이터 관련 정보
- `type` : 메시지의 데이터 타입
- `pattern` : 메시지 페이로드의 형식

<br/>

### `operations`

```yaml
#asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
#channels:
#  hello:
#    address: 'hello'
#    messages:
#      sayHelloMessage:
#        payload:
#          type: string
#          pattern: '^hello .+$'
operations:
  receiveHello:
    action: "receive"
    channel:
      $ref: "#/channels/hello"
```

`operations` 는 애플리케이션이 메시지를 송신하고 수신하는 등의 작업을 설명합니다. 각 Operation은 `receiveHello` 와 같은, 유니크한 식별자가 사용됩니다.

위 예시는 `Hello world application` 이 `hello` 채널에서 `sayHelloMessage` 메시지를 수신하는 소비자라는 것을 나타냅니다. 그래서 `action` 키의 값이 `'receive'` 가 되는 것이죠.

<br/><br/>

## Request/reply 패턴

이번에는 ping pong을 주고 받는 API의 요청/응답을 구현해보겠습니다.

<br/>

### 변하지 않는 응답 주소인 경우

이미 응답 주소를 알고 있고 주소가 변경되지 않는다면, **요청자**의 API 문서를 아래와 같이 구현할 수 있습니다.

요청자가 Ping 채널에 메시지를 발송하고, Pong 채널로 응답을 받게 됩니다.

```yaml
#asyncapi: 3.0.0
#info:
#  title: Ping/pong example with static reply channel
#  version: 1.0.0
#  description: Requester example that initiates the request/reply pattern on a different channel than the reply is using
channels:
  ping:
    address: /ping
    messages:
      ping:
        $ref: "#/components/messages/ping"
  pong:
    address: /pong # 응답받을 채널 주소를 명시적으로 기입함
    messages:
      pong:
        $ref: "#/components/messages/pong"
operations:
  pingRequest: # 요청 오퍼레이션 정보
    action: send # Ping 채널에 메시지를 발송합니다.
    channel: # 발송할 채널의 정보
      $ref: "#/channels/ping"
    reply: # 응답 관련 정보
      channel: # 응답받을 채널의 정보
        $ref: "#/channels/pong"
components:
  messages:
    ping:
      payload:
        type: object
        properties:
          event:
            type: string
            const: ping
    pong:
      payload:
        type: object
        properties:
          event:
            type: string
            const: pong
```

`operations` 의 `reply` 를 통해, 응답받을 채널의 정보를 `'#/channels/pong'` 로 설정했습니다. `channels` 의 `pong` 을 보게 되면, **명시적으로 응답 채널의 주소 `/pong` 이 설정된 것을 확인**할 수 있습니다. 이렇게 변경되지 않는 주소라면, 명시적으로 주소를 표현하면 됩니다.

<br/>

### 동적으로 변화하는 응답 주소인 경우

어떤 요청 메시지를 보낼 때 헤더에 응답 주소를 담아서 보내면, 해당 응답 주소로 응답해야 하는 경우처럼, 런타임 상황에서 응답 주소가 변경되는 경우 어떻게 해야할까요?

```yaml
#asyncapi: 3.0.0
#info:
#  title: Ping/pong example with reply specified as dynamic information provided in the runtime
#  version: 1.0.0
#  description: Example document for an application that processes ping requests and replies to the address dynamically specified by the requestor in the message header
channels:
  ping:
    address: /ping
    messages:
      ping:
        $ref: "#/components/messages/ping"
  pong:
    address: null # 주소를 null로 비워둠
    messages:
      pong:
        $ref: "#/components/messages/pong"
operations:
  pingRequest:
    action: send
    channel:
      $ref: "#/channels/ping"
    reply:
      address:
        description: Reply is sent to topic specified in 'replyTo' property in the message header
        location: "$message.header#/replyTo" # 응답 주소가 담긴 위치 설정
      channel:
        $ref: "#/channels/pong"
components:
  messages:
    ping:
      headers:
        type: object
        properties:
          replyTo: # 이 요청 메시지를 수신 후, 응답할 주소 정보가 담긴 헤더
            type: string
            description: Provide path to which reply must be provided
          requestId:
            type: string
            format: uuid
            description: Provide request id that you will use to identify the reply match
      payload:
        type: object
        properties:
          event:
            type: string
            const: ping
      correlationId:
        $ref: "#/components/correlationIds/pingCorrelationId"
    pong:
      headers:
        type: object
        properties:
          requestId:
            type: string
            format: uuid
            description: Reply message must contain id of the request message
      payload:
        type: object
        properties:
          event:
            type: string
            const: pong
      correlationId:
        $ref: "#/components/correlationIds/pingCorrelationId"
  correlationIds:
    pingCorrelationId:
      location: "$message.header#/requestId"
```

위처럼 `channels.pong` 의 `address` 를 `null`로 비워둡니다. 그리고 `operations` 의 `reply.address` 에서 `location` 속성을 사용하면 됩니다.

위 예시에서는 `location` 속성의 값으로 `"$message.header#/replyTo"` 이 사용됐습니다. 이것은 요청 메시지의 헤더 중 `replyTo` 헤더의 값을 사용한다는 의미입니다.

<br/><br/>

## 서버 정보 추가하기

이번에는 AsyncAPI 문서에 서버 관련 정보를 추가해보겠습니다.

```yaml
#asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
servers:
  production:
    host: broker.mycompany.com
    protocol: amqp
    description: This is "My Company" broker.
#channels:
#  hello:
#    address: 'hello'
#    messages:
#      sayHelloMessage:
#        payload:
#          type: string
#          pattern: '^hello .+$'
#operations:
#  receiveHello:
#    action: 'receive'
#    channel:
#      $ref: '#/channels/hello'
```

서버 관련 정보들을 `servers` 섹션 하위에 작성하면 됩니다. 위 예시에서는 `production` 이라는 이름의 서버의 정보 (`host` , `protocol` , `description`)이 추가됐습니다.

- `host` : 서버 주소
- `protocol` : 통신에 사용될 프로토콜 (mqtt, kafka, ws, http 등)
- `description` : 서버에 대한 설명

`servers` 섹션에는 메시지를 주고 받기 위해서, **애플리케이션이 연결해야 하는 서버의 정보**가 담깁니다. 따라서 아래와 같이 작성할 수 있습니다.

1. Kafka나 RabbitMQ와 같은 메시지 브로커에 연결되어야 한다면, 해당 브로커의 URL을 지정하면 됩니다.
2. REST API와 같은 전통적인 클라이언트-서버 모델을 사용한다면, 해당 서버의 URL을 지정합니다.

<br/><br/>

## 보안 설명 추가하기

이번에는 보안과 관련된 정보들을 추가해볼까요?

```yaml
#asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
servers:
  production:
    host: broker.mycompany.com
    protocol: amqp
    description: This is "My Company" broker.
    security:
      - type: userPassword
# 이하 생략
```

위 예시에서는 `servers` 섹션의 특정 서버 부분에 `security` 가 추가되었습니다. 이 `security` 은 보안에 사용될 여러 메커니즘을 yaml 배열 문법으로 표현할 수 있습니다. 여기에서는 오직 하나의 아이템만 사용됐습니다. 그리고 해당 아이템의 `type` 이 `userPassword` 이므로, 사용자 비밀번호 방식을 사용한다는 것을 표현했습니다.

참고로 `-` 문자가 등장했는데, 이것은 YAML의 문법 중 하나로 배열을 표현합니다. 혹시 이런 문법을 처음 보신다면, 아래 예시를 참고해주세요.

```yaml
# 단순 배열 ( fruits = ["apple", "banana", "grape"] )
fruits:
  - apple
  - banana
  - grape

# 객체 표현
apple:
  color: red
  weight: 200g
  price: 1000won

# 객체 배열 표현
fruits:
  - color: red
    weight: 200g
    price: 1000won
  - color: yellow
    weight: 400g
    price: 2000won
  - color: purple
    weight: 300g
    price: 4000won
```

<br/>

### `components.securitySchemes` 사용하기

위 예시에서는 보안 관련 정보를 직접 `server` 섹션에 추가했습니다. 사실 이러한 방법 보다는, 아래처럼 재사용 가능하도록 작성하는 것이 권장됩니다.

```yaml
#asyncapi: 3.0.0
#info:
#  title: Hello world application
#  version: '0.1.0'
servers:
  production:
    host: broker.mycompany.com
    protocol: amqp
    description: This is "My Company" broker.
    security:
      - $ref: "#/components/securitySchemes/user-password"
components:
  securitySchemes:
    user-password:
      type: userPassword
# 이하 생략
```

<br/><br/>

## 정리하며

지금까지 AsyncAPI가 무엇이고, 어떻게 사용하면 되는지 알아봤습니다. 이 AsyncAPI는 공식적인 표준은 아니지만, 거의 표준처럼 사용될 정도로 발전 중입니다. 그래서 리눅스 재단과 API 테스트 도구인 Postman과도 협력 중이라고 합니다. ([관련 아티클](https://www.asyncapi.com/blog/asyncapi-joins-linux-foundation))

지금까지 기본적인 스펙과 사용방법에 대해서만 다뤘고, 나머지 부분은 직접 사용하며 학습해나가야 하겠습니다.

AsyncAPI를 기반으로 한 다양한 도구들을 살펴보시려면, [https://www.asyncapi.com/tools](https://www.asyncapi.com/tools) 를 참고하시면 됩니다.

저는 스프링 기반으로 서버를 구현하고 있어서, [SpringWolf](https://www.springwolf.dev/) 라는 라이브러리를 도입할 예정입니다. 혹시 관심이 있으시다면, 한번 살펴보시는 것도 좋겠습니다.

감사합니다.

<br/><br/>

## References

- [https://www.asyncapi.com/](https://www.asyncapi.com/)
- [https://sungyong.medium.com/asyncapi-event-driven-architecture의-사실상-표준-api-f3adcbd24d08](https://sungyong.medium.com/asyncapi-event-driven-architecture%EC%9D%98-%EC%82%AC%EC%8B%A4%EC%83%81-%ED%91%9C%EC%A4%80-api-f3adcbd24d08)
- [https://inpa.tistory.com/entry/YAML-📚-yaml-개념-문법-이해하기-💯-총정리](https://inpa.tistory.com/entry/YAML-%F0%9F%93%9A-yaml-%EA%B0%9C%EB%85%90-%EB%AC%B8%EB%B2%95-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)
