---
category: Spring
tags: [스프링, 핵심원리, 빈스코프, 웹스코프, 프록시]
title: "[스프링 - 핵심원리] 스코프와 프록시"
date:   2021-07-26 17:30:00 
lastmod : 2021-07-26 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

[이전 글](https://taegyunwoo.github.io/spring/SPRING_WebScope)에서 request 스코프를 다뤘다. `ObjectProvider` 를 통해 request 빈의 조회를 지연시켰다. 하지만 보다 더 간편한 방법이 있다. 그것이 바로 프록시를 활용하는 것이다.

<br><br><br>

# 스코프와 프록시

- `@Scope(value = "request" , proxyMode = ScopedProxyMode.TARGET_CLASS)`
    - 해당 코드를 통해 프록시를 적용할 수 있다.
    - 적용 대상이 클래스라면, `proxyMode = ScopedProxyMode.TARGET_CLASS`
    - 적용 대상이 인터페이스라면, `proxyMode = ScopedProxyMode.TARGET_INTERFACES`
- **이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들어두고, HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.**

<br><br>

## 예시 코드

> 이전 글에서 `ObjectProvider` 를 제외하고 작성한 부분과 동일하다. (로그 관련 빈 제외)

<br>

### 로그 관련 빈

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.UUID;

@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) //프록시를 사용한다.
public class MyLogger {

	private String uuid;
	private String requestURL;

	public void setRequestURL(String requestURL) {
		this.requestURL = requestURL;
	}

	public void log(String message) {
		System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
	}

	@PostConstruct
	public void init() {
		uuid = UUID.randomUUID().toString();
		System.out.println("[" + uuid + "] request scope bean create:" + this);
	}

	@PreDestroy
	public void close() {
		System.out.println("[" + uuid + "] request scope bean close:" + this);
	}

}
```

<br>

### 로그 테스트 컨트롤러

```java
import hello.core.common.MyLogger;
import hello.core.logdemo.LogDemoService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.http.HttpServletRequest;

@Controller
@RequiredArgsConstructor
public class LogDemoController {
	private final LogDemoService logDemoService;
	private final MyLogger myLogger;

	@RequestMapping("log-demo")
	@ResponseBody
	public String logDemo(HttpServletRequest request) {
		String requestURL = request.getRequestURL().toString();
		myLogger.setRequestURL(requestURL);
		myLogger.log("controller test");
		logDemoService.logic("testId");
		return "OK";
	}

}
```

<br>

### 비즈니스 로직 서비스

```java
package hello.core.logdemo;
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {
	private final MyLogger myLogger;
	
	public void logic(String id) {
		myLogger.log("service id = " + id);
	}
 
}
```

- `ObjectProvider` 를 사용하지 않았는데, 정상적으로 작동한다!
- 프록시의 정확한 원리를 알아보자.

<br><br>

## 프록시

- **CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 객체 (프록시 객체)를 만들어서 주입한다.**
    - 위 예시
        - 프록시 객체 = '로그 관련 빈'( `MyLogger` )을 상속받은 CGLIB 객체
        - 프록시 객체를 주입받는 빈 = '로그 테스트 컨트롤러'( `LogDemoController` ) , '비즈니스 로직 서비스'(`LogDemoService`)
- 스프링 컨테이너에 "myLogger"라는 이름으로 진짜(`MyLogger`) 대신에 프록시 객체가 등록된다.
    - 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입된다.
- **가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.**
    - 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다.
    - 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다. (다형성)

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*