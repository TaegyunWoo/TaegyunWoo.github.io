---
category: Tech
tags: [Tech]
title: "Spring Boot 2.4 이상에서 설정파일을 다루는 방법"
date:   2022-08-07 15:30:00 
lastmod : 2022-08-07 15:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# Spring Boot 2.4 이상에서 설정파일을 다루는 방법

## 개요

- Spring Boot 2.4 부터 설정파일 (`application.properties` , `application.yml`) 를 다루는 방식이 변경되었다.
- 그 이유는 쿠버네티스 때문이다. 쿠버네티스의 볼륨 마운트 구성을 지원하기 위해 변경된 것인데, 자세한 내용은 아래 Spring.io 블로그를 참고하자.
    - [https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)
- 우리는 변경된 사항에 대해 자세히 알아보도록 한다.

<br/>

## Document Order

- 하나에 `yml` 파일에서 여러 프로필을 작성한다면 (즉, 다중 프로필 yml 이라면) 하단 문서가 상단 문서의 값을 덮어씌운다.

```yaml
# 상단 문서
test: "value"
myTest: "value"
---
# 하단 문서
test: "overridden-value" # 위 test의 값을 덮어씌운다. `myTest`는 null이 된다.
```

<br/>

## Multi-document Properties Files

- 기존에는 `.properties` 파일 하나에 여러 프로필을 설정할 수 없었고, 파일마다 관리를 했어야 했다.
    - `application.properties` , `application-dev.properties` 등…
- 이제는 `.yml` 파일처럼 하나의 `.properties` 파일에서 여러 프로필을 관리할 수 있다.
- 그리고 `.yml` 과 마찬가지로 Document Order 도 가능하다.

```text
# 상단 문서
test=value
#---
# 하단 문서
test=overridden-value # 위 test의 값을 덮어씌운다.
```

<br/>

## Profile Specific Documents

- 기존에는 `spring.profiles` 를 통해, 프로필을 활성화했다.
- 이제는 `spring.config.active.on-profile` 을 사용하면 된다.

```text
test=value
#---
spring.config.activate.on-profile=dev # 만약 dev라는 이름을 갖는 프로필이 활성화되면 해당 문서를 함께 활성화시킨다.
test=overridden-value
```

<br/>

## Profile Activation

- 기존의 `spring.profiles.active` 를 사용해서 프로필을 활성화하거나 포함시키는 작업 역시 그대로 할 수 있다.
- **하지만 `spring.config.active.on-profile` 과 함께 사용하는 것은 불가능하다.**

```text
test=value
#---
spring.config.activate.on-profile=dev # 본 문서를 dev에 추가한다.
spring.profiles.active=local # 본 문서가 활성화되면 local 도 활성화시키도록 하려했지만, 실패한다.
test=overridden value
```

- 이렇게 제한을 걸어둔 이유는 설정 자체가 너무 복잡해질 수 있기 때문에 그렇다고 한다.
- 그렇다면 하나의 설정 파일에 하위 설정 파일을 어떻게 지정할까?
    - 일일이 하위 설정 파일마다 `spring.config.activate.on-profile` 을 작성해주거나, 부모 설정 파일에 `spring.profiles.active` 를 작성하고 모든 하위 설정 파일을 명시해주어야 할까?
    - 그 해답은 `Profile Groups` 이다. 이를 통해, 좀 더 보기 좋게 작성할 수 있다.

<br/>

## Profile Groups

- Profile Groups 는 단일 프로필을 여러 하위 프로필로 확장할 수 있는 새로운 기능이다.
- Profile Groups 를 사용하면, 여러 하위 프로필을 한눈에 확인할 수 있다.
- 또한, 다른 프로필로 바꾸기 위해 일일이 필요한 프로필을 지정하지 않아도 된다.
    - 기존에는 배포 시 필요한 프로필 파일이 10개라고 하면, `spring.profiles.active` 에 10개의 프로필 이름을 작성해야 했다.
    - 이때 만약 실수로 9개의 프로필만 작성한다면? ⇒ 상당히 킹받을 것이다.
    - Profile Groups 를 통해 간단히 여러 프로필을 묶을 수 있고, 개발자가 실수할 가능성을 줄일 수 있다.

```text
# 만약 prod 프로필을 활성화시킬때,
# proddb, prodmq, prodmetrics 모두 활성화시켜야 한다면 아래와 같이 작성하면 된다. 

spring.profiles.group.prod=proddb,prodmq,prodmetrics
```

- 이렇게 작성하면, 아래와 같이 동작시킬 수 있다.

```text
# 개발 환경을 사용한다면
spring.profiles.active=dev

# 프로덕트 환경을 사용한다면
spring.profiles.active=prod

# 그룹 설정
spring.profiles.group.dev=devdb,devdata
spring.profiles.group.prod=proddb,prodmq,prodmetrics
```

<br/>

## Importing Additional Configuration

- 추가적인 속성 파일을 가져올 수 있다.
    - **`application-{profile}.properties` 나 `application-{profile}.yml` 형식을 갖는 프로필이 아닌, 말 그대로 속성 파일 (변수 등이 담긴) 을 가져올 때 쓴다.**

```text
application.name=myapp
spring.config.import=developer.properties
```

- 아래는 `application-prod.properties` 가 활성화되었을 때, `prod.properties` 속성 파일을 가져오는 예시이다.

```text
spring.config.activate.on-profile=prod
spring.config.import=prod.properties
```

<br/>

## Volume Mounted Configuration Trees

- URL 속성을 가져온다.
- 접두사가 없으면 일반 파일 또는 폴더로 간주한다.
- `configtree:`
    - 해당 접두사를 사용하는 경우, 해당 위치에 Kubernetes 스타일의 볼륨 마운트 구성 트리가 있어야 한다는 것을 Spring Boot에게 알려준다.

```text
spring.config.import=configtree:/etc/config
```

<br/>

## Cloud Platform Activation

- 특정 클라우드 플랫폼에서 볼륨 마운트 구성 트리(또는 해당 항목의 속성)만 활성화한다.

```text
spring.config.activate.on-cloud-platform=kubernetes
spring.config.import=configtree:/etc/config
```

<br/>

## Using Legacy Processing

- 만약 이전 버전의 설정 방식을 사용하고 싶다면, 아래와 같이 설정하면 된다.

```
spring.config.use-legacy-processing=true
```

<br/><br/>

## 참고 Reference

- [https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)
- [https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.application-properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.application-properties)
- [https://data-make.tistory.com/722](https://data-make.tistory.com/722)