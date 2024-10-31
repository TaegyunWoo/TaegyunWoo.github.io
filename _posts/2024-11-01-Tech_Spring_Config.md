---
category: Tech
tags: [Tech, Spring]
title: "Spring 설정파일의 속성값 다루기 with ConfigurationProperties Anno."
date: 2024-11-01 08:10:00 +0900
lastmod: 2024-11-01 08:10:00 +0900
sitemap:
  changefreq: daily
  priority: 1.0
---

올해 1월부터 첫출근을 하며 업무 적응하느라 바쁘게 지냈네요. 거의 반년 넘게 포스팅을 쉬었는데, 오랜만에 다시 블로그를 시작하려 합니다.

그 첫번째 주제는 스프링에서 properties 값을 다루는 여러 방법들입니다.

회사에서 코드리뷰를 받으며 속성값을 다양한 방법으로 다룰 수 있다는 것에 대해 알게 되었고, 이를 기록하려고 합니다.

그럼 본격적으로 시작해볼까요?

<br/>

## `@Value` 를 활용한 속성값 설정

여러분은 아래와 같은 properties 파일에 설정한 값을 어떻게 스프링 런타임 환경에 들고 오나요?

```yaml
# application.yml

my:
  example:
    properties:
      my-property-a: my_value_a
      my-property-b: my_value_b
```

먼저 가장 기본적인 방법인 `@Value` 에 대해 알아봅시다.

### `@Value` 활용

가장 간단한 방법입니다.

```java
@Getter
@Configuration
public class MyProperties1 {
    @Value("${my.example.properties.my-property-a}")
    private String myPropertyA;
    @Value("${my.example.properties.my-property-b}")
    private String myPropertyB;
}
```

위처럼 `@Value` 애너테이션을 사용해서 `application.yml` 에 설정한 속성값을 가져올 수 있습니다.

테스트를 해보면…

```java
@ContextConfiguration(classes = {MyProperties1.class})
@SpringBootTest
class MyTest {
    @Autowired
    private MyProperties1 myProperties1;

    @DisplayName("@Value 테스트")
    @Test
    public void valueAnnoTest() {
        System.out.println("[@Value 애너테이션 활용]");
        System.out.println("my-property-a: " + myProperties1.getMyPropertyA());
        System.out.println("my-property-b: " + myProperties1.getMyPropertyB());
    }
}
```

```
[@Value 애너테이션 활용]
my-property-a: my_value_a
my-property-b: my_value_b
```

성공적으로 가져오는 것을 확인해볼 수 있습니다.

### `@Value` 의 문제점

위처럼 `@Value` 으로 속성값을 가져와 사용하는 것에  큰 문제는 없습니다. 하지만 약간의 이상한 점이 있지 않나요?

속성값은 보통 **고정된 값**으로 properties 파일에 기록되어 있습니다. 즉, **속성값이 런타임 도중에 변경될 일은 거의 없습니다.** 그렇다면, 해당 값을 사용할 때는 읽기만 가능하고 수정은 불가능해야하지 않을까요?

아래 코드를 다시 보겠습니다.

```java
@Getter
@Configuration
public class MyProperties1 {
    @Value("${my.example.properties.my-property-a}")
    private String myPropertyA;
    @Value("${my.example.properties.my-property-b}")
    private String myPropertyB;
}
```

수정이 불가능하려면 각 필드들이 `final` 로 선언되어야 할 것 같습니다. 그래서 아래처럼 수정해봤습니다.

```java
@Getter
@RequiredArgsConstructor //생성자 추가
@Configuration
public class MyProperties1 {
    @Value("${my.example.properties.my-property-a}")
    private final String myPropertyA; //final 설정
    @Value("${my.example.properties.my-property-b}")
    private final String myPropertyB; //final 설정
}
```

그리고 설정값이 적절히 바인딩되는지 확인해보면, 아래와 같이 Bean 객체를 찾을 수 없다는 예외가 발생하게 됩니다.

```
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException
```

이는 `MyProperties1` 생성자의 파라미터 `String myPropertyA` 와 `String myPropertyB` 로 전달할 Bean 객체가 없기에 발생하는 것입니다. (스프링에서 `String` 타입의 빈을 등록하지 않았으니 당연한 결과죠?)

필드를 `final` 로 선언하고 생성자를 만들어도, **스프링에서는 Bean 객체를 주입받으려고 할 뿐 프로퍼티 값을 세팅하지는 않습니다.**

**즉, 스프링에서 properties 파일의 값을 생성자로 바인딩시키는 것은 아니라고 판단할 수 있습니다.**

속성값이 필드에 바인딩되는 과정을 간략하게나마 알아보면 아래와 같습니다.

1. 스프링 애플리케이션 실행 및 컨텍스트 초기화
2. 프로퍼티 읽기: `PropertySourcesPlaceholderConfigurer`가 `application.properties` 등의 설정 파일을 읽고 프로퍼티 소스를 등록합니다.
3. Bean 객체 생성: `@Configuration` , `@Component` 등이 붙은 클래스에 대해 객체를 생성합니다.
4. 속성값 주입: `AutowiredAnnotationBeanPostProcessor` 이 생성된 Bean 객체마다 실제 값을 찾아 주입합니다. (내부적으로 리플렉션을 사용하기에 setter 메서드가 없어도 됩니다.)

([https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Value.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Value.html))

**정리하자면, 먼저 Bean 객체를 생성한 뒤에 속성값을 바인딩하기에 (생성자로 `final` 필드를 초기화하지 않기 때문에), 속성값을 `final` 키워드로 immutable 하게 관리할 수 없습니다.**

> 또다른 문제는, 동일한 계층에 존재하는 프로퍼티를 가져올 때, 반복적으로 prefix 를 입력해야 합니다. (위 경우, `my.example.properties` 가 반복됨.)
> 

그리고 이러한 문제를 `@ConfigurationProperties` 로 해결할 수 있습니다!

<br/>

## `@ConfigurationProperties` 를 활용한 속성값 설정

바로 코드를 보겠습니다.

```java
@Getter
@RequiredArgsConstructor
@ConfigurationProperties("my.example.properties")
public class MyProperties2 {
    private final String myPropertyA;
    private final String myPropertyB;
}

@Configuration
@EnableConfigurationProperties(MyProperties2.class) //프로퍼티 클래스 활성화
public class MyConfig {
}
```

`@ConfigurationProperties("my.example.properties")` 으로 중복되는 prefix `my.example.properties` 를 클래스 레벨에서 설정했습니다. 그리고 별도의 config 클래스에서 `@EnableConfigurationProperties` 를 사용해 활성화시켰습니다.

각 필드의 이름을 기준으로 값이 설정되게 됩니다. 이때, 필드명을 properties의 네이밍 전략으로 적절히 변환하여 필드로 매핑시켜줍니다. (`myPropertyA` → `my-property-a`)
따라서 각 필드레벨에 추가적인 설정을 할 필요가 없습니다.

또한, `MyProperties2` 에는 `@Configuration` 이나 `@Component` 등의 애너테이션이 없기 때문에 단독으로는 Bean 등록이 되지 않고, `@EnableConfigurationProperties` 에 의해 Bean 객체로 등록된다는 것에 유의합시다.

이를 테스트해보면 정상적으로 값을 가져오는 것을 확인해볼 수 있습니다.

```java
@ContextConfiguration(classes = {MyConfig.class})
@SpringBootTest
class MyTest {
    @Autowired
    private MyProperties2 myProperties2;

    @DisplayName("@ConfigurationProperties 테스트")
    @Test
    public void configPropertiesAnnoTest() {
        System.out.println("[@ConfigurationProperties 애너테이션 활용]");
        System.out.println("my-property-a: " + myProperties2.getMyPropertyA());
        System.out.println("my-property-b: " + myProperties2.getMyPropertyB());
    }
}
```

```
[@ConfigurationProperties 애너테이션 활용]
my-property-a: my_value_a
my-property-b: my_value_b
```

`@ConfigurationProperties` 를 사용하여 아래와 같은 문제 해결이 가능하다고 정리할 수 있습니다.

- `@Value` 사용시, `final` 필드로 선언할 수 없어 불변성을 지킬 수 없었던 문제
- `@Value` 사용시, 반복적으로 등장하는 prefix의 중복

### 참고) setter 로 프로퍼티 바인딩

`@ConfigurationProperties` 이 설정된 클래스의 필드를 초기화할 수 있는 방법은 생성자뿐만이 아닙니다. 아래처럼 setter 메서드를 활용할수도 있습니다. (다만 이때는 `final` 필드로 선언하는 것은 불가능하겠죠?)

아래 코드를 다시 볼까요?

```java
@Getter
@Setter
@ConfigurationProperties("my.example.properties")
public class MyProperties2 {
    private String myPropertyA;
    private String myPropertyB;
}

@Configuration
@EnableConfigurationProperties(MyProperties2.class) //프로퍼티 클래스 활성화
public class MyConfig {
}
```

이처럼 setter로 설정하는 것도 가능은 합니다만… 개인적으로는 불변하게 관리하는 것을 권장드립니다! 단순히 이렇게 쓸수도 있구나~ 하고 참고하시면 좋을 것 같습니다.

### `@ConstructorBinding` 애너테이션

우리가 생성자를 통해서 `final` 로 필드를 선언하면 반드시 생성자가 필요한데, 만약 여러 생성자가 선언되어 있다면 반드시 `@ConstructorBinding` 설정을 해야 합니다. 이전에 살펴본 코드를 다시 가져오겠습니다.

```java
@Getter
@RequiredArgsConstructor
@ConfigurationProperties("my.example.properties")
public class MyProperties2 {
    private final String myPropertyA;
    private final String myPropertyB;
}
```

이처럼 하나의 생성자만 있는 경우는 `@ConstructorBinding` 이 필요 없습니다.

하지만 여러 생성자가 있다면 아래처럼 작성해야 합니다.

```java
@Getter
@ConfigurationProperties("my.example.properties")
public class MyProperties2 {
    private final String myPropertyA;
    private final String myPropertyB;

    public MyProperties2() {
        this.myPropertyA = "my init value a";
        this.myPropertyB = "my init value b";
    }

    @ConstructorBinding
    public MyProperties2(String myPropertyA, String myPropertyB) {
        this.myPropertyA = myPropertyA;
        this.myPropertyB = myPropertyB;
    }

}
```

> 스프링 2 버전인 경우, `@ConstructorBinding` 를 반드시 클래스 레벨에 `@ConfigurationProperties` 와 함께 설정해야 했습니다. 스프링 3 버전부터, 생성자가 하나라면 `@ConstructorBinding` 을 생략해도 되도록 변경되었습니다. 이에 따라 더이상 클래스 레벨에는 설정할 수 없으며, 컴파일 에러가 발생합니다. 이는 “생성자가 하나라면, 생성자에 `@Autowired` 를 붙이지 않아도 생성자 주입시키는 것”과 비슷하게 반영된 것입니다.

<br/>

## 마치며…

Spring 에서 프로퍼티 값을 관리하는 방법들에 대해 알아보았습니다. 지금까지의 내용을 정리해보면 아래와 같습니다.

- Spring 에서 `application.properties` 에 정의된 값을 가져올 땐, 2개의 대표적인 방법을 사용할 수 있다.
    - `@Value` , `@ConfigurationProperties`
- 일반적으로 프로퍼티 값은 런타임 도중에 변경될 일이 없다. 따라서 불변하게 관리하는 것이 좋다.
- `@Value` 의 문제점
    - 생성자를 통해 프로퍼티 값을 바인딩할 수 없어, 필드를 불변하게 관리할 수 없다.
    - 계층적인 구조를 가진 프로퍼티에 대해 중복되는 prefix를 설정해줘야 한다.
- `@ConfigurationProperties`
    - 생성자를 통해서 프로퍼티 값을 바인딩할 수 있어, 필드를 `final` 로 선언하여 안전하게 관리할 수 있다.
    - 클래스 레벨에 설정되는 애너테이션이고, 애너테이션 속성을 통해 prefix 설정이 가능하다. 따라서 중복되는 코드를 제거할 수 있다.
    - 만약 생성자가 여러 개 있다면, 바인딩시 사용할 생성자에 반드시 `@ConstructorBinding` 애너테이션을 붙여야 한다.

이제껏 `@Value` 에 대해서만 알고 계셨거나 사용해오셨던 독자님께서는 꼭 해당 내용을 참고해서 개발해보시기 추천드립니다!

혹시 글에 오류가 있거나 개선이 필요하다면 언제든지 댓글로 알려주세요. 감사합니다.