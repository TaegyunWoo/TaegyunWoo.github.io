---
category: Java
tags: [Java]
title: "Setter에서의 값 검증"
date:   2021-11-19 13:15:00 
lastmod : 2021-11-19 13:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 들어가며

얼마 전, 졸업작품 프로젝트를 진행하며 도메인을 설계하고 기본적인 엔티티를 작성하였다.  
엔티티를 작성하며 getter와 setter를 무의식적으로 작성하고 있었는데, 문득 이런 생각이 들었다.  
**"setter 메서드에서 값 검증 관련 예외를 터뜨리는게 좋은 방법인가?"**

<br/>

# 문제의식

setter 메서드는 엔티티의 필드 값을 설정해주고, 적절한 값이 들어가는지 확인하는 역할을 한다.
따라서 값 검증에 실패시 `IllegalArgumentException` 예외를 터뜨리도록 설계했다.  
**하지만 이렇게 setter에서 예외를 던지는 것이 Best Practice인지 궁금했다.**

<br/>

# 해결
## 해결과정

따라서 국내 개발자 커뮤니티 Okky에 질문 글을 남겼다.  
- 추가로 StackOverFlow에도 비슷한 내용의 질문이 있었지만, 답변이 모호했다...
- 관련 링크: [Is it a good practice to throw Exception inside setters in Java?](https://stackoverflow.com/questions/37092544/is-it-a-good-practice-to-throw-exception-inside-setters-in-java)

> 참고로 Okky는 java 개발자가 많다.

<br/>

질문 글  
![Untitled](/assets/img/2021-11-19-Java_Setter_Best_Practice/Untitled.png)

<br/>

## 답변

이에 대한 답변이 아래와 같이 달렸다.

![Untitled](/assets/img/2021-11-19-Java_Setter_Best_Practice/Untitled%201.png)

<br/>

## 정리
즉, 정리하자면 아래와 같다.

- 기본적인 검증을 통해 setter 메서드에서 예외를 던지는 것은 괜찮다!
- 하지만 특정 행동(Action)에 대한 검증이라면, 해당 검증 관련 로직은 따로 떼어내자.
  - 왜냐하면 **해당 검증 관련 로직은 특정 상황에서만 사용되기 때문**이다.
  - 예시
    - Action: 술을 산다.
    - Validation: 술을 살 때, 20세 이상인가?
    - 위와 같이, 특정 행동(Action)에 종속되는 검증(Validation)이라면, 검증 로직은 따로 떼어내야 한다.

<br/>

> Okky에 남긴 글을 보려면, [여기](https://okky.kr/article/1101050)로 가자.