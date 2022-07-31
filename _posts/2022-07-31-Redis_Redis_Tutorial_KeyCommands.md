---
category: Redis
tags: [Redis]
title: "Redis Key 관리 명령어"
date:   2022-07-31 18:00:00 
lastmod : 2022-07-31 18:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis Key 관리 명령어

## 개요

- Redis 는 key-value 구조로 데이터를 저장한다.
- 이번 포스팅에서는 key 를 관리하는 방법에 대해 알아보자.

<br/>

## 예시

### 문법

```
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

### 예시

```
redis 127.0.0.1:6379> SET tutorialspoint redis
OK
redis 127.0.0.1:6379> DEL tutorialspoint
(integer) 1
```

- `SET tutorialspoint redis`
    - <key, value> 구조로 값을 저장하는 명령어이다.
    - <tutorialspoint, redis>
- `DEL tutorialspoint`
    - key 관리 명령어를 사용했다.
    - `tutorialspoint` 라는 key를 삭제한다는 명령어이다.
    - 물론 value도 삭제된다.

<br/>

## 주요 명령어 Reference

| NO. | 명령어 | 설명 |
| --- | --- | --- |
| 1 | DEL key | 해당 키가 존재하다면, 해당 키를 삭제한다. |
| 2 | DUMP key | 해당 키를 DUMP로 만든다. <br/> 그럼 Redis-specific format으로 값을 리턴하는데, 이 값을 RESTORE 명령어로 사용하면 그 key를 다시 복원할 수 있다. |
| 3 | EXISTS key | 해당 key가 존재하는지 확인한다. <br/> (1: 해당 key가 존재함, 0: 해당 key가 존재하지 않음) |
| 4 | EXPIRE key seconds | 해당 키의 유효 **“시간”** 을 설정한다. <br/> 설정한 시간(초)가 지나면, 자동으로 제거된다. |
| 5 | EXPIREAT key timestamp | 해당 키의 유효 **“시각”** 을 설정한다. <br/> 설정된 시각이 되면, 자동으로 제거된다. |
| 6 | PEXPIRE key milliseconds | 해당 키의 유효 시간을 ms단위로 설정한다. |
| 7 | PEXPIREAT key milliseconds-timestamp | 해당 키의 유효 시각을 ms단위로 설정한다. |
| 8 | KEYS pattern | 정규표현식으로 일치하는 키를 찾는다. |
| 9 | MOVE key db | 다른 db로 키를 옮긴다. |
| 10 | PERSIST key | 해당 key에 설정된 유효 시간을 제거한다. |
| 11 | PTTL key | 남은 유효 시간을 ms단위로 반환한다. |
| 12 | TTL key | 남은 유효 시간을 반환한다. |
| 13 | RANDOMKEY | 저장된 아무키를 반환한다. |
| 14 | RENAME key newkey | 키 이름을 재설정한다. |
| 15 | RENAMENX key newkey | 만약 newkey가 없다면, 키 이름을 재설정한다. |
| 16 | TYPE key | 해당 key의 데이터 타입을 반환한다. |