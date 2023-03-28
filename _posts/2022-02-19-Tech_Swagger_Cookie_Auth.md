---
category: Tech
tags: [Tech]
title: "Swagger에서 jsessionid 인증·인가 사용하기"
date:   2022-02-19 20:10:00 
lastmod : 2022-02-19 20:10:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# Swagger에서의 jsessionid 인증 사용

## 개요

- Swagger 를 통해 RestAPI 문서를 작성할 때, 인가(Authorization)가 필요한 API를 작성하는 경우가 있다.
- 로그인된 사용자만 접근할 수 있는 Endpoint를 Swagger에서 테스팅하려면 어떻게 해야할까?
- 본 글에선 Swagger에서 인가가 필요한 API에 접근하기 위해, 인증 및 인가 처리를 하는 방법에 대해 설명한다.
    - 인증·인가 방식에는 여러 종류가 있지만, 그 중 쿠키 기반 (`JSESSIONID`) 방식에 대해 다룬다.
- **즉 Swagger에서 쿠키 기반 인가 방식을 사용하는 방법에 대해 알아보자.**

<br/><br/>

## 코드

### 의존성 추가: `build.gradle` 파일

```text
implementation 'org.springdoc:spring-boot-springdoc:3.1.2'
```

> Swagger를 사용하려면 기본적으로 추가해야 한다.

<br/>

### Swagger 설정: `SwaggerConfig.java`

```java
import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Component
public class SwaggerConfig {

  @Bean
  public OpenAPI api() {
    Info info = new Info()
      .title("제목")
      .version("V1.0")
      .contact(new Contact()
              .name("Web Site")
              .url("배포 주소"))
      .license(new License()
              .name("Apache License Version 2.0")
              .url("http://www.apache.org/license/LICENSE-2.0"));

		//-------------------- 인가 방식 지정 ---------------------
    SecurityScheme auth = new SecurityScheme()
      .type(SecurityScheme.Type.APIKEY).in(SecurityScheme.In.COOKIE).name("JSESSIONID");
    SecurityRequirement securityRequirement = new SecurityRequirement().addList("basicAuth");

    return new OpenAPI()
      .components(new Components().addSecuritySchemes("basicAuth", auth))
      .addSecurityItem(securityRequirement)
      .info(info);
  }

}
```

<br/>

- 상세 설명
    - 쿠키 기반 인가 방식
        - 클라이언트가 인증(Authentication, 로그인)에 성공하면, 서버가 HTTP 응답 MSG의 헤더에 `Set-Cookie: JSESSIONID=값` 을 담아서 보낸다.
        - 클라이언트는 서버가 보낸 `JSESSIONID` 값을 쿠키에 저장한다.
        - 클라이언트가 HTTP 요청 MSG를 보낼 때, 헤더부분에 `Cookie: JSESSIONID=값` 를 함께 보낸다.
        - 그리고 서버가 해당 `JSESSIONID` 값을 확인하여, 인증된 사용자인지 확인(인가)한다.
        - 위와 같은 방식을 `APIKEY` 방식이라고 한다.
        - 자세한 것은 [공식문서](https://swagger.io/docs/specification/authentication/api-keys/) 를 참고하자.
    - `.type(SecurityScheme.Type.APIKEY)`
        - 우리는 현재 쿠키 기반 인가 방식을 사용하므로, `APIKEY` 타입으로 지정해야 한다.
    - `.in(SecurityScheme.In.COOKIE)`
        - `APIKEY` 값이 쿠키에 담겨있으므로, `COOKIE` 로 지정한다.
    - `.name("JSESSIONID")`
        - 인가에 사용되는 키(Key)가 `JSESSIONID` 이므로, name을 `JSESSIONID` 로 지정한다.

<br/><br/>

# Reference

- [https://swagger.io/docs/specification/authentication/cookie-authentication/](https://swagger.io/docs/specification/authentication/cookie-authentication/)
- [https://swagger.io/docs/specification/authentication/api-keys/](https://swagger.io/docs/specification/authentication/api-keys/)
- [https://prodo-developer.tistory.com/50](https://prodo-developer.tistory.com/50)