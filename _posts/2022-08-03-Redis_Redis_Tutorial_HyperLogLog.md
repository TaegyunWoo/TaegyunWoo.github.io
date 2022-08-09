---
category: Redis
tags: [Redis]
title: "HyperLogLog 알고리즘과 Redis 에서의 데이터 구조"
date:   2022-08-03 10:30:00 
lastmod : 2022-08-03 10:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# HyperLogLog 알고리즘과 Redis 에서의 데이터 구조

## 개요

Redis에서 제공하는 HyperLogLog 데이터 구조에 대해 알아보자.

### HyperLogLog 란?

- 원래 HyperLogLog는 매우 적은 메모리로 `집합의 원소 개수`를 추정할 수 있는 알고리즘이다.
- 집합의 원소 개수를 정확하게 계산하거나 원소의 개수가 아주 많을 때, 정확하지 않지만 최대한 정확한 값을 상대적으로 적은 메모리만 사용해 얻고 싶을 때 사용할 수 있다.
- **Redis 에서 말하는 HyperLogLog 는 알고리즘이 아닌, 데이터 구조이다.**
    - HyperLogLog 알고리즘으로 집합의 원소 개수를 근사하게 알아낼 수 있는 데이터 구조이다.
    - HyperLogLog는 키당 약 12kbyte의 매우 작은 양의 메모리를 사용하며, 0.81% 정도의 아주 작은 오차를 갖는다.
- HyperLogLog 알고리즘을 더 깊게 알아보고 싶다면, 아래 글을 추천한다.
    - [[Naver D2] 확률적 자료구조를 이용한 추정 - 유일한 원소 개수(Cardinality) 추정과 HyperLogLog](https://d2.naver.com/helloworld/711301)

<br/>

## 예시

```text
redis 127.0.0.1:6379> PFADD tutorials "redis"
1) (integer) 1
redis 127.0.0.1:6379> PFADD tutorials "mongodb"
1) (integer) 1
redis 127.0.0.1:6379> PFADD tutorials "mysql"
1) (integer) 1
redis 127.0.0.1:6379> PFCOUNT tutorials
(integer) 3
```

- `PFADD tutorials "redis"`
    - `tutorials` 라는 HyperLogLog 에 `redis` 를 저장한다.
- `PFCOUNT tutorials`
    - `tutorials` 라는 HyperLogLog 에 저장된 원소의 개수를 반환한다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 | 시간복잡도 |
| --- | --- | --- | --- |
| 1 | PFADD key value1 value2 … | 해당 key에 value들을 저장한다. | O(1) |
| 2 | PFCOUNT key1 key2 … | 해당 key들의 원소 개수를 반환한다. | O(N) <br/> N: 원소 개수를 조회할 key 개수 |
| 3 | PFMERGE destination source1 source2 … | destination에 source들을 합친다. | O(N) <br/> N: 합칠 key 개수 |