---
category: TS
tags: [TroubleShooting, 지킬블로그, 제목대문자]
title: "지킬블로그 포스트 제목에서 대문자가 정상적으로 표현되지 않는 문제와 해결법"
date:   2021-07-26 13:30:00 
lastmod : 2021-07-26 13:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 문제 배경
- 블로그 포스팅 중 제목에서 영문 대문자가 모두 소문자로 표현되는 문제를 발견했다.

- 본인은 게시글의 제목을 아래 사진과 같이 설정했다.

  ![제목설정](/assets/img/2021-07-26-TroubleShooting_JekyllTitleUpperCase/Untitled%201.png)

- 보통 `게시글.md` 파일의 제목으로 게시글의 제목을 설정하지만, 본인은 위 방식이 더욱 편리하다고 생각하여, 이와 같이 블로깅을 하는 중이다.

<br><br>

# 문제 해결 탐색과정

- 구글링을 통해 조사를 해본바, 본인과 동일한 문제를 찾을 수는 없었다.
  > 아마 블로그 테마에 따라 설정이 달라서 발생하는 문제일 수 있다고 생각된다.

- 하지만, 카테고리의 첫글자를 제외한 영어 대문자가 모두 소문자로 표현되는 문제는 발견할 수 있었다.
> [관련 페이지1](https://shopify.github.io/liquid/filters/capitalize/),  
[관련 페이지2](https://stackoverflow.com/questions/19074064/why-jekyll-convert-my-capital-words-into-lowercase-in-categories)

- 따라서, 해당 해결방법을 참고하여 문제를 해결했다.

<br><br>

# 문제 해결
## 문제 원인

- `_includes\post_list_item.html` 파일
- `_layouts\post.html` 파일

위 두 가지 파일에 존재하는 `capitalize` 옵션이 문제인 것을 확인했다.

<br>

### `capitalize`
해당 옵션이 **특정 문자열의 가장 첫번째 글자만 대문자로 변경한 뒤, 나머지 글자는 모두 소문자로 변경하는 것** 으로 추정된다.

따라서, 해당 부분을 관련 파일에서 모두 지우는 방법으로 문제를 해결했다.

<br><br>

## 해결방법

### `_includes\post_list_item.html` 파일 내용 변경

- 해당 파일은 **게시글 목록에서 보여지는 게시글** 에 대한 파일이다.

- 즉, **게시글 목록에서 보여지는 게시글의 제목** 을 수정하는 과정이다.
- 주석으로 처리된 부분 (Line: 12) 을 그 아랫줄 (Line: 13) 과 같이 변경하였다.

![_includes\post_list_item.html](/assets/img/2021-07-26-TroubleShooting_JekyllTitleUpperCase/Untitled%202.png)

<br>

### `_layouts\post.html` 파일 내용 변경

- 해당 파일은 **게시글 자체** 에 대한 파일이다.

- 즉, **게시글을 클릭하여 들어갔을 때의 게시글의 제목** 을 수정하는 과정이다.
- 주석으로 처리된 부분 (Line: 8) 을 그 아랫줄 (Line: 9) 과 같이 변경하였다.

![_layouts\post.html](/assets/img/2021-07-26-TroubleShooting_JekyllTitleUpperCase/Untitled%203.png)