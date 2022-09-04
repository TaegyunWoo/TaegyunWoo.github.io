---
category: Tech
tags: [DB]
title: "Cursor Pagination 과 무한 스크롤"
date:   2022-09-04 22:30:00 
lastmod : 2022-09-04 22:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

## 개요

최근 프로젝트를 진행하다 페이지 처리 관련 문제가 발생했고, 글을 통해 어떻게 해결했는지 정리하고자 한다.

<br/>

### 요구사항

프로젝트 개발 요구사항은 아래와 같다.

- 글 목록 조회를 무한 스크롤이 가능하도록 해야한다.
- `작성일 내림차순` , `조회수 내림차순` , `댓글 수 내림차순` 으로 조회할 수 있어야 한다.

<br/>

### 무한 스크롤이랑 일반 게시판형이 뭐가 달라?

무한 스크롤이 가능하도록 페이지네이션을 하는 것은 처음이었다.

나는 단순히 게시판형 페이징 API 를 만들고, 스크롤 이벤트 마다 클라이언트가 API를 호출하면 되지 않을까 라고 생각했다.

하지만 이 생각이 틀렸음을 동료 덕분에 알았다.

> 땡큐 woooooooody!

<br/>

### 무한 스크롤 vs 일반 게시판형

일반 게시판에서 페이지네이션을 구현할 때, 보통 `OFFSET` 과 `LIMIT` 쿼리를 사용하여 구현한다.

성능적으로 좋지 않다고 하지만, 아주 단순하여 널리 사용되고 있다.

그렇다면 이것을 무한 스크롤에 적용을 하면 어떤 문제가 발생할까?

아래와 같이 게시글 테이블(`ARTICLE`)이 있다고 해보자.

| ID | TITLE | BODY | VIEWS | COMMENT_COUNT |
| --- | --- | --- | --- | --- |
| 1 | Article 1 | Lorem ipsum dolor sit amet… | 13 | 3 |
| 2 | Article 2 | Lorem ipsum dolor sit amet… | 13 | 2 |
| 3 | Article 3 | Lorem ipsum dolor sit amet… | 4 | 0 |
| 4 | Article 4 | Lorem ipsum dolor sit amet… | 23 | 5 |
| 5 | Article 5 | Lorem ipsum dolor sit amet… | 19 | 0 |
| 6 | Article 6 | Lorem ipsum dolor sit amet… | 7 | 3 |

`OFFSET` 과 `LIMIT` 을 사용하여 첫번째 페이지를 조회해보자.

페이지당 3개 아이템이 출력되고, 최신순으로 조회한다고 가정한다. (최신순 = `ID` 오름차순)

```sql
SELECT * FROM ARTICLE
  ORDER BY ID DESC
  LIMIT 3 OFFSET 0;
```

위 쿼리에 대한 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled.png)

<br/><br/>

그럼 두번째 페이지를 조회해보자.

```sql
SELECT * FROM ARTICLE
  ORDER BY ID DESC
  LIMIT 3 OFFSET 3;
```

아래는 그 결과이다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%201.png)

<br/><br/>

여기까지는 아주 정상적으로 동작한다.

<br/>

### 문제가 되는 상황

**그렇다면 두번째 페이지를 조회하기 전에, 새로운 게시글이 추가되면 어떻게 될까?**

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%202.png)

<br/><br/>

위 상태에서 아까와 동일한 쿼리로 두번째 페이지를 조회하면 아래와 같이 될 것이다.

```sql
-- 이전과 동일한 쿼리
SELECT * FROM ARTICLE
  ORDER BY ID DESC
  LIMIT 3 OFFSET 3;
```

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%203.png)

<br/><br/>

결과적으로 `ID` 가 4인 `ARTICLE` 이 중복되어 조회된다.

이런 현상이 단순한 게시판형 서비스라면 크게 문제되지 않을 수 있다.

**하지만 무한 스크롤 형식의 서비스라면, 사용자가 직전에 확인한 `ARTICLE` 이 바로 다음에 다시 한번 중복되어 출력되는 심각한 문제가 있다.**

<br/><br/>

## 중복 조회 해결 방법

그렇다면 이런 문제를 어떻게 해야할까?

그 답은 바로 Cursor 기반 Pagination 에 있다!

<br/>

### Cursor Pagination 이 뭐야?

페이징을 하는 방식에는 크게 `OFFSET` 기반 방식과 `Cursor` 기반 방식이 있다.

`OFFSET` 방식은 우리가 지금까지 살펴본 방식이다.

`OFFSET` 방식은 **"어디까지 생략하고 조회할 것인지"** 에 집중한다.

그에 반해, `Cursor` 방식은 **"어떤 아이템 이후부터 조회해야 하는지"** 에 포커스가 있다.

바로 알아보자.

<br/>

### Cursor Pagination 를 사용해보자.

아까 살펴본 `OFFSET` 방식 대신 `Cursor` 방식을 통해 Pagination 을 해보자.

첫번째 페이지를 조회하는 쿼리는 아래와 같다.

```sql
SELECT * FROM ARTICLE
	WHERE ID < 무한대
	ORDER BY ID DESC
	LIMIT 3;
```

`OFFSET` 이 전혀 사용되지 않은 것을 확인할 수 있다.

`무한대` 를 사용한 이유는 좀 더 지나서 설명하겠다.

결과적으로 위 쿼리를 실행하면, 아래와 같이 조회가 된다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%204.png)

<br/><br/>

`OFFSET` 을 사용하여 조회한 것과 동일한 결과이다.

계속해서 두번째 페이지를 조회해보자.

아래는 두번째 페이지를 조회하는 쿼리이다.

```sql
SELECT * FROM ARTICLE
  WHERE ID < 4
  ORDER BY ID DESC
	LIMIT 3;
```

`WHERE` 절을 보자. 마지막으로 조회된 `ARTICLE`의 `ID` 인 4보다 작은 `ARTICLE` 만 조회하도록 조건이 걸렸다.

이것을 통해 `OFFSET` 을 전혀 사용하지 않고 원하는 결과를 얻을 수 있다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%205.png)

여기에서 핵심은 **"바로 직전 아이템의 `ID` 4를 사용했다는 점"** 이다.

즉 **"어떤 아이템 이후부터 조회할 것인지"** 에 집중하게 되었고, 그 결과가 정상적으로 나왔다.

이러한 동작방식 때문에, 첫번째 페이지를 조회할 때 `무한대` 를 사용한다.

<br/>

### 그럼 중복 조회되는 문제는?

`OFFSET` 방식을 사용할 때, 새로운 튜플이 추가되면 이전에 조회한 튜플이 다시 한번 더 조회되는 문제가 있었다.

다시 동일하게 두번째 페이지를 조회해보자.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%206.png)

<br/><br/>

위 그림과 같은 상태에서 두번째 페이지를 조회하는 쿼리를 실행시켜보자.

```sql
-- 직전에 살펴본 쿼리와 동일하다
SELECT * FROM ARTICLE
  WHERE ID < 4
  ORDER BY ID DESC
	LIMIT 3;
```

그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%207.png)

전혀 중복된 `ARTICLE` 이 조회되지 않았다!

**왜냐하면 직전까지 받은 아이템의 `ID` 이후의 아이템들을 조회했기 때문이다.**

이를 통해, 기존의 `OFFSET` Pagination 의 문제점을 해결할 수 있음을 확인했다.

> 참고로 `Cursor` 방식을 사용하는 것이 성능적으로도 우수하다!  
자세한 것은 [https://jojoldu.tistory.com/528](https://jojoldu.tistory.com/528) 을 참고하자.

<br/><br/>

## Cursor Pagination 의 한계

나는 이 방식을 통해, 무한 스크롤에 필요한 조회 API 를 구현하려고 했다.

하지만 또 다른 문제가 있었는데…

다시 요구사항을 보자.

- 글 목록 조회를 무한 스크롤이 가능하도록 해야한다.
- **`작성일 내림차순` , `조회수 내림차순` , `댓글 수 내림차순` 으로 조회할 수 있어야 한다.**

위 요구사항에서 주목해야하는 것은 2번째 항목이다.

`Cursor` Pagination 을 사용하여 `작성일 내림차순` 조회를 무한 스크롤로 구현할 수 있다.

문제는 `조회수 내림차순` , `댓글수 내림차순` 조회이다.

그대로 `Cursor` Pagination 을 사용해서 `조회수 내림차순` , `댓글수 내림차순` 으로 조회를 하면 문제가 발생한다.

<br/>

### 기존 방식의 Cursor 페이징으로 조회수 내림차순 조회

| ID | TITLE | BODY | VIEWS | COMMENT_COUNT |
| --- | --- | --- | --- | --- |
| 1 | Article 1 | Lorem ipsum dolor sit amet… | 13 | 3 |
| 2 | Article 2 | Lorem ipsum dolor sit amet… | 13 | 2 |
| 3 | Article 3 | Lorem ipsum dolor sit amet… | 4 | 0 |
| 4 | Article 4 | Lorem ipsum dolor sit amet… | 23 | 5 |
| 5 | Article 5 | Lorem ipsum dolor sit amet… | 19 | 0 |
| 6 | Article 6 | Lorem ipsum dolor sit amet… | 7 | 3 |

위 테이블(`ARTICLE`)을 조회수 내림차순으로 첫번째 페이지를 조회한다면, 아래와 같이 쿼리를 작성해야  한다.

```sql
SELECT * FROM ARTICLE
	WHERE VIEWS < 무한대
	ORDER BY VIEWS DESC
	LIMIT 3;
```

`WHERE` 절에 조건으로 `ID` 대신 `VIEWS` 가 들어갔다.

그리고 정렬 기준이 `VIEWS` 가 된다.

그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%208.png)

<br/><br/>

여기까지 살펴봤을 때, 정상적으로 동작한다.

바로 아래 쿼리를 실행시켜 다음 두번째 페이지를 조회해보자.

```sql
SELECT * FROM ARTICLE
	WHERE VIEWS < 13
	ORDER BY VIEWS DESC
	LIMIT 3;
```

첫번째 페이지에서 조회한 가장 마지막 `ARTICLE` 의 `VIEWS` 가 13이므로, `VIEWS < 13` 을 사용하여 조회한다.

그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%209.png)

<br/><br/>

결과가 우리가 원하는대로 나오지 않았다.

분명히 `ID` 가 1인 `ARTICLE` 부터 조회를 해야하지만 그렇지 않다.

그러면 조건절을 아래와 같이 바꾸면 해결이 될까?

```sql
-- WHERE 절 조건이 '<=' 으로 변경됨
SELECT * FROM ARTICLE
	WHERE VIEWS <= 13
	ORDER BY VIEWS DESC
	LIMIT 3;
```

그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2010.png)

<br/><br/>

당연하게도 이미 조회했던 `ID` 가 2인 `ARTICLE` 이 중복되어 조회된다.

어떻게 해결해야 할까?

<br/>

### Cursor Pagination 은 유니크한 값으로 해야 한다!

**문제는 `VIEWS` 라는 컬럼이 유니크한 값을 갖지 않는다는 것이다.**

Cursor Pagination 을 사용할 때는 무조건 유니크한 값을 기준으로 처리해야 한다.

그렇지 않으면 위 문제가 발생한다.

**위 문제를 해결하기 위해, 유니크한 값을 갖는 `ID` 컬럼과 함께 사용해보자!**

<br/>

### 정렬 기준 (ORDER BY) 추가

그 전에 우리는 `VIEWS` 가 같은 경우에 어떻게 정렬할 것인지 정해야 한다.

그리고 그 정렬 기준으로 유니크 컬럼을 사용해야 정상적으로 동작한다.

유니크 컬럼인 `ID` 를 사용해서 아래와 같이 정렬해보자.

```sql
SELECT * FROM ARTICLE
  ORDER BY VIEWS DESC, ID DESC;
```

`VIEWS` 를 기준으로 내림차순으로 정렬을 하고, 만약 `VIEWS` 값이 동일하다면 유니크 컬럼인 `ID` 을 기준으로 다시 내림차순 정렬을 한다.

<br/>

### 중복 가능한 컬럼과 유니크 컬럼으로 Cursor Pagination 구현

자 그러면 이제 본격적으로 문제를 해결해보자.

다시 두번째 페이지를 조회해보자.

아래 그림은 첫번째 페이지를 조회한 상태이다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2011.png)

<br/><br/>

여기에서 아래 쿼리를 실행시켜 두번째 페이지를 조회해보자.

```sql
SELECT * FROM ARTICLE
  WHERE (VIEWS < 13) OR (VIEWS = 13 AND ID < 2)
  ORDER BY VIEWS DESC, ID DESC
	LIMIT 3;
```

역시 주목해야하는 포인트는 `WHERE` 절의 조건이다.

- `(VIEWS < 13)`
    - 가장 마지막에 조회한 `ARTICLE` 의 `VIEWS` 보다 작은 것을 조회한다.
    - 아까 우리가 살펴본 문제(동일한 `VIEWS` 를 갖는 `ARTICLE` 이 누락되는 문제)가 발생한다.
    - 따라서 다음 조건이 추가되었다.
- `(VIEWS = 13 AND ID < 2)`
    - `VIEWS` 가 동일하게 13이고, 가장 마지막에 조회한 `ARTICLE` 의 `ID` 보다 작은 것을 조회한다.
    - **`ID` 는 유니크한 값을 갖기 때문에, `(VIEWS < 13)` 조건 만으로는 누락시킬 수 있는(동일한 `VIEWS` 값을 갖는) `ARTICLE` 까지 조회할 수 있게 된다.**

`(VIEWS < 13)` 조건과 `(VIEWS = 13 AND ID < 2)` 조건을 함께 사용함으로써 누락되는 `ARTICLE` 이 없도록 방지할 수 있게 되었다!

실제로 그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2012.png)

<br/><br/>

정상적으로 동작한다.

<br/>

### 새 아이템이 추가되어도 괜찮을까?

그렇다면 새로운 `ARTICLE` 이 추가되었을 땐 어떻게 될까?

첫번째 페이지를 조회했다는 전제하에 설명하겠다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2013.png)

<br/><br/>

위 상태에서 새로운 아이템이 추가되면 아래와 같이 테이블의 상태가 변경된다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2014.png)

<br/><br/>

이렇게 새로 추가된 `ARTICLE` 이 엄청난 주목을 받아 조회수가 폭발했다고 해보자!

그러면 아래와 같이 테이블 상태가 변경될 것이다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2015.png)

<br/><br/>

이제 아래 쿼리로 두번째 페이지를 다시 조회해보자.

```sql
-- 직전에 사용한 쿼리와 동일
SELECT * FROM ARTICLE
  WHERE (VIEWS < 13) OR (VIEWS = 13 AND ID < 2)
  ORDER BY VIEWS DESC, ID DESC
	LIMIT 3;
```

그 결과는 아래와 같다.

![Untitled](/assets/img/2022-09-04-Tech_InfiniteScroll_Cursor_Pagination/Untitled%2016.png)

<br/><br/>

결과적으로 문제없이 동작한다.

<br/><br/>

## 정리

이번 이슈로 페이징 처리에 대해 더 깊이 있는 고민을 해볼 수 있었다.

지금까지 페이징 처리를 단순하게 생각해왔지만, 고려해볼 것이 많이 주제였다는 것을 알았다.

이번 글에서 "무한 스크롤에서의 중복 아이템 문제" 만을 다뤘지만, 사실 더 많은 이야기가 있다.

페이징 처리를 어떻게 구현하느냐에 따라 성능적으로도 많은 차이를 가져올 수 있다는 것, 역시 더 심도있게 조사해봐야하는 주제이다.

이번 글을 통해, 보다 많은 사람들이 무한 스크롤에 대한 궁금증을 풀었기를 바란다.

<br/><br/>

### Reference

- [JOJOLDU - "페이징 성능 개선하기 - No Offset 사용하기"](https://jojoldu.tistory.com/528)
- [minsangk - "커서 기반 페이지네이션 (Cursor-based Pagination) 구현하기"](https://velog.io/@minsangk/%EC%BB%A4%EC%84%9C-%EA%B8%B0%EB%B0%98-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98-Cursor-based-Pagination-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)
- [devjem - "[프로젝트] No-offset 방식과 Spring Data JPA Slice를 사용해서 무한 스크롤 구현하기"](https://devjem.tistory.com/74)