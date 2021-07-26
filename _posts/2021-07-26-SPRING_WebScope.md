---
category: Spring
tags: [스프링, 핵심원리, 빈스코프, 웹스코프]
title: "[스프링 - 핵심원리] 웹 스코프"
date:   2021-07-26 15:30:00 
lastmod : 2021-07-26 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

[이전 글](https://taegyunwoo.github.io/spring/SPRING_BeanScope)에서 싱글톤과 프로토타입 스코프를 학습했다. 이제 나머지 웹 스코프에 대해 알아보자.

<br><br><br>

# 웹 스코프

## 웹 스코프의 특징

- 웹 스코프는 웹 환경에서만 동작한다.
- 웹 스코프는 프로토타입과 다르게 스프링이 **해당 스코프의 종료시점까지 관리** 한다. 따라서 종료 메서드가 호출된다.

<br><br>

## 웹 스코프의 종류

- **request**
    - HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프이다.
    - 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
- **session**
    - HTTP Session과 동일한 생명주기를 가지는 스코프이다.
- **application**
    - 서블릿 컨텍스트 ( `ServletContext` )와 동일한 생명주기를 가지는 스코프이다.
- **websocket**
    - 웹 소켓과 동일한 생명주기를 가지는 스코프이다.

<br><br>

## request 스코프 예제

- 동시에 여러 HTTP 요청이 오면, 어떤 요청이 남긴 로그인지 구분하기 어렵다.
- 이때 사용할 수 있는 것이 바로, request 스코프이다.

<br>

### 로그 관련 빈

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.UUID;

@Component
@Scope(value = "request") //스코프를 request로 지정한다.
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

- `@Scope(value = "request")` 를 사용하여 request 스코프로 지정한다.
    - 해당 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에 소멸된다.

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
@RequiredArgsConstructor //자동으로 생성자를 생성해서 의존관계를 주입해준다.
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

- 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러이다.

<br>

### 비즈니스 로직 서비스

```java
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor //자동으로 생성자를 생성해서 의존관계를 주입해준다.
public class LogDemoService {
	private final MyLogger myLogger;

	public void logic(String id) {
		myLogger.log("service id = " + id);
	}

}
```

- http://localhost:8080/log-demo
    - 이제 해당 URL로 테스트해보려고 하는데...
    - 서버가 실행이 안된다. 서버 실행이 안되는 것이 정상이다!
- 만약, 이 서비스 계층에 로그를 찍는 코드가 들어간다면 유지보수 관점에서 좋지 않다. request 빈을 사용하지 않고 모든 정보를 서비스 계층으로 넘겨서 처리한다면 말이다.  
따라서, 이렇게 따로 구분을 해야한다.

서버 실행 시점에는 당연히 오류가 뜰 수 밖에 없다. 왜냐하면, request 빈은 실제 사용자의 요청이 와야만 생성되기 때문이다. 그렇기 때문에, `LogDemoService` **의 멤버변수** `myLogger` **에 주입할 빈 객체가 존재하지 않는다. (생성자 주입)** 이러한 오류를 한번 해결해보자.

<br><br>

## 스코프와 Provider

### 컨트롤러 수정

```java
import hello.core.common.MyLogger;
import hello.core.logdemo.LogDemoService;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.http.HttpServletRequest;

@Controller
@RequiredArgsConstructor
public class LogDemoController {

	private final LogDemoService logDemoService;
	private final ObjectProvider<MyLogger> myLoggerProvider; //ObjectProvider

	@RequestMapping("log-demo")
	@ResponseBody
	public String logDemo(HttpServletRequest request) {
		String requestURL = request.getRequestURL().toString();

		//이제서야 myLogger 빈을 가져온다.
		MyLogger myLogger = myLoggerProvider.getObject();

		myLogger.setRequestURL(requestURL);
		myLogger.log("controller test");
		logDemoService.logic("testId");
		return "OK";
	}

}
```

<br>

### 서비스 수정

```java
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {
	private final ObjectProvider<MyLogger> myLoggerProvider; //ObjectProvider
	
	public void logic(String id) {

		//이제서야 myLogger 빈을 가져온다.
		MyLogger myLogger = myLoggerProvider.getObject();

		myLogger.log("service id = " + id);
	}

}
```

- `ObjectProvider` 덕분에 `ObjectProvider.getObject()` 를 호출하는 시점까지 **request scope 빈의 생성(조회)를 지연할 수 있다.**
- 이제 서버실행도 정상적으로 되고, http://localhost:8080/log-demo 요청시에도 성공적으로 작동한다.

<br>

---

<br>

<a href="https://inf.run/pcN8"><img src="/assets/img/Inflearn_Spring_SpringCore/Logo.png" width="400px" height="250px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*