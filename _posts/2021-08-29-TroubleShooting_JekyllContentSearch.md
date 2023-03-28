---
category: TS
tags: [TS]
title: "Simple-Jekyll-Search에서 게시글 content 내용 검색이 안되는 문제와 해결방법"
date:   2021-08-29 18:30:00 
lastmod : 2021-08-29 18:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 문제 배경
- 현재 블로그 테마로 사용 중인 [`jekyll-theme-textalic`](https://github.com/UniFreak/jekyll-theme-textalic)의 검색기능이 제대로 작동하지 않는다.

- 제목, 날짜 등의 검색조건으로 검색은 정상적으로 작동하지만, **작성된 내부 글(post.content)로는 검색이 되지않는 문제**가 발견되었다.

- 문제 상황  
  ![문제 상황](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%201.png)  
  ![문제 상황](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled.png)

  - **검색어(`이전 게시글`)이 다른 포스트들에 작성되어있지만, 검색되지 않는다.**

<br><br>

# 문제 해결 탐색과정

- 해당 테마 (`jekyll-theme-textalic`)가 검색기능으로 사용 중인 `Simple-Jekyll-Search` 의 문서를 읽어보았다. ([관련 문서](https://github.com/christian-fei/Simple-Jekyll-Search/blob/master/README.md))

- 해당 문서에서 다음과 같은 내용에 주목하였다. ([내용](https://github.com/christian-fei/Simple-Jekyll-Search/blob/master/README.md#enabling-full-text-search))  
  ![내용](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%203.png)

<br><br>

# 문제 해결
## 문제 원인

- 위 그림에서 표시된 부분을 보자. 해당 부분에는 `post.content` 로 작성이 되어있다.

- 하지만, 테마 `jekyll-theme-textalic`에서의 해당 부분은 아래 그림과 같이 작성되어있다.  
  ![내용](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%204.png)

- 5번째 라인의 `for`문을 보면, post 라는 변수로 `site.posts`의 요소를 담는데 뜬금없이 `page.content`로 작성되어있다.

- **`page.content` 의 값이 null이므로, 결과적으로 게시글의 내용으로 검색이 되지 않았다.**

<br><br>

## 해결순서

### 테마 루트경로에 위치한 `search.json` 파일 수정

- `search.json` 파일을 다음과 같이 수정한다.

- 기존 내용  
  ![내용](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%204.png)

- 수정된 내용  
  ![내용](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%205.png)

<br/>

## Pull Request
- 해당 문제를 해결하여 `jekyll-theme-textalic` Github 저장소에 PR을 해둔 상태이다.

![PR](/assets/img/2021-08-29-TroubleShooting_JekyllContentSearch/Untitled%202.png)