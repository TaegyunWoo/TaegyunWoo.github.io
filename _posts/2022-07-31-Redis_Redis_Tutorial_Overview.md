---
category: Redis
tags: [Redis]
title: "Redis 란 무엇인가"
date:   2022-07-31 17:10:00 
lastmod : 2022-07-31 17:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# Redis 란 무엇인가

Redis에 대해 시리즈로 포스팅을 할 예정이다. 이 포스팅를 통해, 시리즈의 첫번째로 Redis 가 무엇인지 알아보자.

<br/>

## Redis 개요

- Redis 는 오픈소스이다
- key-value 기반의 저장소이다.
- 확장 가능한 고성능 웹 애플리케이션을 만드는데 사용할 수 있는 apt(advanced packaging tool) 솔루션이다.

<br/>

## Redis의 특징

- Redis는 모든 데이터를 Memory 에 올려두고 있다. 백업 등의 영구저장을 할 때만 Disk를 사용한다.
- Redis는 다른 key-value 데이터 저장소들에 비해, 비교적 풍부한 데이터 유형 세트를 가지고 있다.
- Redis는 데이터를 원하는 수만큼의 secondary 인스턴스에 복제할 수 있다.

<br/>

## Redis의 장점

- **메모리를 사용하기 때문에 상당히 빠르다.**
    - 초당 약 110000 SET, 초당 약 81000 GET 연산을 수행할 수 있다.
- **다양한 자료구조를 지원한다.**
    - Redis는 잘 알려진 자료구조을 대부분 지원한다.
        - List, Set, Sorted-Set, Hash
    - 잘 알려진 자료구조를 사용하기 때문에, 문제 해결을 위한 자료구조 선택이 유용하다. 즉 접근성이 좋다.
- **모든 연산은 Atomic 하다.**
    - Atomic 하다는 것은 `한번에 하나의 명령만 처리할 수 있다` 는 것이다.
    - Redis는 싱글 쓰레드로 동작하기 때문에, 모든 작업을 하나씩 처리한다.
    - 따라서 명령어의 시간복잡도를 잘 고려해야 한다.
- **유용한 유틸리티 툴로 사용할 수 있다.**
    - Redis는 캐싱, Message Queue, Web Application Session 등의 경우에 사용할 수 있다.

<br/>

## `Redis` vs `다른 key-value DB`

- Redis는 수많은 key-value DB 중 하나이다. Redis의 value에는 복잡한 데이터 유형이 저장될 수 있다.
- Redis는 In-Memory DB이다. 따라서 전체 메모리보다 큰 데이터를 저장할 수 없지만, Read/Write 속도가 매우 빠르다.
- In-Memory DB의 또다른 장점으로, 데이터의 구조가 더욱 간단해진다. 동일한 자료구조를 Memory에 저장하는 것이 Disk에 저장하는 것보다 복잡도가 낮다.

다음 포스팅에서 Redis 의 각 데이터 타입에 대해 설명하도록 하겠다.