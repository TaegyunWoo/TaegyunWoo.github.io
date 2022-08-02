---
category: Redis
tags: [Redis]
title: "Redis Set 타입과 명령어"
date:   2022-08-02 16:50:00 
lastmod : 2022-08-02 16:50:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Set 타입과 명령어

## 개요

Redis의 데이터 타입 중 하나인 Set에 대해 알아보자.

<br/>

## Set 데이터 타입

- Set 자료구조는 정렬되지 않은, 중복이 허용되지 않는 자료구조이다.
- Set과 관련된 모든 명령어는 O(1) 로 동작한다.
- 구조는 아래와 같다.
    - `key: {value1, value2, value3}`

<br/>

## 예시

```text
redis 127.0.0.1:6379> SADD tutorials redis
(integer) 1
redis 127.0.0.1:6379> SADD tutorials mongodb
(integer) 1
redis 127.0.0.1:6379> SADD tutorials mysql
(integer) 1
redis 127.0.0.1:6379> SADD tutorials mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS tutorials
1) "mysql"
2) "mongodb"
3) "redis"
```

- `SADD tutorials redis`
    - `tutorials` 라는 key에 `redis` 를 저장한다.
    - 이때 중복되는 value는 저장할 수 없다.
- `SMEMBERS tutorials`
    - `tutorials` 라는 key에 저장된 모든 아이템을 반환한다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 |
| --- | --- | --- |
| 1 | SADD key member1 member2 … | 해당 key에 하나 이상의 member를 추가한다. |
| 2 | SCARD key | 해당 key의 멤버 갯수를 반환한다. |
| 3 | SDIFF key1 key2 … | 해당 key들의 차집합(`key1 - key2 - …`) 결과를 반환한다. |
| 4 | SDIFFSTORE destination key1 key2 … | 해당 key들의 차집합 결과를 destination에 저장한다. |
| 5 | SINTER key1 key2 … | 해당 key들의 교집합(`key1 ∩ key2 ∩ …`) 결과를 반환한다. |
| 6 | SINTERSTORE destination key1 key2 … | 해당 key들의 교집합(`key1 ∩ key2 ∩ …`) 결과를 destination에 저장한다. |
| 7 | SISMEMBER key member | 해당 key에 해당 member가 있는지 확인한다. <br/> 1: 있음, 0: 없음 |
| 8 | SMEMBERS key | 해당 key의 모든 멤버를 반환한다. |
| 9 | SMOVE source destination member | source의 member를 destination에 옮긴다. |
| 10 | SPOP key | 해당 key의 무작위 멤버를 꺼낸다. |
| 11 | SRANDMEMBER key [count] | 해당 key의 무작위 멤버를 count 개만큼 반환한다. |
| 12 | SREM key member1 member2 … | 해당 key의 한 개 이상의 member를 제거한다. |
| 13 | SUNION key1 key2 … | 해당 key들의 합집합(`key1 ∪ key2 ∪ …`) 결과를 반환한다. |
| 14 | SUNIONSTORE destination key1 key2 … | 해당 key들의 합집합(`key1 ∪ key2 ∪ …`) 결과를 destination에 저장한다. |
| 15 | SSCAN key cursor [MATCH pattern] [COUNT count] | 해당 key의 몇 개의 멤버 반환 |