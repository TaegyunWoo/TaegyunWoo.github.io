---
category: TS
tags: [트러블슈팅, BeanValidation]
title: "Bean Validation과 400 오류"
date:   2021-09-24 17:30:00 
lastmod : 2021-09-24 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

- 문제 발생 개발기록
  - [CRUD Web 개발일지: 2021-09-24](https://taegyunwoo.github.io/CRUD_Web/2021-09-24)

<br/>

# 문제 배경

- `BoardController` 컨트롤러에 대한 테스트 코드를 작성하던 중, 에러가 발생했다.
- `BoardController` 클래스의 `addArticle` 메서드에 대한 테스트 작성하던 상태다.
- 테스트 코드 작성 완료 후 실행을 했는데, 400 오류가 반환되었다.
- `BoardController` 클래스의 `addArticle` 메서드
  ```java
  @PostMapping("/new-article")
  public String addArticle(@Validated @ModelAttribute("article") NewArticleForm form,
                          @SessionAttribute("loginUser") User user,
                          BindingResult bindingResult) {

    if (bindingResult.hasErrors()) {
      return "add-article";
    }

    //로직...

    return "redirect:/board/1";
  }
  ```
- `BoardControllerTest` 테스트 클래스의 `addArticleTest` 메서드
  ```java
  @DisplayName("게시글 추가")
  @Test
  void addArticleTest() throws Exception {
    //given
    NewArticleForm wrongForm = new NewArticleForm();
    User loginUser = new User();
    MockHttpSession session = new MockHttpSession();

    wrongForm.setTitle("");
    wrongForm.setContent("");
    loginUser.setId(5L);
    loginUser.setName("BoardTestUser");
    loginUser.setEmail("BoardTestUser@test.com");
    loginUser.setPassword("123test!");
    session.setAttribute("loginUser", loginUser);

    given(boardService.addNewArticle(any())).willReturn(true);


    //when
    ResultActions wrongPerform = mockMvc.perform(
        post("/board/new-article")
            .param("title", wrongForm.getTitle())
            .param("content", wrongForm.getContent())
            .session(session)
    );

    //then
    wrongPerform.andExpect(status().isOk()).andExpect(view().name("add-article")); //validation 실패시 기대 결과
  }
  ```
  - 의도적으로 Bean Validation에 실패하는 요청을 테스트했다.
  - `addArticle` 메서드에서 Bean Validation에 실패시, `add-article` 뷰를 응답하도록 로직 작성하였다.
  - 하지만, 400 오류가 발생하고 테스트에 실패했다.
  > Bean Validation에 실패하더라도 200 코드로 `add-article` 뷰를 응답하도록 작성했는대도 말이다.

<br/>

- 발생 오류
  ![Untitled](/assets/img/2021-09-24-TroubleShooting_BeanValidation400Error/Untitled.png)

<br><br>

# 문제 해결 탐색과정
- `@WebMvcTest`를 통해, 유닛 테스트를 진행하였다. 그에 따라, 테스트 과정에서 Bean Validation이 제대로 적용되는지 확인하였다.
  - 그 결과, Bean Validation은 정상 작동한다는 사실을 알았다.
- 실제 로컬에 Web App 실행하여 확인해본 결과, Bean Validation 조건에 일치하지 않는 요청시, 로컬에서도 동일하게 400 오류가 발생했다.
- 따라서, 컨트롤러 메서드의 매개변수 순서가 의심되었다.

<br><br>

# 문제 해결
## 문제 원인

- `BoardController` 컨트롤러 클래스의 `addArticle` 메서드의 매개변수가 문제였다.
- 해당 메서드의 매개변수 순서는 다음과 같다.  
  1. `@Validated @ModelAttribute("article") NewArticleForm form`
  2. `@SessionAttribute("loginUser") User user`
  3. `BindingResult bindingResult`

<br/>

- **위 순서와 같이 "`@Validated` 가 적용된 매개변수"와 "BindingResult 매개변수"가 떨어져서 작성되어 발생하는 문제였다.**

<br/>

- `BoardController` 클래스의 `addArticle` 메서드
  ```java
  @PostMapping("/new-article")
  public String addArticle(@Validated @ModelAttribute("article") NewArticleForm form,
                          @SessionAttribute("loginUser") User user,
                          BindingResult bindingResult) {

    if (bindingResult.hasErrors()) {
      return "add-article";
    }

    //로직...

    return "redirect:/board/1";
  }
  ```

<br><br>

## 해결방법

- `@Validated` 매개변수와 `BindingResult` 매개변수를 아래 순서와 같이 작성한다.
  1. `@Validated` 매개변수
  2. `BindingResult` 매개변수
- **항상 `@Validated` 바로 뒤에 `BindingResult` 매개변수가 와야한다.**
- 예시
  - `BoardController` 클래스의 `addArticle` 메서드
  ```java
  @PostMapping("/new-article")
  public String addArticle(@SessionAttribute("loginUser") User user,
                          @Validated @ModelAttribute("article") NewArticleForm form,
                          BindingResult bindingResult) {

    if (bindingResult.hasErrors()) {
      return "add-article";
    }

    //로직...

    return "redirect:/board/1";
  }
  ```