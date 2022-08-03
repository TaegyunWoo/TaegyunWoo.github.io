---
category: Redis
tags: [Redis]
title: "Redis 트랜잭션"
date:   2022-08-03 13:00:00 
lastmod : 2022-08-03 13:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Transaction

## 개요

Redis 에서 지원하는 Transaction 기능에 대해 알아보자.

## Redis Transaction 이란?

- Redis Transaction 은 일종의 명령 그룹을 한번에 실행시키는 것이다.
- Redis Transaction 의 특징은 아래와 같다.
    - 트랜잭션의 모든 명령은 하나의 독립된 작업으로 순차적으로 실행된다.
    - **트랜잭션이 실행되는 도중에는 다른 클라이언트의 요청을 처리할 수 없다.**
    - Atomic 하게 동작한다.
    - 트랜잭션 도중 에러가 발생해도 Rollback 되지 않고, 나머지 명령을 실행한다.

여기서 주목해야하는 것은 **Transaction 전체가 Atomic 하게 동작하여, 다른 클라이언트의 요청을 처리하지 않는다는 것** 이다.

따라서 실행할 명령어의 시간복잡도에 주의해서 사용해야 한다.

<br/>

## 예시

### 트랜잭션 정의

```
redis 127.0.0.1:6379> MULTI
OK
[실행할 명령어 입력]
redis 127.0.0.1:6379> EXEC
```

- `MULTI`
    - 해당 명령어로 트랜잭션을 정의하는 작업을 시작한다.
- `EXEC`
    - 트랜잭션 정의 작업을 종료하고, `MULTI` 이후에 입력한 모든 명령어들을 하나의 트랜잭션으로 실행한다.

### 예시

```text
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> SET tutorial redis
QUEUED
redis 127.0.0.1:6379> GET tutorial
QUEUED
redis 127.0.0.1:6379> INCR visitors
QUEUED
redis 127.0.0.1:6379> EXEC
1) OK
2) "redis"
3) (integer) 1
```

- `SET tutorial redis` , `GET tutorial` , `INCR visitors` 를 하나의 트랜잭션으로 묶어 실행한다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 | 시간복잡도 |
| --- | --- | --- | --- |
| 1 | DISCARD | MULTI 명령 이후에 입력한 모든 트랜잭션 명령어들을 취소한다. | O(N) <br/> N: 트랜잭션으로 queue한 명령 개수 |
| 2 | EXEC | MULTI 명령 이후에 입력한 모든 명령어들을 하나의 트랜잭션으로 실행한다. | 트랜잭션으로 정의한 명령어에 의존 |
| 3 | MULTI | 트랜잭션 정의 작업을 시작한다. | O(1) |
| 4 | UNWATCH | WATCH를 설정한 모든 key를 취소한다. | O(1) |
| 5 | WATCH key1 key2 … | MULTI 명령 이전에 실행한다. 해당 key들이 명령어 실행 시점 이후에 변경되었다면, 트랜잭션 대상에서 제외된다. | O(N) <br/> N: 설정할 key 개수 |