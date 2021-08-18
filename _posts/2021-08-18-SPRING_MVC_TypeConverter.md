---
category: Spring-MVC
tags: [스프링, MVC, Converter]
title: "[스프링 - MVC] 스프링 타입 컨버터"
date:   2021-08-17 14:00:00 
lastmod : 2021-08-17 14:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 스프링 타입 컨버터

## 개요

- 애플리케이션을 개발하다보면, 타입을 변환해야 하는 경우가 상당히 많다.
- 스프링은 "숫자→문자", "문자→숫자" 변환 등의 기초적인 타입변환을 자체적으로 지원한다.
    - `@RequestParam` , `@ModelAttribute` , `@PathVariable` 등을 통해, 요청값을 바인딩할 때
- 이런 기초적인 타입변환 대신, **개발자가 새로운 타입을 만들어서 변환하고 싶을 때 타입 컨버터를 직접 구현하면 된다.**

<br/><br/>

## 타입 컨버터

### `Converter` 인터페이스

```java
package org.springframework.core.convert.converter;
public interface Converter<S, T> {
  T convert(S source);
}
```

- 스프링에 추가적인 타입 변환이 필요하면, 해당 인터페이스를 구현해서 등록하면 된다.
- `implements Converter<원타입, 변환될_타입>`

<br/><br/>

## 컨버전 서비스

### 컨버전 서비스란?

- `Converter` 를 통해 구현한 타입 변환 클래스(컨버터)를 하나씩 찾아서 사용하기엔 불편하다.
- **스프링은 컨버전 서비스를 통해, 개별 컨버터를 모아두고 그것들을 묶어서 사용할 수 있는 기능을 제공한다.**
- **스프링은 내부에서 `ConversionService` 을 제공한다.**

    > 자세한 내용은 이후에 설명한다.

<br/>

### `ConversionService` 인터페이스

```java
public interface ConversionService {
	boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
	boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);

	<T> T convert(@Nullable Object source, Class<T> targetType);
	Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

- `canConvert()`
    - 컨버팅이 가능한지 확인하는 메서드
- `convert()`
    - 컨버팅 메서드

<br/>

### `ConversionService` 구현체: `DefaultConversionService` 클래스

- 스프링이 `ConversionService` 인터페이스를 구현한 `DefaultConversionService` 를 제공한다.

> `DefaultConversionService` 를 직접 사용하는 예시는 아래 내용 참고

<br/>

### `Converter` 인터페이스 구현과 (직접)등록 예시

- **변환 목표**
    - IpPort 객체 → 문자열
    - 문자열 → IpPort 객체

<br/>

- **`IpPort` 객체**

    ```java
    public class IpPort {
      private String ip;
      private int port;

      public IpPort(String ip, int port) {
        this.ip = ip;
        this.port = port;
      }

      public String getIp() {
        return ip;
      }

      public int getPort() {
        return port;
      }

    	/**
    	 * 테스트 코드에서 값을 비교하기 위한 메서드들
    	 */
    	@Override
      public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        IpPort ipPort = (IpPort) o;
        return port == ipPort.port && Objects.equals(ip, ipPort.ip);
      }

      @Override
      public int hashCode() {
        return Objects.hash(ip, port);
      }
    }
    ```

<br/>

- **`IpPort→String` 변환 구현: `IpPortToStringConverter` 클래스**

    ```java
    import org.springframework.core.convert.converter.Converter;

    public class IpPortToStringConverter implements Converter<IpPort, String> {

        @Override
        public String convert(IpPort source) {
            String ip = source.getIp();
            String port = String.valueOf(source.getPort());

            return ip + ":" + port;
        }

    }
    ```

<br/>

- **`String→IpPort` 변환 구현: `StringToIpPortConverter` 클래스**

    ```java
    import org.springframework.core.convert.converter.Converter;

    public class StringToIpPortConverter implements Converter<String, IpPort> {

        @Override
        public IpPort convert(String source) {
            String[] ipAndPort = source.split(":");
            IpPort ipPort = new IpPort(ipAndPort[0], Integer.parseInt(ipAndPort[1]));
            return ipPort;
        }

    }
    ```

<br/>

- **컨버전 서비스 직접 사용 테스트코드: `ConversionServiceTest` 클래스**

    ```java
    import org.assertj.core.api.Assertions;
    import org.junit.jupiter.api.Test;
    import org.springframework.core.convert.support.DefaultConversionService;

    public class ConversionServiceTest {

        @Test
        void conversionService() {
    				//스프링이 제공하는 ConversionService 구현체
            DefaultConversionService conversionService = new DefaultConversionService();

            //개발자가 직접 구현한 컨버터 등록
            conversionService.addConverter(new IpPortToStringConverter());
            conversionService.addConverter(new StringToIpPortConverter());

            Assertions.assertThat(conversionService.convert("127.0.0.1:8080", IpPort.class))
                    .isEqualTo(new IpPort("127.0.0.1", 8080));

            Assertions.assertThat(conversionService.convert(new IpPort("127.0.0.1", 80), String.class))
                    .isEqualTo("127.0.0.1:80");
        }

    }
    ```

<br/>

### 컨버전 서비스의 특징

- **타입 변환을 원하는 사용자(개발자)는 컨버전 서비스 인터페이스에만 의존하면 된다.**
    - 컨버터 등록: 타입 컨버터 클래스를 명확하게 알아야 등록할 수 있다.
    - 컨버터 사용: 어떤 타입 컨버터 클래스가 사용되는지 알 필요가 없다. 단지, "어떤 타입을 어떤 타입으로 변환할 것인가?" 에만 집중하면 된다.
    - **따라서, 사용자가 어떤 타입 컨버터 클래스가 사용되는지 몰라도 타입변환을 할 수 있도록, 컨버전 서비스 인터페이스가 지원한다.**

<br/>

- **인터페이스 분리 원리 - ISP (Interface Segregation Principal)**
    - `DefaultConversionService` 는 다음 두 인터페이스를 구현했다.
        - `ConversionService` 인터페이스: 컨버터 사용에 초점
        - `ConversionRegistry` 인터페이스: 컨버터 등록에 초점
    - 이렇게 인터페이스를 분리하면 컨버터를 사용하는 클라이언트와 컨버터를 등록하고 관리하는 클라이언트의 관심사를 명확하게 분리할 수 있다.
    - **특히, 컨버터를 사용하는 입장에선 `ConversionService` 인터페이스만 의존하면 되므로, 컨버터를 어떻게 등록하고 관리하는지는 전혀 몰라도 된다.**

<br/><br/>

## 스프링에 Converter 적용하기

### 컨버터 등록: `WebConfig` 클래스

```java
import ...

@Configuration
public class WebConfig implements WebMvcConfigurer {

  /**
   * 컨버터 등록
   */
  @Override
  public void addFormatters(FormatterRegistry registry) {
    registry.addConverter(new IpPortToStringConverter());
    registry.addConverter(new StringToIpPortConverter());
  }

}
```

- **스프링은 내부에서 `ConversionService` 을 제공하므로, `WebMvcConfigurer` 가 제공하는 `addFormatters()` 를 사용해서 추가하고 싶은 컨버터를 등록하면 된다.**
    - 이렇게하면 스프링은 내부에서 사용하는 `ConversionService` 에 컨버터를 추가해준다.

<br/>

### 컨트롤러: `ConversionController` 클래스

```java
import ...

@RestController
public class ConversionController {

    @GetMapping("/StringToIpPort")
    public IpPort StringToIpPort(@RequestParam IpPort ipPort) {
        return ipPort;
    }

}
```

### 요청 결과

- **요청**

    ![Untitled](/assets/img/2021-08-18-SPRING_MVC_TypeConverter/Untitled%2030.png)

<br/>

- **결과**

    ![Untitled](/assets/img/2021-08-18-SPRING_MVC_TypeConverter/Untitled%2031.png)

    - 쿼리 스트링 `ipPort=127.0.0.1:66` 이 IpPort 타입으로 잘 변환되었다.

<br/>

### 처리 과정

- `@RequestParam`
    - `ArgumentResolver` 중 해당 애너테이션을 처리하는 `RequestParamMethodArgumentResolver` 가 처리한다.
- `RequestParamMethodArgumentResolver` 에서 `ConversionService` 를 사용해서 타입을 변환한다.

<br/><br/>

## 뷰 템플릿에 컨버터 적용

- 타임리프에서 `${{..}}` 을 사용하면, 컨버터를 사용할 수 있다.
- 예시 코드로 알아보자.

<br/>

### 뷰 호출 컨트롤러: `ConverterController` 클래스

```java
import ...

@Controller
public class ConversionController {

  @GetMapping("/converterView")
  public String converterView(Model model) {
    model.addAttribute("ipPort", new IpPort("127.0.0.1", 66));
    return "converter";
  }

}
```

<br/>

### 뷰 템플릿: `converter.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
		<!-- ${{...}} 문법 적용 -->
    <p th:text="${{ipPort}}">ipPort</p>
</body>
</html>
```

<br/>

### 호출결과

![Untitled](/assets/img/2021-08-18-SPRING_MVC_TypeConverter/Untitled%2032.png)

- `IpPort` 객체를 `String` 으로 변환해야하므로, **`IpPortToStringConverter` 컨버터가 적용되었다.**

<br/><br/>

## 뷰 템플릿 폼에 컨버터 적용

- 타임리프 폼에서 `th:field` 을 사용하면, 컨버터를 사용할 수 있다.
- 예시 코드로 알아보자.

<br/>

### 뷰 호출 컨트롤러: `ConverterController` 클래스

```java
import ...

@Controller
public class ConversionController {

	@GetMapping("/edit")
  public String converterForm(Model model) {
    model.addAttribute("form", new Form(new IpPort("127.0.0.1", 66)));
    return "converterForm";
  }

  @PostMapping("/edit")
  public String converterEdit(@ModelAttribute Form form, Model model) {
    model.addAttribute(form.getIpPort());
    return "converter";
  }

  @Data
  static class Form {
    private IpPort ipPort;

    public Form(IpPort ipPort) {
      this.ipPort = ipPort;
    }
  }

}
```

<br/>

### 폼 뷰 템플릿: `converterForm.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <form th:object="${form}" method="post">
    th:field = <input type="text" th:field="*{ipPort}"/>
    th:value = <input type="text" th:value="*{ipPort}"/>
  </form>
</body>
</html>
```

<br/>

### 호출결과

![Untitled](/assets/img/2021-08-18-SPRING_MVC_TypeConverter/Untitled%2033.png)

- `th:field="*{ipPort}"`
    - `form` 객체의 필드 `ipPort` 에 `IpPortToStringConverter` 컨버터를 적용하여 출력한다.
- `th:value="*{ipPort}"`
    - `form` 객체의 필드 `ipPort` 의 참조값을 그대로 출력한다. (컨버터 적용 X)

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*