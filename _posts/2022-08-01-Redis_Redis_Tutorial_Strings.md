---
category: Redis
tags: [Redis]
title: "Redis Key 관리 명령어"
date:   2022-08-01 10:50:00 
lastmod : 2022-08-01 10:50:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Strings 타입과 명령어

## 개요

Redis 의 가장 기본적인 데이터 타입 Strings 에 대해 알아보자.

<br/>

## Strings 데이터 타입

- Redis 는 key-value 형태로 데이터를 저장한다.
- 여기서 Strings 타입은 value 부분의 타입을 의미한다.
- 즉 value 에 저장될 데이터의 형태가 String 이라는 것이다.

> Strings 타입뿐만 아니라, 다른 모든 타입들도 모두 value 부분을 의미한다.

<br/>

## 예시

### 문법

```text
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

### 예시

```text
redis 127.0.0.1:6379> SET tutorialspoint redis
OK
redis 127.0.0.1:6379> GET tutorialspoint
"redis"
```

- `SET tutorialspoint redis`
    - key: `tutorialspoint`
    - value: `redis`
    - 위와 같은 형태로 데이터를 저장한다.
- `GET tutorialspoint`
    - `tutorialspoint` 라는 키를 갖는 데이터의 value를 반환한다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 |
| --- | --- | --- |
| 1 | SET key value | 데이터(key-value)를 저장한다. |
| 2 | GET key | 해당 key의 value를 반환한다. |
| 3 | GETRANGE key start end | 해당 key의 value의 substring을 반환한다. <br/> (start~end 까지의 string 반환) |
| 4 | GETSET key value | 해당 key에 저장된 기존의 value를 반환하고, 새 value를 저장한다. |
| 5 | GETBIT key offset | 해당 key에 저장된 문자열의 offset(index)에 해당하는 문자의 비트값을 반환한다. |
| 6 | MGET key1 key2 ... | 여러 key의 value를 순서대로 반환한다. |
| 7 | SETBIT key offset value | 해당 key에 저장된 문자열의 offset(index)에 해당하는 문자의 비트값을 설정한다. |
| 8 | SETEX key seconds value | key-value를 저장하고, 유효시간을 초단위로 설정한다. |
| 9 | SETNX key value | 해당 key에 기존의 value가 없다면, 저장한다. |
| 10 | SETRANGE key offset value | 해당 key의 기존 value에서 offset에 해당하는 부분을 새 value로 덮어쓴다. |
| 11 | STRLEN key | 해당 key의 value의 길이를 반환한다. |
| 12 | MSET key1 value1 key2 value2 ... | 여러 key-value를 저장한다. |
| 13 | MSETNX key1 value1 key2 value2 | 여러 key-value를 key가 존재하지 않는 경우에만 저장한다. |
| 14 | PSETEX key milliseconds value | key-value를 저장하고, 유효시간을 밀리초 단위로 설정한다. |
| 15 | INCR key | 해당 key에 저장된 정수형 value의 값을 1 증가시킨다. |
| 16 | INCRBY key increment | 해당 key에 저장된 정수형 value의 값을 increment 만큼 증가시킨다. |
| 17 | INCRBYFLOAT key increment | 해당 key에 저장된 실수형 value의 값을 increment 만큼 증가시킨다. |
| 18 | DECR key | 해당 key에 저장된 정수형 value의 값을 1 감소시킨다. |
| 19 | DECRBY key decrement | 해당 key에 저장된 정수형 value의 값을 increment 만큼 감소시킨다. |
| 20 | APPEND key value | 해당 key의 기존 value 뒤에 새 문자열을 이어붙인다. |