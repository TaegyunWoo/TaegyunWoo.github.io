---
category: TS
tags: [TS]
title: "@WebMvcTest로 컨트롤러 테스트시 ViewResolver 에러 발생 문제"
date:   2021-09-23 22:08:00 
lastmod : 2021-09-23 22:08:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 문제 발생 개발기록
  - [CRUD Web 개발일지: 2021-09-23](https://taegyunwoo.github.io/CRUD_Web/2021-09-23)

<br/>

# 문제 배경

- `BoardController` 컨트롤러에 대한 테스트 코드를 작성하던 중, 에러가 발생했다.
- `@WebMvcTest` 를 통해, 유닛 테스트를 작성하던 상태였다.
- `BoardController` 클래스의 `showArticleAddForm` 메서드에 대한 테스트 작성하던 상태다.

- 테스트 코드 작성 완료 후 실행을 했는데, View Resolver 관련 에러가 발생했다.
- `BoardController` 클래스의 `showArticleAddForm` 메서드
  ```java
  @GetMapping("/new-article")
  public String showArticleAddForm(@ModelAttribute("article") NewArticleForm form) {
      return "new-article";
  }
  ```
- `BoardControllerTest` 테스트 클래스의 `showArticleAddFormTest` 메서드
  ```java
  @DisplayName("새 게시글 작성 창 보이기")
  @Test
  void showArticleAddFormTest() throws Exception {
    //given
    NewArticleForm form = new NewArticleForm();
    form.setTitle("New Article Title");
    form.setContent("New Article Content");

    //when
    ResultActions actions = mockMvc.perform(
      get("/board/new-article")
    );

    //then
    actions.andExpect(status().isOk())
        .andExpect(view().name("new-article"));
  }
  ```
- 발생 오류
  ![Untitled](/assets/img/2021-09-23-TroubleShooting_TestViewResolverError/Untitled.png)

<br><br>

# 문제 해결 탐색과정
- 해당 에러 메시지에 대해 구글링을 하여 조사하였다.

<br><br>

# 문제 해결
## 문제 원인

- `BoardController` 컨트롤러 클래스의 `showArticleAddForm` 메서드의 문제였다.
- **"해당 메서드의 `@GetMapping`의 URI"와 "해당 메서드의 반환값인 뷰 이름"이 동일하여 발생하는 문제이다.**
- `BoardController` 클래스의 `showArticleAddForm` 메서드
  ```java
  @GetMapping("/new-article") //이것과
  public String showArticleAddForm(@ModelAttribute("article") NewArticleForm form) {
    return "new-article"; //이것이 동일한 문자열이다.
  }
  ```

<br><br>

## 해결방법

- 뷰의 이름과 매핑되는 URI 명을 다르게 변경하면 된다.
- 예시
  - `BoardController` 클래스의 `showArticleAddForm` 메서드
  ```java
  @GetMapping("/new-article")
  public String showArticleAddForm(@ModelAttribute("article") NewArticleForm form) {
    // return "new-article"; //기존 뷰이름
    return "add-article"; //새 뷰이름
  }
  ```