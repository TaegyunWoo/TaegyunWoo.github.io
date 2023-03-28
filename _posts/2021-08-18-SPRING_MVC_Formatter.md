---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 포맷터(Formatter)"
date:   2021-08-18 14:30:00 
lastmod : 2021-08-18 14:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
    1. [[스프링 - MVC] 스프링 타입 컨버터](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_TypeConverter)

<br/><br/>

# 포맷터

## 개요

- 이전 게시글에서 스프링이 제공하는 `Converter` 인터페이스를 구현하여 타입을 변환하는 방법에 대해 알아보았다.
- `Converter` 는 입력과 출력 타입에 제한이 없는 범용 타입 변환 기능을 제공한다.
- 하지만, 대부분의 상황에서 필요한 것은 **"문자→다른타입" 이나 "다른타입→문자" 로 변환하는 것이다.**
- **포맷터는 객체를 특정한 포멧에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 특화된 기능이다.**

> 포맷터는 컨버터의 특별한 버전으로 이해하면 된다.

<br/>

### Converter VS Formatter

- `Converter`
    - 범용 타입 변환
    - 객체 → 객체
- `Formatter`
    - 문자에 특화
    - 객체 → 문자
    - 문자 → 객체
    - 현지화(Locale)

<br/><br/>

## 포맷터

- 일반 컨버전 서비스(`ConversionService`)는 **컨버터만 등록**할 수 있고, 포맷터를 등록할 수 는 없다.
- **따라서 포맷터를 지원하는 컨버전 서비스를 사용해야 포맷터를 등록할 수 있다.**
    - [이전 게시글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_TypeConverter#7)에서 살펴본, `DefaultConversionService` 구현체는 포맷터를 등록할 수 없다.
    - `FormattingConversionService` 의 구현체인 `DefaultFormattingConversionService` 를 통해 포맷터를 등록할 수 있다.
    - `FormattingConversionService` 은 `ConversionService` 를 상속받았기 때문에, 컨버터 역시 등록할 수 있다.

<br/><br/>

## 포맷터 구현

### `Formatter` 인터페이스

```java
public interface Printer<T> {
	String print(T object, Locale locale);
}

public interface Parser<T> {
	T parse(String text, Locale locale) throws ParseException;
}

public interface Formatter<T> extends Printer<T>, Parser<T> { }
```

- `String print(T object, Locale locale)`
    - 객체를 문자로 변경한다.
- `T parse(String text, Locale locale)`
    - 문자를 객체로 변경한다.

<br/>

### 포맷터 구현: `MyNumberFormatter` 클래스

```java
import org.springframework.format.Formatter;
import java.text.NumberFormat;
import java.text.ParseException;
import java.util.Locale;

public class MyNumberFormatter implements Formatter<Number> {

  //Number는 Integer, Long, Double 등의 부모 클래스이다.
  
  /**
   * 문자를 객체(Number)로 변환
   * 쉼표가 있는 숫자 문자열을 Number 객체로 변환
   */
  @Override
  public Number parse(String text, Locale locale) throws ParseException {
    NumberFormat format = NumberFormat.getInstance(locale);
    return format.parse(text);
  }

  /**
   * 객체(Number)를 문자로 변환
   * Number 객체를 쉼표가 적용된 문자열로 변환
   */
  @Override
  public String print(Number object, Locale locale) {
    return NumberFormat.getInstance(locale).format(object);
  }
}
```

- `NumberFormat`
    - "1,000" 처럼 숫자 중간의 쉼표를 적용하려면, 자바가 기본으로 제공하는 `NumberFormat` 객체를 사용하면 된다.
    - 이 객체는 `Locale` 정보를 활용해서 나라별로 다른 숫자 포맷을 만들어준다.

- `parse()`
    - 문자를 객체(숫자)로 변환한다.
    - 참고로 `Number` 타입은 `Integer, Long` 과 같은 숫자 타입의 부모 클래스이다.

- `print()`
    - 객체(숫자)를 문자로 변환한다.

<br/>

### 포맷터 등록: `WebConfig` 클래스

```java
package example.exception;

import ...

@Configuration
public class WebConfig implements WebMvcConfigurer {

    /**
     * 컨버터 등록
     */
    @Override
    public void addFormatters(FormatterRegistry registry) {
        //-------- 컨버터 등록(이전 게시글에서 작성했던 코드) --------
        registry.addConverter(new IpPortToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
        //--------------------------------------------------------
        
        //-------- 포맷터 등록 --------
        registry.addFormatter(new MyNumberFormatter());
    }

}
```

<br/>

### 컨트롤러: `FormatterController` 클래스

```java
import ...

@Controller
public class FormatterController {

    /**
     * 쉼표가 포함된 String을 일반 숫자로
     */
    @ResponseBody
    @GetMapping("/toNum")
    public Number toNum(@RequestParam Number number) {
        return number;
    }

    /**
     * 일반 숫자를 쉼표가 포함된 String으로
     */
    @GetMapping("/toStr")
    public String toStr(Model model) {
        model.addAttribute("number", 100000L);
        return "toStr";
    }

}
```

<br/>

### 뷰 템플릿: `toStr.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p th:text="${ {number} }"></p>
</body>
</html>
```

<br/>

### 요청 결과: `/toNum`

- **결과**

    ![Untitled](/assets/img/2021-08-18-SPRING_MVC_Formatter/Untitled%2034.png)

- **설명**
    - 쿼리 스트링으로 넘긴 값 `10,000` 을 정상적으로 숫자로 변환하여 출력하였다.

<br/>

### 요청 결과: `/toStr`

- **결과**

    ![Untitled](/assets/img/2021-08-18-SPRING_MVC_Formatter/Untitled%2035.png)

- **설명**
    - `100000` 을 정상적으로 문자로 변환하여 출력하였다.

<br/><br/>

## 스프링이 제공하는 기본 포맷터

- 스프링은 자바에서 기본으로 제공하는 타입들에 대해 수 많은 포맷터를 기본으로 제공한다.
- **스프링은 애노테이션 기반으로 원하는 형식을 지정해서 사용할 수 있는 두 가지 포맷터를 기본으로 제공한다.**

<br/>

### 애너테이션과 기본 포맷터 종류

- `@NumberFormat`
    - 숫자 관련 형식 지정 포맷터를 사용하는 애너테이션
    - 사용되는 포맷터: `NumberFormatAnnotationFormatterFactory`
- `@DateTimeFormat`
    - 날짜 관련 형식 지정 포맷터를 사용하는 애너테이션
    - 사용되는 포맷터: `Jsr310DateTimeFormatAnnotationFormatterFactory`

<br/>

### 사용 예시

- **컨트롤러: `FormatterController` 클래스**

    ```java
    import ...

    @Controller
    public class FormatterController {

        @GetMapping("/formatter")
        public String formatterForm(Model model) {
            Form form = new Form();
            form.setNumber(10000);
            form.setLocalDateTime(LocalDateTime.now());

            model.addAttribute(form);
            return "formatter-form";
        }

        @Data
        static class Form {
            @NumberFormat(pattern = "###,###")
            private Integer number;

            @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
            private LocalDateTime localDateTime;
        }
    }
    ```

<br/>

- **뷰 템플릿**

    ```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
      <form th:object="${form}" method="post">
        number = <input type="text" th:field="*{number}">
        localDateTime = <input type="text" th:field="*{localDateTime}">
      </form>
    </body>
    </html>
    ```

    > 폼이 아닌 뷰에서 출력할 때는 `${ {...} }` 을 사용하면 된다.

<br/>

- **결과**

    ![Untitled](/assets/img/2021-08-18-SPRING_MVC_Formatter/Untitled%2036.png)

<br/><br/>

## HTTP 메시지 컨버터와 컨버전 서비스

- HTTP 메시지 컨버터 ([이전 게시글 참고](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_HTTPMessageConverter)) 에는 컨버전 서비스가 적용되지 않는다.
- HTTP 메시지 컨버터와 컨버전 서비스는 전혀 관계가 없다.
- **HTTP 메시지 컨버터의 역할**
    - HTTP 메시지 바디의 내용을 객체로 변환
    - 객체를 HTTP 메시지 바디에 입력
- **컨버전 서비스의 역할**
    - `@RequestParam` , `@ModelAttribute` , `@PathVariable` , 뷰 템플릿 등에서 타입 변환

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*