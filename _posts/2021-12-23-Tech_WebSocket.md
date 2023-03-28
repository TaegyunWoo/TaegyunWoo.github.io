---
category: Tech
tags: [Tech]
title: "WebSocket과 Spring-Boot"
date:   2021-12-23 13:55:00 
lastmod : 2021-12-23 13:55:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# WebSocket

## 개요

### WebSocket 이란?

- 양방향 통신을 제공하기 위해 개발된 프로토콜이다.
- 기존의 단방향 HTTP 프로토콜과 호환된다.
- 일반 Socket 통신과 달리, HTTP 80 포트를 사용하므로 방화벽 제약이 없다.
- 접속까지는 HTTP 프로토콜을 이용하고, 통신은 자체적인 WebSocket 프로토콜을 사용한다.

<br/>

### 일반 HTTP 통신의 동작 방식

- **클라이언트의 요청이 있어야만, 서버가 응답 msg를 보낼 수 있다.**
- **응답 msg를 보낸 후, 클라이언트와의 연결이 종료된다.**
- 일반 HTTP는 반이중 통신이다.
    - 반이중 통신: 동시에 양쪽이 데이터를 보낼 수 없음

<br/>

![출처: 솔내시스템 ezTCP님의 블로그](/assets/img/2021-12-23-Tech_WebSocket/Untitled.png)

출처: 솔내시스템 ezTCP님의 블로그

<br/>

- 순수 HTTP만으로 WebSocket과 같이 동작시키는 방법들
    - **polling**
        - 클라이언트가 일반 HTTP Request 메시지를 서버로 계속 요청한다.
        - HTTP Connection을 맺고 끊음을 반복하므로 부담이 크다.
    - **Long polling**
        - 클라이언트가 일반 HTTP Request 메시지를 서버에 일단 요청한다.
        - 클라이언트가 그 상태로 계속 대기하다가, 서버에서 이벤트가 발생하면 그제서야 응답한다.
        - 그 후, 클라이언트가 다시 Request 메시지를 보낸다.
        - 이벤트 간의 시간간격이 좁으면, polling과 차이점이 없어진다. 즉, 부담이 커진다.
    - **Streaming**
        - 클라이언트가 일반 HTTP Request 메시지를 서버에 일단 요청한다.
        - 클라이언트가 그 상태로 계속 대기하다가, 서버에서 이벤트가 발생하면 그제서야 응답한다.
        - **이때 메시지만 보내고 (Flush) 연결을 끊지는 않는다.**
        - Connection 과정이 최초 한 번만 필요하므로, 부담이 경감된다.

<br/>

### WebSocket의 동작 방식

![출처: 위키백과](/assets/img/2021-12-23-Tech_WebSocket/Untitled%201.png)

출처: 위키백과

- 최초 Connection을 맺을 때, **HTTP Handshake**를 진행한다.
- 그 후부터는, Websocket 프로토콜을 사용한다.
- **클라이언트가 접속 요청을 하고, 웹 서버가 응답한 후 Connection을 유지한다.**
- **클라이언트의 요청 없이도 데이터를 전송할 수 있는 프로토콜이다.**
- HTTP 환경에서 전이중 통신을 지원한다.

> **ws://~ 로 WebSocket 요청 MSG를 보낸다.**
> 

<br/>

### WebSocket 접속 과정

![출처: [https://blog.naver.com/eztcpcom/220070508655](https://blog.naver.com/eztcpcom/220070508655)](/assets/img/2021-12-23-Tech_WebSocket/Untitled%202.png)

출처: [https://blog.naver.com/eztcpcom/220070508655](https://blog.naver.com/eztcpcom/220070508655)

- HTTP와 마찬가지로 WebSocket 역시 TCP/IP 위에서 동작하므로 가장 먼저 TCP/IP 접속을 해야한다.
- 그 후, HTTP Handshake를 진행한다.

> 참고로 소켓통신에서 클라이언트와 서버는 N:1 관계로 연결된다.
> 

<br/><br/>

## WebSocket 구현 (With Spring-boot)

### 의존성 추가: `build.gradle`

```xml
implementation 'org.springframework.boot:spring-boot-starter-websocket'
```

<br/>

### 웹소켓 핸들러: `WebSocketHandler`

```java
@Slf4j
@Component
public class WebSocketHandler extends TextWebSocketHandler {

	//웹소켓에 접속한 클라이언트 정보를 저장할 컬렉션
	private static List<WebSocketSession> sessionList = new ArrayList<>();

	//클라이언트가 msg 전송 시, 호출되는 메서드
  @Override
  protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    String payload = message.getPayload(); //전달 받은 msg의 페이로드
    log.info("payload : " + payload);

		//요청받은 msg의 페이로드를 웹소켓에 접속된 모든 클라이언트에게 전송한다.
    for(WebSocketSession sess: sessionList) {
      sess.sendMessage(message);
    }
  }

  // 클라이언트가 웹소켓에 접속 시, 호출되는 메서드
  @Override
  public void afterConnectionEstablished(WebSocketSession session) throws Exception {
    list.add(session); //접속한 클라이언트 정보를 저장한다.
    log.info(session + " 클라이언트 접속");
  }

  //클라이언트가 웹소켓 접속 해지 시, 호출되는 메서드
  @Override
  public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
    log.info(session + " 클라이언트 접속 해제");
    list.remove(session); //해제한 클라이언트의 정보를 제거한다.
  }
}
```

- `extends TextWebSocketHandler`
    - 텍스트 기반 웹소켓 핸들러를 구현하려면, `TextWebSocketHandler` 를 상속받아서 구현해야 한다.
- `List<WebSocketSession> sessionList`
    - 웹소켓에 접속한 클라이언트 정보를 저장할 리스트이다.
    - `WebSocketSession`
        - 웹소켓에 접속한 사용자(클라이언트)의 정보가 담겨있는 객체이다.

<br/>

### 웹소켓핸들러 등록: `WebSocketConfig`

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	//생성자 주입
  private final WebSocketHandler webSocketHandler;

  @Override
  public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(webSocketHandler, "/ws").setAllowedOrigins("*");
  }
}
```

- `addHandler(webSocketHandler, "/ws")`
    - 웹소켓에 접속할 엔드포인트를 설정한다.
    - 위 경우, 아래 주소로 요청하여 웹소켓에 접속(Connection)할 수 있다.
        
        ```xml
        ws//localhost:8080/ws
        ```
        
- `setAllowedOrigins("*")`
    - CORS 를 허용한다.