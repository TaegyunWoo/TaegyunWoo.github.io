---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 스프링부트와 H2 연결 세팅"
date:   2021-09-05 16:10:00 
lastmod : 2021-09-05 16:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 스프링부트와 H2 데이터베이스
## 개요
- 본 게시글은 개발용으로 자주 사용되는 H2 DB를 스프링부트와 연동하는 방법에 대해서 다룬다.

<br/>

## H2 작동방식
### Embedded 모드
- H2 데이터베이스를 JVM 위에서 구동시킨다.
- Application (JVM)이 종료되면, 저장 혹은 수정한 데이터가 모두 날라간다.
- 즉, 휘발성을 갖는다.
- DB 접근 속도가 빠르다.

### Server 모드
- 별도의 프로세스를 가지고, Application과 독립적으로 작동한다.
- 즉, 데이터베이스 서버를 설치한 것과 동일한 개념이다.
- Application과 TCP/IP 통신을 통해 데이터를 주고 받는다.
- DB 접근 속도가 비교적 느리다.

### 추천 모드
- 프로젝트 개발단계에서 Embedded 모드를 사용하는 것이 편리하다.
- 왜냐하면, 테스팅에 사용되는 더미 데이터가 저장되지 않고(휘발) 반응 속도가 비교적 빠르기 때문이다.

<br/>

## 연동 방법

### 1. build.gradle 파일 작성
- H2를 사용하기 위해, dependencies 설정을 해야한다.
```gradle
	implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'
	implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'
	runtimeOnly 'com.h2database:h2'
```
- `implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'`
  - MyBatis 프레임워크를 사용한다면, 추가한다.

<br/>

### 2. application.properties 파일 작성
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.datasource.url=jdbc:h2:mem:testdb;
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

- `jdbc:h2:mem:testdb`
  - In-Memory 주소이다.
  - 해당 주소(mem)로 설정해야 In-Memory DB를 사용할 수 있다.
  - 만약 In-Memory가 아닌 영속성을 갖는 H2 DB를 사용하려면 아래 주소를 입력하면 된다.
    - `jdbc:h2:~/test`

- `spring.h2.console.enabled=true`
  - in-memory 방식을 사용하는 경우, H2 콘솔을 사용할 것인지의 유무이다.

- **In-Memory 모드이기 때문에 Application이 시작될 때마다, 전체 DB가 초기화된다.**
- **따라서 Application이 시작될 때마다, 테이블과 데이터(튜플)을 생성해주어야 한다.**
  - schema.sql 과 data.sql 파일을 통해 해결한다.
  - Application 실행시, schema.sql 과 data.sql 파일에 작성한 내용을 자동적으로 수행해준다.

<br/>

### 3. schema.sql 파일 작성
- 파일 경로
  - `루트/src/main/resources/schema.sql`
- **해당 파일에는 보통 테이블 생성, 수정 등의 DDL 스크립트를 작성한다.**
- 대체로 사용할 테이블을 생성하는 쿼리문을 작성하면 된다.
- 예시
  ```sql
    CREATE TABLE member (
        id BIGINT(5) NOT NULL AUTO_INCREMENT,
        email VARCHAR(255) NOT NULL,
        name VARCHAR(255) NOT NULL,
        password VARCHAR(255) NOT NULL,
        PRIMARY KEY (id)
    );
  ```

<br/>

### 4. data.sql 파일 작성
- 파일 경로
  - `루트/src/main/resources/data.sql`
- **해당 파일에는 보통 레코드(튜플) 추가, 수정 등의 DML 스크립트를 작성한다.**
- 대체로 테스트용으로 사용할 더미데이터를 추가하는 쿼리문을 작성하면 된다.
- 예시
  ```sql
    INSERT INTO member (email, name, password) VALUES ('test@test.com', 'test', '123test!');
  ```

