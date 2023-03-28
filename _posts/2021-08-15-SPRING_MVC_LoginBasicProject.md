---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 로그인처리 - 기본 프로젝트 설정"
date:   2021-08-15 13:30:00 
lastmod : 2021-08-15 13:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 기본 프로젝트

## 개요

앞으로 로그인 처리에 대해 설명할 예정이다. 로그인 처리에 대해 설명하기 전, **설명의 바탕이 될 프로젝트를 생성**하자. 그리고 기본적인 코드를 작성해두자.

> [기초 프로젝트 다운로드](https://github.com/TaegyunWoo/TaegyunWoo.github.io/blob/master/ExampleProjects/myPrac.zip)

<br/><br/>

## 기본 프로젝트 설명

### 제공 기능

- 회원가입 (검증 기능 포함)
- 로그인 (검증 기능 포함)
- 회원 정보 조회

<br/>

### 예시

- 홈

    ![홈](/assets/img/2021-08-15-SPRING_MVC_LoginBasicProject/Untitled%2026.png)

- 회원가입

    ![회원가입](/assets/img/2021-08-15-SPRING_MVC_LoginBasicProject/Untitled%2027.png)

- 로그인

    ![로그인](/assets/img/2021-08-15-SPRING_MVC_LoginBasicProject/Untitled%2028.png)

<br/>

### 주요 소스코드

- **컨트롤러: 회원가입**

    ```java
    import ...

    @Controller
    @RequestMapping("/member")
    public class UserAddController {
      private final UserRepository userRepository;

      @Autowired
      public UserAddController(UserRepository userRepository) {
          this.userRepository = userRepository;
      }

      @GetMapping("/add")
      public String viewUserAddForm(@ModelAttribute User user) {
          return "addUser";
      }

      @PostMapping("/add")
      public String addUser(@Validated @ModelAttribute("user") AddUserForm form,
                            BindingResult bindingResult) {

          if (bindingResult.hasErrors()) {
              return "addUser";
          }

          User user = new User();
          user.setId(form.getId());
          user.setPassword(form.getPassword());
          user.setName(form.getName());

          userRepository.addUser(user);

          return "redirect:/";
      }

    }
    ```

<br/>

- **컨트롤러: 로그인**

    ```java
    import ...

    @Controller
    @RequestMapping("/member")
    public class LoginController {

      private final LoginService loginService;

      @Autowired
      public LoginController(LoginService loginService) {
          this.loginService = loginService;
      }

      @GetMapping("/login")
      public String viewUserLoginForm(@ModelAttribute User user) {
          return "login";
      }

      @PostMapping("/login")
      public String login(@Validated @ModelAttribute("user") LoginUserForm form,
                          BindingResult bindingResult) {

          //검증 오류시
          if (bindingResult.hasErrors()) {
              return "login";
          }

          //로그인 실행
          User user = loginService.login(form.getId(), form.getPassword());

          //로그인 실패시, 오브젝트 에러
          if (user == null) {
              bindingResult.reject("wrongUser", "아이디나 비밀번호를 확인하세요.");
              return "login";
          }

          // 로그인 성공시, TODO

          return "redirect:/";

      }
    }
    ```

> 나머지 상세 코드는 첨부파일을 통해 확인하자!

<br/>

## 정리

이제 기본적인 프로젝트 세팅은 끝났다. 다음 게시글에서 본격적으로 로그인 처리에 대해 알아보자.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*