---
category: Spring-MVC
tags: [스프링, MVC, 요청매핑]
title: "[스프링 - MVC] @RequestMapping과 원리"
date:   2021-07-27 20:30:00 
lastmod : 2021-07-27 21:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

스프링은 매우 유연하고, 실용적인 컨트롤러를 만들 수 있도록 `@RequestMapping` 애노테이션을 지원한다.

<br>

# `@RequestMapping` 애노테이션

## `@RequestMapping` 의 목적

- 애노테이션을 활용하여 매우 유연하고, 실용적인 컨트롤러를 만들 수 있도록 하는 것

- RequestMappingHandlerMapping , RequestMappingHandlerAdapter
    - 위 두가지 기능(핸들러 매핑 , 핸들러 어댑터)을 모두 수행하기 위한 애노테이션이다.
- 대부분 실무에선 `@RequestMapping` 을 사용한다.

<br><br>

## `@RequestMapping` 기반의 스프링 MVC 컨트롤러 - V1

`@RequestMapping` 애너테이션을 적용한 컨트롤러를 작성해보자. 이전에 다뤘던 회원 도메인을 기반으로 컨트롤러를 작성할 것이다.

> 회원 도메인 관련 코드는 [이전 글](https://taegyunwoo.github.io/spring/SPRING_OCP_DIP)의 도메인 부분을 참고하자.

<br>

### SpringMemberFormControllerV1 - 회원 등록 폼

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class SpringMemberFormControllerV1 {
	
	@RequestMapping("/springmvc/v1/members/new-form")
	public ModelAndView process() {
		return new ModelAndView("new-form");
	}

}
```

- `@Controller`

    - 스프링이 자동으로 스프링 빈으로 등록한다.
    - `@Controller` 는 `@Component` 를 포함하고 있어서, `@ComponentScan` 의 대상이 된다.
- `@RequestMapping`
    - 요청 정보를 매핑한다. 해당 URL이 호출되면 해당 메서드가 호출된다.
    - 메서드 이름은 임의로 지어도 된다. (애노테이션 기반으로 작동하기 때문에)
- `ModelAndView`
    - 모델과 뷰 정보를 담아서 반환하면 된다.

<br>

### SpringMemberSaveControllerV1 - 회원 저장

```java
import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class SpringMemberSaveControllerV1 {
	
	private MemberRepository memberRepository = MemberRepository.getInstance();
	
	@RequestMapping("/springmvc/v1/members/save")
	public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
		
		//전달받은 파라미터 얻기
		String username = request.getParameter("username");
		int age = Integer.parseInt(request.getParameter("age"));
		
		Member member = new Member(username, age);
		System.out.println("member = " + member);
		
		memberRepository.save(member);
		
		ModelAndView mv = new ModelAndView("save-result");
		
		//ModelAndView에 member 객체 추가
		mv.addObject("member", member);

		return mv;
	}
}
```

- `mv.addObject("member", member)`

    - `ModelAndView` 를 통해 Model 데이터를 추가할 때 사용하는 메서드
    - 해당 Model 데이터 (`member`)는 뷰를 렌더링할 때 사용된다.

<br>

### SpringMemberListControllerV1 - 회원목록

```java
import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import java.util.List;

@Controller
public class SpringMemberListControllerV1 {

	private MemberRepository memberRepository = MemberRepository.getInstance();

	@RequestMapping("/springmvc/v1/members")
	public ModelAndView process() {
		List<Member> members = memberRepository.findAll();
		ModelAndView mv = new ModelAndView("members");
		mv.addObject("members", members);
		return mv;
	}

}
```

<br>

### `@RequestMapping` 의 특징

- `@RequestMapping` 은 **클래스 단위 뿐만 아니라,** **메서드 단위에서도 적용될 수 있다.**
- 따라서, 위에서 작성한 여러 컨트롤러들을 하나의 클래스로 통합할 수 있다!

<br><br>

## `@RequestMapping` 기반의 컨트롤러 통합 - V2

### SpringMemberControllerV2

```java
import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

//RequestMapping을 클래스와 메서드에 사용하면,
//"클래스 단위 + 메서드 단위" 로 매핑하는 것을 가능하게 한다.
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {
	
	private MemberRepository memberRepository = MemberRepository.getInstance();

	// "/springmvc/v2/members/new-form"
	@RequestMapping("/new-form")
	public ModelAndView newForm() {
		return new ModelAndView("new-form");
	}

	// "/springmvc/v2/members/save"
	@RequestMapping("/save")
	public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
		String username = request.getParameter("username");
		int age = Integer.parseInt(request.getParameter("age"));
		Member member = new Member(username, age);
		memberRepository.save(member);
		ModelAndView mav = new ModelAndView("save-result");
		mav.addObject("member", member);
		return mav;
	}

	// "/springmvc/v2/members"
	@RequestMapping
	public ModelAndView members() {
		List<Member> members = memberRepository.findAll();
		ModelAndView mav = new ModelAndView("members");
		mav.addObject("members", members);
		return mav;
	}

}
```

- **조합**

    - "클래스 단위에서 쓰인 `@RequestMapping` " 과 "메서드 단위에서 쓰인 `@RequestMapping` " 은 조합이 가능하다.
- **조합 결과**
    - 클래스 레벨: `@RequestMapping("/springmvc/v2/members")`
        - 메서드 레벨: `@RequestMapping("/new-form")` ⇒ `/springmvc/v2/members/new-form`
        - 메서드 레벨: `@RequestMapping("/save")` ⇒ `/springmvc/v2/members/save`
        - 메서드 레벨: `@RequestMapping` ⇒ `/springmvc/v2/members`

결과적으로 상당히 깔끔하고 유지보수하기 쉬워졌다.

<br><br>

## `@RequestMapping` 기반의 컨트롤러를 보다 실용적으로 - V3

각각의 메서드에서 `ModelAndView` 객체에 '뷰의 논리 이름'과 'Model'을 세팅하여, 다시 일일히 반환하기가 번거롭다. 따라서 위에서 작성한 코드(V2)를 보다 실용적인 방식으로 수정해본다.

> **실무에서는 지금부터 설명하는 방식을 주로 사용한다.**

<br>

### SpringMemberControllerV3

```java
import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.util.List;

@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

	private MemberRepository memberRepository = MemberRepository.getInstance();

	// HTTP 메서드가 GET 방식인 요청 중 /springmvc/v3/members/new-form 으로 요청이 들어오면 호출됨.
	@GetMapping("/new-form")
	public String newForm() {
		return "new-form"; //뷰의 논리적 이름를 반환한다.
	}

	// HTTP 메서드가 POST 방식인 요청 중 /springmvc/v3/members/save 으로 요청이 들어오면 호출됨.
	@PostMapping("/save")
	public String save(
							@RequestParam("username") String username, //getParameter("username")
							@RequestParam("age") int age, //getParameter("age")
							Model model) {
		Member member = new Member(username, age);
		memberRepository.save(member);

		//model에 member 객체 저장(바인딩)
		model.addAttribute("member", member);

		return "save-result";
	}

	// HTTP 메서드가 GET 방식인 요청 중 /springmvc/v3/members 으로 요청이 들어오면 호출됨.
	@GetMapping
	public String members(Model model) {
		List<Member> members = memberRepository.findAll();
		model.addAttribute("members", members);
		return "members";
	}

}
```

- **Model 파라미터**

    - 위에서 작성한 메서드 `save()` , `members()` 를 보면 Model을 파라미터로 받는 것을 확인할 수 있다.
    - `model` 객체에 데이터 추가 및 조회 할 수 있다.
    - 스프링 컨테이너가 알아서 메서드에게 매개변수를 넘겨주고, 또 다시 view 단에 해당 `model` 객체를 넘겨준다.
- **ViewName 직접 반환**
    - 뷰의 논리 이름을 반환할 수 있다.
- `@RequestParam` **사용**
    - `@RequestParam("username")` 은 `request.getParameter("username")` 과 거의 같은 코드라고 생각하면 된다.
- `@RequestMapping` **을** `@GetMapping` , `@PostMapping` **으로 대체**
    - `@RequestMapping` 은 URL과 HTTP Method를 구분할 수 있다.
    - 예시) `@RequestMapping(value = "/new-form" , method = RequestMethod.GET)`
    - 이것을 `@GetMapping` 과 `@PostMapping` , `@PutMapping` , `@PatchMapping` , `@DeleteMapping` 이 간편하게 지원한다.

<br>

---

<br>

<a href="https://inf.run/RfTn"><img src="/assets/img/Inflearn_Spring_MVC1/Logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*