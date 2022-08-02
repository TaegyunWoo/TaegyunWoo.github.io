---
category: Redis
tags: [Redis]
title: "Redis Hashes 타입과 명령어"
date:   2022-08-02 16:30:00 
lastmod : 2022-08-02 16:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis List 타입과 명령어

## 개요

Redis의 데이터 타입 중 하나인 List에 대해 알아보자.

<br/>

## List 데이터 타입

- List 는 말그대로 문자열의 리스트형 구조이다.
- 삽입한 순서대로 정렬되며, 리스트 앞·뒤에 아이템을 추가할 수 있다.
- 구조
    - `key: [value1, value2, value3, ...]`

<br/>

## 예시

```text
redis 127.0.0.1:6379> LPUSH tutorials redis
(integer) 1
redis 127.0.0.1:6379> LPUSH tutorials mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH tutorials mysql
(integer) 3
redis 127.0.0.1:6379> LRANGE tutorials 0 10
1) "mysql"
2) "mongodb"
3) "redis"
```

- `LPUSH tutorials redis`
    - tutorials 라는 리스트(key) 앞에 redis를 추가한다.
- `LRANGE tutorials 0 10`
    - 0번째부터 10번째까지 key-value pair 를 반환한다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 |
| --- | --- | --- |
| 1 | BLPOP key1 key2 ... timeout | 해당 key의 앞쪽 아이템을 반환한다. <br/> 만약 꺼낼 수 있는 아이템이 없다면 설정한 timeout(초) 만큼 기다리면서 꺼낼 수 있는 상황일 때 꺼내온다. |
| 2 | BRPOP key1 key2 ... timeout | 해당 key의 뒷쪽 아이템을 반환한다. <br/> 만약 꺼낼 수 있는 아이템이 없다면 설정한 timeout(초) 만큼 기다리면서 꺼낼 수 있는 상황일 때 꺼내온다. |
| 3 | BRPOPLPUSH source destination timeout | source의 뒷쪽 아이템을 꺼내서 destination의 앞쪽에 추가한다. <br/> 만약 꺼낼 수 있는 아이템이 없다면 설정한 timeout(초) 만큼 기다리면서 꺼낼 수 있는 상황일 때 꺼낸다. |
| 4 | LINDEX key index | 해당 index의 아이템을 꺼낸다. |
| 5 | LINSERT key BEFORE|AFTER pivot value | 해당 key에 저장된 pivot(value)의 앞|뒤에 value를 삽입한다. |
| 6 | LLEN key | 해당 key의 길이를 반환한다. |
| 7 | LPOP key | 해당 key의 앞쪽 아이템을 꺼낸다. |
| 8 | LPUSH key value1 value2 ... | 해당 key의 앞쪽에 하나 이상의 value를 추가한다. |
| 9 | LPUSHX key value | 만약 해당 key 리스트가 존재한다면 value를 앞쪽에 추가한다. |
| 10 | LRANGE key start stop | (start ~ stop)까지의 아이템을 반환한다. |
| 11 | LREM key count value | 해당 key의 value를 앞에서부터 count 개 만큼 제거한다. |
| 12 | LSET key index value | 해당 index를 갖는 아이템을 value로 교체한다. |
| 13 | LTRIM key start stop | 해당 리스트를 (start ~ stop)까지로 자른다. |
| 14 | RPOP key | 해당 key의 뒷쪽 아이템을 꺼낸다. |
| 15 | RPOPLPUSH source destination | source의 뒷쪽 아이템을 꺼내서 destination의 앞쪽에 추가한다. |
| 16 | RPUSH key value1 value2 ... | 해당 key의 뒷쪽에 하나 이상의 value를 추가한다. |
| 17 | RPUSHX key value | 만약 해당 key 리스트가 존재한다면 value를 뒷쪽에 추가한다. |