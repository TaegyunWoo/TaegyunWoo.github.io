---
category: JPA
tags: [JPA]
title: "[JPA] DDL-AUTO 옵션을 통한 칼럼 업데이트"
date:   2022-05-17 14:19:00 
lastmod : 2022-05-17 14:19:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# DDL-AUTO 옵션을 통한 칼럼 업데이트

## 문제 상황

### 개요

- JPA(Hibernate ORM)을 사용하여 개발을 하던 중, '기존 테이블의 칼럼을 삭제'해야하는 상황이 발생했다.

### 당시 DDL-AUTO 설정

- 당시 개발환경에서는 아래와 같이 `DDL-AUTO` 옵션을 사용하던 중이었다.

```
spring.jpa.hibernate.ddl-auto=update
```

### `update` 옵션이란?

- `update` 옵션은 'JPA 엔티티 설계'와 '실제 테이블 상태'를 비교하여, **차이점이 있다면 그것을 테이블에 반영**하는 옵션이다.

### 당시 작업 내용

- 따라서 단순히 엔티티의 필드를 삭제하였고, 이것이 실제 테이블에도 반영되기를 기대했다.
- 예시) 기존 엔티티
    ```java
    public class Entity {
        @Id @GeneratedValue
        private Long id;

        @Column
        private String oldColumn1;

        @Column
        private String oldColumn2;
    }
    ```
- 예시) 수정된 엔티티
    ```java
    public class Entity {
        @Id @GeneratedValue
        private Long id;

        @Column
        private String oldColumn1;

        /* 필드 삭제
        @Column
        private String oldColumn2;
        */
    }
    ```

### 작업 결과

- **하지만 이것이 실제 테이블에 반영되지 않았다.**

<br/>

## 문제 원인
### `update`의 특성
- `update` 옵션이 엔티티 상태에 따라 테이블을 수정하는 옵션이긴 하지만, 칼럼 삭제의 경우에는 해당되지 않는다.
- **즉 `ddl-auto=update` 을 통해 칼럼을 삭제할 수 없다.**

### `update`와 칼럼 삭제
- 칼럼 삭제의 경우, 테이블의 칼럼이 엔티티에 존재하지 않아도 삭제되지 않는다.
- Why?
    - 이러한 동작은 의도된 설계이다.
    - 왜냐하면, 레거시 DB를 사용할 수 있도록 허용한 것이기 때문이다.
    - 즉 과거에 사용하던 DB를 사용할 수 있도록 설계된 사항이다.

### `update`와 칼럼 추가
- 덧붙이자면 칼럼 추가의 경우, 문제없이 추가된다.
- 예시) 기존 엔티티
    ```java
    public class Entity {
        @Id @GeneratedValue
        private Long id;

        @Column
        private String column1;

        @Column
        private String column2;
    }
    ```
- 예시) 수정된 엔티티
    ```java
    public class Entity {
        @Id @GeneratedValue
        private Long id;

        @Column
        private String column1;

        @Column
        private String column2;

        //필드 추가
        @Column
        private String column3;
    }
    ```
- **문제없이 실제 테이블에도 칼럼이 추가된다.**

<br/>

## 문제 해결 방안
### `ddl-auto=create-drop`을 사용하라
- `update`을 적용하여 칼럼을 삭제할 수 없으므로, `create-drop`을 통해 해결하자.
- `create-drop`은 애플리케이션이 실행될 때마다, 전체 테이블을 삭제하고 다시 처음부터 테이블을 생성한다.
- 따라서 엔티티에 존재하는 (매핑될 수 있는) 필드만 실제 테이블에 반영한다.

### 직접 DB에 접근하여, SQL을 통해 수정하라
- 사실 이 방법이 가장 안전한 방법이다.
- 왜냐하면 ORM 프레임워크를 100% 신뢰할 수는 없고, DB DDL을 전적으로 맡기기는 무리가 있기 때문이다.
- (일종의 블랙박스인)프레임워크가 DB를 조작한다면, 어떤 문제가 발생할지 예측할 수 없다.
- 따라서 실제로 프로젝트에서 해당 방법을 통해 칼럼을 삭제하였다.

<br/>

## 결론
- `ddl-auto=update` 설정으로 엔티티와 매핑되지 않는 칼럼을 삭제할 순 없다.
- 그 대안으로 아래 두 가지 방법이 있다.
  - `spring.jpa.hibernate.ddl-auto=update` 대신 `spring.jpa.hibernate.ddl-auto=create-drop` 을 사용
  - 직접 SQL을 작성하여 칼럼 삭제
- ORM 프레임워크가 직접 기존 칼럼을 삭제하는 등의 작업을 수행한다면, 어떤 문제가 발생할지 예측하기 어렵다.  
따라서 직접 SQL을 통해 테이블을 수정하는 것이 안전하다.

<br/>

> 만약 실제 운영환경이라면 당연히 `validate` 옵션을 사용해야 한다!  
(소중한 DB를 프레임워크에 맡기긴 어렵다...)

<br/><br/>

## REFERENCE
- [https://stackoverflow.com/questions/7079954/hibernate-hbm2ddl-auto-update-does-not-drop-columns-with-mysql](https://stackoverflow.com/questions/7079954/hibernate-hbm2ddl-auto-update-does-not-drop-columns-with-mysql)

<br><br>