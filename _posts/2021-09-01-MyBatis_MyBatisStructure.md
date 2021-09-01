---
category: MyBatis
tags: [MyBatis, 마이바티스]
title: "MyBatis와 SpringBoot 구조"
date:   2021-09-01 17:30:00 
lastmod : 2021-09-01 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# MyBatis와 MyBatis-Spring 구조

## 개요

- 스프링부트, 마이바티스, H2 DB 를 이용한다.

<br/><br/>

## 구조

### 전체 Application 흐름

![전체 app](/assets/img/2021-09-01-MyBatis_MyBatisStructure/Untitled.png)

<br/>

### MyBatis 구조

![MyBatis 구조](/assets/img/2021-09-01-MyBatis_MyBatisStructure/Untitled%201.png)

- **`MyBatis Config 파일`**
    - DB 주소, Mapping 파일(SQL파일) 경로 설정 파일이다.
- **`Mapping 파일`**
    - SQL 문, OR Mapping 설정 파일이다.
- **`SqlSessionFactoryBuilder`**
    - `MyBatis Config 파일` (설정파일)을 바탕으로 `SqlSessionFactory` 을 생성한다.
- **`SqlSessionFactory`**
    - `SqlSession` 을 생성한다.
- **`SqlSession`**
    - 핵심 역할을 수행한다.
    - SQL 실행, 트랜잭션 관리 등을 수행한다.
    - Thread-Unsafe
    - 쓰레드마다 생성된다.

<br/>

### MyBatis-Spring 구조

![MyBatis-Spring 구조](/assets/img/2021-09-01-MyBatis_MyBatisStructure/Untitled%202.png)

- **`SqlSessionFactoryBean`**
    - 개발자가 직접 Spring Bean으로 등록해야한다.
    - `MyBatisConfig 파일` 을 기반으로 `SqlSessionFactory` 를 생성한다.
- **`MyBatisConfig 파일`**
    - vo 객체 정보를 설정한다.
- **`SpringBean 설정 파일`**
    - DB 접속 정보, Mapping 파일의 경로를 설정한다.
- **`Mapping 파일`**
    - SQL 문, OR Mapping 을 설정한다.
- **`SqlSessionTemplate`**
    - 핵심 역할을 담당한다.
    - SQL 실행, 트랜잭션 관리 등을 수행한다.
    - `SqlSession 인터페이스` 의 구현체이다.
    - Thread-safe

<br/>

## MyBatis-Spring-Boot-Starter 효과

- 스프링 부트에 MyBatis를 편리하고 빠르게 적용할 수 있다.
- `@Mapper` 생성외의 다른 필수 설정 작업들을 자동으로 수행해준다.
- 즉, MyBatis를 사용하기 위해 수행해야했던 갖가지 설정을 할 필요가 없어서 사용하기 편리하다.
- MyBatis-Spring-Boot-Starter 가 수행하는 작업
    - `DataSource` 자동 감지
    - `SqlSessionFactory` 를 전달하는 인스턴스를 자동으로 생성하고 등록한다.
    - `SqlSessionFactoryBean` 의 인스턴스를 만들고 등록한다.
    - `@Mapper` 주석이 표시된 매퍼를 자동으로 스캔하고 연결한다.
    - `SqlSessionTemplateSpring` 컨텍스트에 등록하여 Spring Bean에 주입 할 수 있도록 한다.

> 참고 자료: [https://kils-log-of-develop.tistory.com/576](https://kils-log-of-develop.tistory.com/576)