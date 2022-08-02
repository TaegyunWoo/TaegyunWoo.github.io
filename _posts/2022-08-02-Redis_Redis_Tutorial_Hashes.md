---
category: Redis
tags: [Redis]
title: "Redis Hashes 타입과 명령어"
date:   2022-08-02 15:50:00 
lastmod : 2022-08-01 15:50:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Hashes 타입과 명령어

## 개요

Redis의 데이터 타입 중 하나인 Hash에 대해 알아보자.

<br/>

## Hashes 데이터 타입

- Hash 구조는 아래와 같은 구조를 갖는다.
    - `<key, <field, value>>`
- Hash 구조를 통해, 객체(object)를 편리하게 표현할 수 있다. 아래는 그 예시이다.
    
    ```text
    key: 사람1
      이름(field): 홍길동
      성별(field): 남자
      나이(field): 18
    ```
    
<br/>

## 예시

```text
redis 127.0.0.1:6379> HMSET tutorialspoint name "redis tutorial"
description "redis basic commands for caching" likes 20 visitors 23000
OK
redis 127.0.0.1:6379> HGETALL tutorialspoint
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```

- `HMSET tutorialspoint name "redis tutorial"`
    - `tutorialspoint` 라는 key로, `<name, redis tutorial>` 이라는 key-value pair를 저장한다.
- `HGETALL tutorialspoint`
    - `tutorialspoint` 라는 key의 모든 key-value pair를 반환한다.

<br/>

## **주요 명령어 Reference**

| NO. | 명령어 | 설명 |
| --- | --- | --- |
| 1 | HDEL key field1 field2 … | 해당 key에 저장된 하나 이상의 field들을 제거 |
| 2 | HEXISTS key field | 해당 key에 해당 field가 존재하는지 확인 <br/> 1: 존재함, 0: 존재안함 |
| 3 | HGET key field | 해당 key의 해당 field 값을 반환 |
| 4 | HGETALL key | 해당 key의 모든 field-value 를 반환 |
| 5 | HINCRBY key field incremen` | 해당 key의 해당 field의 정수형 값을 increment 만큼 증가 |
| 6 | HINCRBYFLOAT key field increment | 해당 key의 해당 field의 실수형 값을 increment 만큼 증가 |
| 7 | HKEYS key | 해당 key의 모든 fields 반환 |
| 8 | HLEN key | 해당 key의 모든 fields 개수 반환 |
| 9 | HMGET key field1 field2 ... | 해당 key의 한 개 이상의 field 값 반환 |
| 10 | HMSET key field1 value1 field2 value2 ... | 해당 key에 한 개 이상의 field-value 저장 |
| 11 | HSET key field value | 해당 key에 field-value 저장 |
| 12 | HSETNX key field value | 해당 key에 동일한 field가 없다면 field-value 저장 |
| 13 | HVALS key | 해당 key의 모든 value 반환 |
| 14 | HSCAN key cursor [MATCH pattern] [COUNT count] | 해당 key의 몇 개의 field-value 반환 |