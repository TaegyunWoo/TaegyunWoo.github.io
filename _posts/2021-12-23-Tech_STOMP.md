---
category: Tech
tags: [WebSocket]
title: "STOMP와 Spring-Boot"
date:   2021-12-23 17:55:00 
lastmod : 2021-12-23 17:55:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# STOMP

## 개요

### STOMP란?

- Simple/Stream Text Oriented Messaging Protocol의 약자로, 간단한 문자 기반 메시징 프로토콜이다.
- HTTP와 마찬가지로, Frame을 사용해 전송하는 프로토콜이다.
    - Frame: 주소와 명령, 명령 수행을 위한 데이터가 모두 포함된 데이터
- TCP/IP 위에서 동작한다.
- STOMP는 일종의 스펙(명세서)으로, 기능 구현은 전적으로 서버에 달려있다.
    - STOMP 스펙에 맞춰 직접 구현할 필요가 없다. `org.springframework.boot:spring-boot-starter-websocket` 이 관련 로직을 제공한다.
- STOMP는 웹소켓 위에 얹어 함께 사용할 수 있는 하위(서브) 프로토콜이다.
- **PUB/SUB 구조를 갖는다.**
- **Message Broker가 중간에서 메시지를 전달한다.**

<br/>

### Pub/Sub 구조

- 아래 요소들로 이루어져, 메시징 처리를 하는 구조를 뜻한다.
    - **메시지 공급 주체 (Publisher)**
    - **메시지 소비 주체 (Subscriber)**
    - **메시지 주제 (Topic)**
- 행위별 정리
    - 채팅방 생성 ⇒ Topic를 생성하는 것
    - 특정 채팅방 입장 ⇒ 해당 Topic에 대한 Subscriber가 되는 것
    - 특정 채팅방에 채팅을 보내는 것 ⇒ 해당 Topic에 대해 Publisher가 되는 것

<br/>

### Message Broker

- Message Broker란
    - Publisher가 전달한 msg를 Subscriber에게 전달하는 역할을 수행한다.
    - STOMP가 Pub/Sub 구조로 동작할 수 있게 만드는 주체이다.

<br/>

### WebSocket vs STOMP

- WebSocket은 메시지의 형식이 정해져 있지 않다.
- 따라서 대규모 프로젝트에 WebSocket을 적용하기엔, 어려움이 많다.
- 그에 반해 STOMP는 메시지 송·수신 시, **미리 정의된 규칙을 사용**한다.
- 또한 이름에서 알 수 있듯이, **텍스트 기반 프로토콜**이다.
    - 따라서 메시지 작성 시, 텍스트로 모든 것을 작성해야 한다.

<br/>

### STOMP의 Frame 구조

- **Frame은 텍스트 라인들로 이루어져 있다.**
    - 텍스트 기반 프로토콜이기 때문에
- UTF-8 인코딩 방식을 사용해야 한다.

```xml
COMMAND 
header1:value1 
header2:value2 

Body^@
```

- 구성요소
    - **명령(COMMAND) - 클라이언트가 요청 시, 사용하는 주요 COMMAND**
        - `CONNECT`
            - 클라이언트의 TCP/IP 연결 요청
            - `accept-version` 헤더에 수용할 STOMP 버전을 반드시 명시해야 한다.
            - `host` 헤더에 연결할 호스트명을 반드시 명시해야 한다.
            
            ```xml
            CONNECT
            accept-version:1.0,1.1,2.0
            host:stomp.github.org
            
            ^@
            ```
            
            > 당연히 바디부분은 필요없다.
            > 
        - `SEND`
            - 클라이언트(Publisher)가 msg를 보낸다.
            - `destination` 헤더에 msg를 보낼 Topic을 반드시 지정해야 한다.
            
            ```xml
            SEND
            destination:/queue/a
            
            hello queue a
            ^@
            ```
            
        - `SUBSCRIBE`
            - 클라이언트가 특정 Topic에 대해, 구독을 한다. ⇒ 해당 클라이언트는 Subscriber가 된다.
            - 해당 Topic으로 오는 모든 msg를 클라이언트(Subscriber)가 받게 된다.
            - `destination` 헤더로 Subscribe할 Topic을 반드시 지정해야 한다.
            - `id` 헤더로 유니크한 Subscribe Id를 설정해야 한다.
            
            ```xml
            SUBSCRIBE
            id:0
            destination:/queue/foo
            ack:client
            
            ^@
            ```
            
            > 당연히 `SUBSCRIBE` 에는 Body 부분이 필요없다.
            > 
        - `UNSUBSCRIBE`
            - 말 그대로, Subscribe을 취소한다.
            - `id` 헤더에 `Subscribe` 때 설정한 `id` 를 반드시 넣어야 한다.
            
            ```xml
            UNSUBSCRIBE
            id:0
            
            ^@
            ```
            
        - `DISCONNECT`
            - 연결을 해지한다. 해지 순서는 아래와 같다.
            1. 클라이언트 측 요청 msg
                
                ```xml
                DISCONNECT
                receipt:77
                
                ^@
                ```
                
            2. 이에 대한 서버 측 응답 msg
                
                ```xml
                RECEIPT
                receipt-id:77
                
                ^@
                ```
                
                - 이때 클라이언트가 보낸 `receipt` 의 값과 서버가 보낸 `receipt-id` 의 값이 같아야 한다.
            3. 소켓을 닫는다.
        
    <br/>

    - **명령(COMMAND) - 서버가 응답 시, 사용하는 주요 COMMAND**
        - `CONNECTED`
            - 클라이언트가 `CONNECT` 프레임을 보내면, 아래와 같이 응답해야 한다.
            - `version` 헤더에 현재 STOMP 버전을 명시해야 한다.
            
            ```xml
            CONNECTED
            version:1.2
            
            ^@
            ```
            
        - `MESSAGE`
            - 서버가 해당 Topic(`destination`)를 Subscribe하는 클라이언트들에게 msg를 보낸다.
            
            ```xml
            MESSAGE
            subscription:0
            message-id:007
            destination:/queue/a
            
            hello queue a^@
            ```
            
        - `RECEIPT`
            
            ```xml
            RECEIPT
            receipt-id:message-12345
            
            ^@
            ```
            
        - `ERROR`
            
            ```xml
            ERROR
            receipt-id:message-12345
            content-type:text/plain
            content-length:170
            message:malformed frame received
            
            The message:
            -----
            MESSAGE
            destined:/queue/a
            receipt:message-12345
            
            Hello queue a!
            -----
            Did not contain a destination header, which is REQUIRED
            for message propagation.
            ^@
            ```
            
    - **헤더(Header)**
    - **바디(Body)**

> 자세한 것은 [STOMP 레퍼런스](https://stomp.github.io/stomp-specification-1.2.html#STOMP_Frames) 참고
> 

<br/><br/>

## STOMP 구현 (With Spring-boot)

### 스프링에서의 STOMP 통신 흐름

![참고 영상: [[10분 테코톡] 아론의 웹소켓&스프링](https://www.youtube.com/watch?v=rvss-_t6gzg)](/assets/img/2021-12-23-Tech_STOMP/Untitled.png)

참고 영상: [[10분 테코톡] 아론의 웹소켓&스프링](https://www.youtube.com/watch?v=rvss-_t6gzg)

- 가정
    - 구독자(Subscriber)들이 `/topic` 이라는 Topic을 현재 구독 중이다.
    - 발신자(Publisher)가 `/topic` 에 msg를 보내고자 한다.
- 이때 발신자가 선택할 수 있는 선택지는 아래와 같다.
    - **`SEND` 프레임의 `destination` 헤더에 `/topic` 을 넣어, 바로 msg을 전달하는 방법**
    - **메시지 가공 및 처리를 하기 위해, `SEND` 프레임의 `destination` 헤더에 `/app` 을 넣어 msg를 전달하는 방법**

<br/>

### 웹소켓메시지브로커 등록: `WebSocketBrokerConfig`

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketBrokerConfig implements WebSocketMessageBrokerConfigurer {

	//메시지 브로커 관련 설정
	@Override
	public void configureMessageBroker(MessageBrokerRegistry registry) {
		//발신자가 선택할 수 있는 1번 방법
		registry.enableSimpleBroker("/queue", "/topic");

		//발신자가 선택할 수 있는 2번 방법 (메시지 가공)
		registry.setApplicationDestinationPrefixes("/app");
	}

	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/stomp-websocket").withSockJS();
	}

}
```

- `implements WebSocketMessageBrokerConfigurer`
    - STOMP 방식으로 구현하려면, 해당 인터페이스를 구현해야 한다.

<br/>

- `enableSimpleBroker("/queue", "/topic")`
    - 내장 메시지 브로커를 사용하도록 설정한다.
    - “`SEND` 프레임의 `destination` 헤더에 `/topic` 을 넣어, 바로 msg을 전달하는 방법” 에 해당된다.
    - **파라미터로 전달받은 값이 msg의 `destination` 헤더의 prefix로서 사용되면, 내장 메시지 브로커가 처리한다.**
    - 위 예시 코드의 경우, `/queue` 나 `/topic` 이 `destination` 헤더의 prefix로 사용되면 내장 메시지 브로커인 `SimpleBroker` 가 그 메시지를 받아 구독자들에게 전달해준다.
    - 관례적인 prefix들
        - **/queue** : 메시지가 일대일로 전송될 때
        - **/topic** : 메시지가 일대다로 브로드캐스팅 될 때

<br/>

- `setApplicationDestinationPrefixes("/app")`
    - 클라이언트로부터 전달받은 msg를 구독자(Subscriber)들에게 전달하기 전에, 메시지 가공이 필요한 경우의 prefix를 설정한다.
    - “메시지 가공 및 처리를 하기 위해, `SEND` 프레임의 `destination` 헤더에 `/app` 을 넣어 msg를 전달하는 방법” 에 해당된다.
    - 위 예시 코드의 경우, `/app` 이 `destination` 헤더의 prefix로 사용되면 msg를 가공하는 핸들러로 msg가 전달된다.
    - 일반 WebSocket처럼 일일이 핸들러를 등록하지 않아도 된다.
        - 일반 컨트롤러처럼 핸들러가 매핑되므로

<br/>

- `addEndpoint("/stomp-websocket")`
    - HTTP HandShaking을 할 때, 사용될 주소를 지정한다.
    - 즉, 웹소켓을 **연결(Connection)** 할 주소를 지정한다.

<br/>

### 메시지 가공 핸들러: `MsgProcessController`

```java
@RestController
public class MsgProcessController {

	@MessageMapping("/hello")
	@SendTo("/topic/roomId")
	public MessageResponseDto process(MessageRequestDto request) throws Exception {
		return new MessageResponseDto(
			"Hello, " + request.getName() + "!"
		);
	}

}
```

- `@MessageMapping("/hello")`
    - 만약 클라이언트에서 “../hello” 라는 api로 메시지를 보내면, `process()` 메서드가 호출된다.
        
        > `WebSocketBrokerConfig` 에서 설정했던 `/app` 과 합쳐져, `/app/hello` 를 `destination` 헤더 값으로 갖는 메시지들이 `process()` 메서드에 전달되게 된다.
        > 
    - 그리고 `process()` 메서드의 파라미터인 `MessageRequestDto` 에 전송된 msg가 바인딩된다.

<br/>

- `@SendTo("/topic/roomId")`
    - “/topic/roomId” 라는 Topic을 구독하고 있는 클라이언트(Subscriber)들에게 ‘`process()` 메서드에서 return하는 값’을 msg로서 브로드캐스팅한다.
    - “/topic/...” 에 msg를 전달하는 것이니, 리턴값이 바로 Simple Broker로 전달된다.