---
category: Spring-MVC
tags: [스프링, MVC, HTTP메시지]
title: "[스프링 - MVC] HTTP 메시지 컨버터"
date:   2021-07-29 15:30:00 
lastmod : 2021-07-29 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# HTTP 메시지 컨버터

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우, HTTP 메시지 컨버터를 사용하면 편리하다.

<br><br>

## `@ResponseBody` 사용 원리

- `@ResponseBody` **사용 X**
    - 뷰 템플릿으로 응답해야 한다면 **뷰 리졸버를 통해 뷰를 호출** 하게 된다.
- `@ResponseBody` **사용 O**
    - 뷰 템플릿으로 응답하지 않고 단순히 HTTP 응답 메시지를 통해 응답한다면, **뷰 리졸버를 사용하지 않고, HTTP 메시지 컨버터를 통해 응답** 하게 된다.
    - 기본 문자 처리
        - `StringHttpMessageConverter`
    - 기본 객체 처리
        - `MappingJackson2HttpMessageConverter`

<br><br>

## HTTP 메시지 컨버터 적용의 경우

HTTP 메시지 컨버터는 아래 경우에서 적용된다.

### HTTP 요청시

- `@ReqeustBody`
- `HttpEntity`
- `RequestEntity`

### HTTP 응답시

- `@ResponseBody`
- `HttpEntity`
- `ResponseEntity`

<br><br>

## HTTP 메시지 컨버터 특징

- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- '대상 클래스 타입'과 '미디어 타입' 두 가지를 체크해서 사용여부를 결정한다.

<br><br>

## 주요 메시지 컨버터

![메시지 컨터버들](/assets/img/2021-07-29-SPRING_MVC_HTTPMessageConverter/Untitled%204.png)

스프링 부트가 제공하는 주요 메시지 컨버터의 종류는 위와 같다. 이때, 클래스타입과 미디어타입을 비교하여 어떤 메시지 컨버터를 사용할 것인지 결정된다.

<br>

### ByteArrayHttpMessageConverter

- `byte[]` 데이터를 처리한다.
- 클래스 타입: `byte[]` , 미디어 타입: `*/*`
- 요청 예) `@RequestBody byte[] data`
- 응답 예) `return byte[]`
    - 이때 미디어타입은 `application/octet-stream` 으로 설정된다.

<br>

### StringHttpMessageConverter

- `String` 문자로 데이터를 처리한다.
- 클래스 타입: `String` , 미디어 타입: `*/*`
- 요청 예) `@RequestBody String data`
- 응답 예) `return String`
    - 이때 미디어타입은 `text/plain` 으로 설정된다.

<br>

### MappingJackson2HttpMessageConverter

- `JSON` 형식으로 데이터를 처리한다.
- 클래스 타입: 객체 또는 HashMap , 미디어 타입: `application/json`
- 요청 예) `@RequestBody HelloData data`
- 응답 예) `return HelloData`
    - 이때 미디어타입은 `application/json` 으로 설정된다.

<br><br>

## 데이터 읽고 쓰기 원리

### HTTP 요청 데이터를 읽을 때의 원리

1. HTTP 요청이 오고, 컨트롤러에서 `@RequestBody` 나 `HttpEntity` 파라미터를 사용한다.
2. 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해, `canRead()` 를 호출한다.
    - 대상 클래스 타입을 지원하는가?
        - 예) `@RequestBody` 의 대상 클래스 ( `byte[]` , `String` , `HelloData` ) 체크
    - HTTP 요청의 미디어 타입(`content-type`)을 지원하는가?
        - 예) `text/plain` , `application/json` , `*/*` 체크
3. `canRead()` 조건을 만족하면 `read()` 를 호출해서 객체 생성하고, 반환한다.
4. 파라미터에 전달된다.

<br>

### HTTP 응답 데이터를 쓸 때의 원리

1. 컨트롤러에서 `@ResponseBody` 나 `HttpEntity` 로 값이 반환된다.
2. 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해, `canWrite()` 를 호출한다.
    - 대상 클래스 타입을 지원하는가?
        - 예) `return` 의 대상 클래스 ( `byte[]` , `String` , `HelloData` ) 체크
    - **HTTP 요청** 의 미디어 타입 (`Accept`)을 지원하는가?
        - 예) `text/plain` , `application/json` , `*/*` 체크
3. `canWrite()` 조건을 만족하면 `write()` 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*