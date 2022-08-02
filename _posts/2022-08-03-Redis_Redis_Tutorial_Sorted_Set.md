---
category: Redis
tags: [Redis]
title: "Redis Sorted-Set 타입과 명령어"
date:   2022-08-03 01:00:00 
lastmod : 2022-08-03 01:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Sorted-Set 타입과 명령어

## 개요

Redis의 데이터 타입 중 하나인 Sorted-Set에 대해 알아보자.

## Sorted-Set 데이터 타입

- Sorted-Set 은 우선 순위대로 정렬되는 Set 자료구조이다.
- Sorted-Set 에 저장되는 각 아이템들은 score를 가지고 있으며, score가 낮은 순서대로 정렬된다.
- Sorted-Set 의 구조는 아래와 같다.
    - `key: {(score, value1), (score, value2), ...}`

## 예시

```text
redis 127.0.0.1:6379> ZADD tutorials 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD tutorials 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD tutorials 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD tutorials 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD tutorials 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE tutorials 0 10 WITHSCORES
1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```

- `ZADD tutorials 1 redis`
    - `tutorials` 라는 key를 갖는 Sorted-Set 에 `redis` 라는 value를 score `1` 로 저장한다.
- Set 과 마찬가지로 중복되는 value 는 저장되지 않는다.
    - score 가 다르더라도, 저장되지 않는다.
- `ZRANGE tutorials 0 10 WITHSCORES`
    - `tutorials` 에 저장된 아이템을 score가 낮은 순서대로 0번째부터 10번째까지 score 값과 함께 조회한다.

## **주요 명령어 Reference**

| NO. | 명령어 | 설명 | 시간복잡도 | 비고 |
| --- | --- | --- | --- | --- |
| 1 | ZADD key score1 member1 score2 member2 … | 해당 key에 여러 member들을 score와 함께 저장한다. | 추가할 아이템마다 O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 2 | ZCARD key | 해당 key에 저장된 member 갯수를 반환한다. | O(1) |  |
| 3 | ZCOUNT key min max | 해당 key에 저장된 member 중 score 가 (min ~ max) 에 해당되는 member의 갯수를 반환한다. | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 4 | ZINCRBY key increment member | 해당 key에 저장된 member 의 score값을 increment 만큼 증가시킨다. | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 5 | ZINTERSTORE destination numkeys key1 key2 … | key들의 교집합을 destination 에 저장한다. <br/> numkeys 에는 교집합 연산을 할 Sorted-Set의 갯수를 입력해야 한다. | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 6 | ZLEXCOUNT key min max | 모든 member 들의 score가 같은 경우, 사전순 정렬된 결과를 반환한다. 반환 범위는 min ~ max 이다. | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 7 | ZRANGE key start stop [WITHSCORES] | (start ~ stop) 까지의 member를 score 오름차순으로 반환한다. 같은 score라면 사전 오름차순으로 반환한다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 반환될 아이템 갯수 | 기타 옵션이 많다. |
| 8 | ZRANGEBYLEX key min max [LIMIT offset count] | 모든 member 들의 score가 같은 경우, 사전순 정렬된 결과를 반환한다. 반환 범위는 min ~ max 이다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 반환될 아이템 갯수 | deprecated (비권장), ZRANGE 사용 |
| 9 | ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] | score 오름차순으로 member 들을 반환한다. 반환 범위는 min ~ max 이다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 반환될 아이템 갯수 | deprecated (비권장), ZRANGE 사용 |
| 10 | ZRANK key member | 해당 member의 rank를 반환한다. 가장 낮은 score를 갖는다면 rank는 0이다. | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 |  |
| 11 | ZREM key member1 member2 … | 해당 member 들을 삭제한다. | O(M*log(N)) <br/> N: Sorted-Set의 아이템 갯수 M: 제거될 아이템 갯수 |  |
| 12 | ZREMRANGEBYLEX key min max | 모든 member 들의 score가 같은 경우, 사전순 정렬하여 삭제한다. 삭제 범위는 min ~ max 이다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 삭제될 아이템 갯수 |  |
| 13 | ZREMRANGEBYRANK key start stop | rank 순으로 정렬하여 삭제한다. 삭제 범위는 min ~ max 이다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 삭제될 아이템 갯수 |  |
| 14 | ZREMRANGEBYSCORE key min max | score 순으로 정렬하여 삭제한다. 삭제 범위는 min ~ max 이다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 삭제될 아이템 갯수 |  |
| 15 | ZREVRANGE key start stop [WITHSCORES] | (start ~ stop) 까지의 member를 score 내림차순으로 반환한다. 같은 score라면 사전 내림차순으로 반환한다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 반환될 아이템 갯수 | deprecated (비권장), ZRANGE 사용 |
| 16 | ZREVRANGEBYSCORE key max min [WITHSCORES] | (start ~ stop) 까지의 member를 score 내림차순으로 반환한다. 같은 score라면 사전 내림차순으로 반환한다. | O(log(N)+M) <br/> N: Sorted-Set의 아이템 갯수 <br/> M: 반환될 아이템 갯수 | deprecated (비권장), ZRANGE 사용 |
| 17 | ZREVRANK key member | 해당 member의 rank를 거꾸로 계산하여 반환한다.  | O(log(N)) <br/> N: Sorted-Set의 아이템 갯수 | deprecated (비권장), ZRANGE 사용 |
| 18 | ZSCORE key member | 해당 member의 score를 반환한다. | O(1) |  |
| 19 | ZSCAN key cursor [MATCH pattern] [COUNT count] | 해당 key의 몇 개 member 반환 | O(1) <br/> 단, 모든 아이템을 조회한 경우 총 O(N) |  |