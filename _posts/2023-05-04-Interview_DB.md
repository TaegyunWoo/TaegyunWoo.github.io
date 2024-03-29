---
category: Interview
tags: [코딩인터뷰 완전분석]
title: "[코딩인터뷰 완전분석] 면접 대비 정리 - 데이터베이스"
date:   2023-05-04 17:00:00 +0900
lastmod : 2023-05-04 17:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

> 이 글은 게일 라크만 맥도웰 저자의 ‘코딩인터뷰 완전분석’을 읽으며, 깨달은 사실과 내용을 정리하는 글입니다.  
자세한 내용이 궁금하다면 [http://www.yes24.com/Product/Goods/44305533](http://www.yes24.com/Product/Goods/44305533) 에서 책을 구매하시면 좋겠습니다.
> 

면접에서 DB와 관련된 질문을 받을 수 있습니다.

이번 포스팅에서는 데이터베이스 관련 문제를 받았을 때, 어떻게 접근하고 해결해야 하는지 정리해보겠습니다.

*참고로 ‘NoSQL’, ‘Transaction’, ‘Index’ 등 자세한 내용은 제외되어 있고, DB 설계와 관련된 내용을 다룹니다! 관련 내용을 추가로 학습하시기 바랍니다.*

<br/><br/>

## 정규화 vs 비정규화

정규화는 **데이터 중복을 최소화하도록 설계하는 것**을 말합니다.

비정규화는 **조회 시간을 최적화하도록 설계하는 것**을 의미합니다.

<br/>

### 정규화·비정규화 예시

하나 예시를 들어볼까요?

한 DB에 `COURSES` 와 `TEACHERS` 라는 테이블이 존재한다고 가정해보겠습니다.

- 정규화 설계시
    - `TEACHERS` 테이블에 대한 외래키 (`TEACHER_ID`)를 갖는 칼럼이 `COURSES` 테이블에 존재
    - 장점 : **교사 정보를 DB에 한번만 저장해도 된다. (`TEACHER` 테이블만 관리하면 되기 때문에)**
    - 단점 : **교사 정보와 수업 정보를 함께 조회할 때, `JOIN` 연산이 필요하기 때문에 비교적 처리속도 ↓**
- 비정규화 설계시
    - `TEACHERS` 와 `COURSES` 테이블 모두 교사 이름 칼럼이 존재 (즉, 중복 데이터 존재)
    - 장점 : **`JOIN` 연산을 최소화할 수 있어, 비교적 처리속도 ↑**
    - 단점 : **두 테이블에 중복되는 데이터가 존재하기 때문에, `TEACHERS` 와 `COURSES` 테이블을 함께 관리해야함**

위 예시를 통해, 정규화와 비정규화의 차이점과 각각의 이점에 대해 알아볼 수 있습니다.

<br/>

### 정규화·비정규화의 특징과 차이점 정리

- 정규화
  - 주요 장점
    - 데이터 무결성 유지
  - 주요 단점
    - `JOIN` 연산으로 인한 조회 성능 저하
- 비정규화
  - 주요 장점
    - `JOIN` 연산 최소화로 조회 성능 향상
  - 주요 단점
    - 데이터 갱신 비용 증가 (여러 테이블에 존재하는 중복 데이터를 함께 수정해야 하므로)
    - 무결성 저해

### 비정규화를 고려해야 하는 상황

- 서비스 특성상, **읽기 성능이 매우 중요한 경우**
  - 데이터 무결성을 지키기 위해 더 많은 노력을 해야하기 때문에, 읽기 성능이 매우 중요한 경우에 사용하는 것을 권장
- 서비스 특성상, **데이터 수정 빈도가 낮은 경우**
  - 데이터 무결성을 해치는 업데이트 작업을 수행할 확률이 낮은 경우

위와 같은 상황에서, **Join 연산이 포함된 쿼리가 빈번하게 발생**한다면 **비정규화를 고려**할 수 있습니다.

또한 비정규화를 진행했다면, **데이터의 무결성을 지키기 위한 전략**까지 생각해두는 것이 필수적입니다.

> [데이터 무결성을 어떻게 지킬 수 있을까?]  
대표적인 방법인 트랜잭션을 통해서 무결성을 유지할 수 있습니다.  
트랜잭션을 통해서, 중복된 데이터가 서로 다른 값을 갖는 것을 방지하도록 처리할 수 있습니다.  
이런 트랜잭션만을 관리하는 DB 접근 계층을 하나 더 추가해서, 애플리케이션 계층이 DB 접근 계층을 통해, 간접적으로 DB에 접근하도록 처리하는 것도 하나의 방법이 될 수 있습니다.

<br/><br/>

## 소규모 데이터베이스 설계

면접장에서 DB를 직접 설계해보라는 요청을 받을 수 있습니다. 이번에는 이를 어떻게 대처해야 하는지 정리해보겠습니다.

*참고로 여기서 다루는 접근방법은 [이전에 포스팅한 ‘객체 지향 설계’에서 설명한 내용](https://taegyunwoo.github.io/interview/Interview_OOD)과 비슷한 점이 많습니다.*

이제 DB 설계 문제를 어떻게 해결해야하는지 각 단계별로 알아보겠습니다.

<br/>

### 1단계 : 문제 모호성 처리하기

DB 문제에서 분명히 모호한 부분이 내포되어 있을 것입니다. 따라서 **설계를 시작하기 전에 정확히 무엇을 설계해야 하는지 이해**해야 합니다. 그러니 **면접관에게 질문을 던져 이를 해결**합시다.

> [예시 - 아파트 임대 대행업체인 경우]  
문제 : 아파트 임대 대행업체가 사용할 시스템의 DB를 설계하라.  
모호한 부분 : 업체 대리점이 여러 개인지?, 한 사람이 같은 빌딩에 집(아파트)을 2개 이상 빌릴 수 있는지? 등…
> 

<br/>

### 2단계 : 핵심 객체 정의

이제 **해당 시스템의 핵심 객체가 무엇인지 고민**해봅니다.

그리고 보통 여기서 도출된 **핵심 객체를 하나의 테이블로 사용**하게 됩니다.

> [예시 - 아파트 임대 대행업체인 경우]  
핵심 객체 : `Building` , `Apartment` , `Manager` , `Tenant` 등
> 

<br/>

### 3단계 : 관계 분석

핵심 객체를 결정하고 테이블을 만들었으면, **테이블 간의 관계를 설정**합니다.

일대일, 일대다, 다대다 관계 중 적절한 것으로 지정합니다.

> [예시 - 아파트 임대 대행업체인 경우]  
<br/>
관계 : 한 빌딩에 여러 아파트가 존재할 수 있다. (일대다)  
답 : `Apartment` 테이블에 `Builder_ID` 외래키 칼럼을 추가한다.  
<br/>
관계 : 한 사람이 여러 아파트를 가질 수 있고, 여러 사람이 한 아파트에 살 수 있다. (다대다)  
답 : 중간테이블 `TENANT_APARTMENT` 를 생성하고 `TENANT_ID` 와 `APARTMENT_ID` 외래키 칼럼을 갖도록 처리한다.
> 

<br/>

### 4단계 : 행위 조사

최종으로 할 일은 **세부적인 부분을 채우는 작업**입니다.

**요구사항에 맞춰 주로 수행될 작업들을 살펴보고, 관련 데이터를 어떻게 저장하고 가져올 것인지 이해**해야 합니다.

<br/><br/>

## 마무리하며

이번에는 면접에서 나올 수 있는 DB 관련 문제(특히 설계!)를 어떻게 해결해야하는지 알아봤습니다.

다소 내용이 적은 것 같지만, 사실 DB 자체에서 나올 수 있는 질문의 양은 어마무시합니다.

따라서 이 글에서 다룬 설계 문제뿐만 아니라, 다른 기본 DB 지식을 알아두는게 좋겠습니다.