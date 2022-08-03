---
category: Redis
tags: [Redis]
title: "Redis Pub/Sub 과 명령어"
date:   2022-08-03 11:20:00 
lastmod : 2022-08-03 11:20:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Pub/Sub 과 명령어

## 개요

Redis에서 제공하는 Pub/Sub 기능에 대해 알아보자.

<br/>

## Redis Pub/Sub 란?

- Redis Pub/Sub는 Publisher가 메시지를 보내면, Subscriber가 메시지를 수신하는 Message System 이다.
- 메시지가 전송되는 Link 를 Channel 이라고 한다.

<br/>

## 예시

1. Terminal - Subscriber
    
    ```text
    redis 127.0.0.1:6379> SUBSCRIBE redisChat
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "redisChat"
    3) (integer) 1
    ```
    
    - `redisChat` 을 구독하여, 메시지가 들어올 때까지 대기한다.
2. Terminal - Publisher
    
    ```text
    redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
    (integer) 1
    ```
    
    - `redisChat` 으로 메시지를 보낸다.
3. Terminal - Subscriber
    
    ```text
    message
    redisChat
    Redis is a great caching technique
    ```
    
    - `redisChat` 으로부터 수신받은 메시지(`Redis is a great caching technique`)를 출력한다.

> `ctrl + c` 로 수신 상태를 종료할 수 있다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 | 시간복잡도 |
| --- | --- | --- | --- |
| 1 | PSUBSCRIBE pattern1 [pattern2 …] | 해당 패턴들을 구독한다. | O(N) <br/> N: 클라이언트가 이미 구독한 channel의 개수 |
| 2 | PUBSUB CHANNELS [pattern] | 현재 활성화된 channel 들을 반환한다. | O(N) <br/> N: 활성화된 channel의 개수 |
| 3 | PUBSUB NUMPAT | 클라이언트가 현재 구독하는 패턴의 개수를 반환한다. | O(1) |
| 4 | PUBSUB NUMSUB [channel1 [channel2 ...]] | 해당 channel의 구독자 수를 반환한다. 단, 패턴을 통해 구독하는 클라이언트는 제외한다. | O(N) <br/> N: 조회할 channel의 개수 |
| 5 | PUBLISH channel message | 해당 channel 에 message 를 보낸다. | O(N+M) <br/> N: 메시지가 전송될 채널의 구독자 수, M: 시스템 전체 패턴 구독자 수 |
| 6 | PUNSUBSCRIBE [pattern1 [pattern2 …]] | 해당 channel들을 구독 취소한다. | O(N+M) <br/> N: 구독 취소할 채널의 구독자 수, M: 시스템 전체 패턴 구독자 수 |
| 7 | SUBSCRIBE channel1 [channel2 …] | 해당 채널들을 구독한다. | O(N) <br/> N: 구독할 channel의 개수 |
| 8 | UNSUBSCRIBE channel1 [channel2 …] | 해당 채널들을 구독 취소한다. | O(N) <br/> N: 구독 취소할 channel의 구독자 수 |