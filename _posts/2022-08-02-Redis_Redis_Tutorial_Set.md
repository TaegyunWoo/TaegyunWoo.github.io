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
- Redis에서 Set 에 저장, 제거, 멤버 존재 여부 확인은 모두 O(1) 시간이 소요된다.
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

| NO. | 명령어 | 설명 | 시간복잡도 |
| --- | --- | --- | --- |
| 1 | SADD key member1 member2 … | 해당 key에 하나 이상의 member를 추가한다. | O(N) <br/> N: 저장할 member 갯수 |
| 2 | SCARD key | 해당 key의 멤버 갯수를 반환한다. | O(1) |
| 3 | SDIFF key1 key2 … | 해당 key들의 차집합(key1 - key2 - …) 결과를 반환한다. | O(N) <br/> N: 계산될 모든 Set의 아이템 갯수 |
| 4 | SDIFFSTORE destination key1 key2 … | 해당 key들의 차집합 결과를 destination에 저장한다. | O(N) <br/> N: 계산될 모든 Set의 아이템 갯수 |
| 5 | SINTER key1 key2 … | 해당 key들의 교집합(key1 ∩ key2 ∩ …) 결과를 반환한다. | O(N*M) <br/> N: 가장 작은 Set의 아이템 갯수 <br/> M: Set의 갯수 |
| 6 | SINTERSTORE destination key1 key2 … | 해당 key들의 교집합(key1 ∩ key2 ∩ …) 결과를 destination에 저장한다. | O(N*M) <br/> N: 가장 작은 Set의 아이템 갯수 <br/> M: Set의 갯수 |
| 7 | SISMEMBER key member | 해당 key에 해당 member가 있는지 확인한다. <br/> 1: 있음, 0: 없음 | O(1) |
| 8 | SMEMBERS key | 해당 key의 모든 멤버를 반환한다. | O(N) <br/> N: Set의 아이템 갯수 |
| 9 | SMOVE source destination member | source의 member를 destination에 옮긴다. | O(1) |
| 10 | SPOP key [count] | 해당 key의 무작위 멤버를 count 개 만큼 꺼낸다. | O(N) <br/> N: count |
| 11 | SRANDMEMBER key [count] | 해당 key의 무작위 멤버를 count 개만큼 반환한다. | O(N) <br/> N: count |
| 12 | SREM key member1 member2 … | 해당 key의 한 개 이상의 member를 제거한다. | O(N) <br/> N: 삭제할 아이템 갯수 |
| 13 | SUNION key1 key2 … | 해당 key들의 합집합(key1 ∪ key2 ∪ …) 결과를 반환한다. | O(N) <br/> N: 모든 Set의 아이템 갯수 |
| 14 | SUNIONSTORE destination key1 key2 … | 해당 key들의 합집합(key1 ∪ key2 ∪ …) 결과를 destination에 저장한다. | O(N) <br/> N: 모든 Set의 아이템 갯수 |
| 15 | SSCAN key cursor [MATCH pattern] [COUNT count] | 해당 key의 몇 개의 멤버 반환 | O(1) <br/> 단, 모든 아이템을 조회한 경우 총 O(N) |