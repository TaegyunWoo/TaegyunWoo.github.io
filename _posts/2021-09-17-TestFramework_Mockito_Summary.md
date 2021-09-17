---
category: Test-Framework
tags: [Mockito, 개요]
title: "[Mockito] Mockito 개요"
date:   2021-09-17 17:00:00 
lastmod : 2021-09-17 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- **[예시 프로젝트 다운로드](https://github.com/TaegyunWoo/Spring-Test-Code-Example)**

<br/><br/>

# 개요

## Mockito 소개

### Mock이란?

- 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체이다.

<br/>

### Mockito란?

- Java 용 오픈 소스 테스트 프레임워크이다.
- 이 프레임워크를 사용하면 TDD(테스트 주도 개발)을 위해, 자동화된 단위 테스트에서 테스트 이중 오브젝트를 작성할 수 있다.

> 출처: [위키백과](https://en.wikipedia.org/wiki/Mockito)

<br/><br/>

## Mockito 시작하기

### 스프링에 Mockito 의존성 추가

```xml
<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-core</artifactId>
	<version>3.1.0</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-junit-jupiter</artifactId>
	<version>3.1.0</version>
	<scope>test</scope>
</dependency>
```

<br/>

### 스프링부트에 Mockito 의존성 추가

- 스프링부트 2.2버전 이상으로 프로젝트 생성시, 의존성이 자동으로 추가된다.
- `spring-boot-starter-test` 에서 자동으로 추가해준다.

<br/>

### Mockito 주요 학습 Point

- Mock을 만드는 방법
- Mock이 어떻게 동작해야 하는지 관리하는 방법
- Mock의 행동을 검증하는 방법

<br>

---

<br>

<a href="https://inf.run/htNB"><img src="/assets/img/Inflearn_Java_Test/logo.png" width="400px" height="300px"></a>

- *본 게시글은 백기선님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*