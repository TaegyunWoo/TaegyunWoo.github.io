---
category: Redis
tags: [Redis]
title: "Redis 파이프라이닝"
date:   2022-08-03 13:46:00 
lastmod : 2022-08-03 13:46:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Pipelining

## 개요

Redis에서 Pipelining을 하는 방법에 대해 알아보자.

<br/>

### Redis 가 요청에 대해 응답하는 방법

- Redis는 당연하게도 TCP 프로토콜을 이용한다. 따라서 요청/응답 모델을 사용한다.
- 아래는 Redis 에서 요청/응답 하는 과정이다.
    1. 클라이언트가 서버에 쿼리를 보내고, 서버는 일반적으로 Blocking 방식으로 소켓을 통해 읽는다.
    2. 서버가 명령을 처리하고 응답을 다시 클라이언트로 보낸다.

### Pipelining 이란?

- Pipelining 의 일반적인 의미는 아래와 같다.
    - 클라이언트가 응답을 전혀 기다리지 않고, 서버에 여러 요청을 보낸다.  
    그리고 서버는 모든 요청을 받은 후 한번에 요청을 읽는다.
- Redis 에서 Pipelining 을 사용하는 경우
    - 만약 클라이언트가 10,000 개의 `SET`·`GET` 명령을 서버에 요청한다고 해보자.
    - 10,000 개의 명령을 각각 따로 요청했을 때, 전체 요청 RTT(Round Trip Time)가 증가할 수 밖에 없다.
        - TCP 프로토콜을 사용하기 때문에, Handshake 과 같은 번거로운 작업을 총 10,000번 해야한다.
        - 이것은 상당히 비효율적이다.
    - 바로 이런 경우, Pipelining을 사용하면 효율적으로 문제를 해결할 수 있다.
- Redis 에서의 Pipelining
    - **클라이언트가 보내려는 요청을 모두 모아서, 단 한번의 요청으로 Redis 서버에 보내는 것을 말한다.**
    - **모든 명령들은 클라이언트 측에서 모아둔다.**

<br/>

## 예시

```text
$(echo -en "PING\r\n SET tutorial redis\r\nGET tutorial\r\nINCR
visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379
+PONG
+OK
redis
:1
:2
:3
```

- 터미널에서 위 명령어를 입력한다.
- `"PING\r\n SET tutorial redis\r\nGET tutorial\r\nINCR
visitor\r\nINCR visitor\r\nINCR visitor\r\n"`
    - 문자열 부분을 보면, 원하는 명령어들을 모두 한번에 작성한 것을 알 수 있다.
- `| nc localhost 6379`
    - 그리고 그 메시지를 한번에 전송한다.
    - 바로 이것을 pipelining 이라고 한다.

<br/>

## Pipelining 을 사용할 때의 장점

로컬 호스트에 대한 연결의 경우 약 5배 정도의 성능 향상을 볼 수 있고, 느린 인터넷 연결을 통한 경우 약 100배이다.

<br/>

## Transaction vs Pipelining

지금까지 Pipelining 에 대해 알아보았다.

이전에 설명한 Transaction 과 유사하다는 것을 알 수 있다.

그렇다면 Transaction 과 Pipelining 의 차이점은 무엇일까?

> 이전 포스팅: [“Redis 트랜잭션”](https://taegyunwoo.github.io/redis/Redis_Redis_Tutorial_Transaction)

<br/>

가장 큰 차이점은 아래와 같다.

- Pipelining 은 Atomic 하지 않다.
- Transaction 은 Atomic 하다.

아래 그림을 보자.

![Untitled](https://buildatscale.tech/content/images/size/w1600/2021/07/redis-pipeline-vs-transaction.png)
> 그림 출처: [https://buildatscale.tech/what-is-redis-pipeline/](https://buildatscale.tech/what-is-redis-pipeline/)

- Pipeline 의 경우, 서로 다른 Pipeline 들이 번갈아가면서 수행된다.
    - 쉽게 말해, `pipeline1` 의 첫번째 명령을 실행하고, `pipeline2` 의 첫번째 명령을 실행하는 식으로 동작한다.
- Transaction 의 경우, 하나의 Transaction 이 모두 끝나면 다음 Transaction이 실행된다.