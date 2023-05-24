---
category: Tech
tags: [Tech]
title: "Java 애너테이션 프로세서는 어떻게 동작할까?"
date:   2023-05-23 18:15:00 +0900
lastmod : 2023-05-23 18:15:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

최근 애너테이션 프로세서에 대한 개념을 복기하던 중, 잘못 알고 있는 사실이 있었습니다.

이번 기회에 애너테이션 프로세서가 어떻게 동작하는지 알아보게 되었고, 이를 포스팅으로 남기려 합니다.

<br/>

## 자바의 애너테이션 프로세서란?

먼저 애너테이션 프로세서가 무엇인지 확실히 알아둬야겠습니다!

애너테이션 프로세서란, **개발자가 작성한 애너테이션 정보를 가지고 추가적인 프로세스를 동작하게 만들어주는 것**을 의미합니다.

**컴파일 단계에서 실행**되기 때문에, 빌드 단계에서 에러를 출력하게 할 수 있거나, 소스코드 및 바이트 코드를 생성할 수도 있습니다.  
모든 처리가 컴파일 시점에 발생하기 때문에, 런타임에 처리되는 것 보다 속도가 빠릅니다.

대표적인 예시로는 `@Override` 나 록복 라이브러리가 있습니다.

<br/>

### 애너테이션 프로세싱 과정

![Untitled](/assets/img/2023-05-23-Tech_AnnotationProcessor/Untitled.png)

- 애너테이션 처리는 **여러 라운드**에 걸쳐서 수행되게 됩니다.
- 각 라운드마다 아직 처리되지 않은 애너테이션을 스캔하고, 만약 처리되지 않은 애너테이션이 있다면 작업을 수행합니다.
- 모든 애너테이션이 처리되었다면, 만들어둔 모든 `.java` 파일을 컴파일합니다.
    
    > 참고로 `.java` 파일뿐만 아니라, 바이트코드로 이루어진 `.class` 도 생성할 수 있습니다.
    > 

<br/><br/>

## 애너테이션 프로세서를 통해, 자동으로 소스파일 만들기

> [프로젝트 전체 소스코드](https://github.com/TaegyunWoo/Study-Pratices/tree/main/Annotation-Processor)

이번에는 실제로 애너테이션 프로세서를 사용해서, 소스파일을 만들어봅시다.

**어떤 객체를 나타내는 클래스(`Employee`)가 있을 때, 이 클래스에 대한 빌더 클래스(`EmployeeBuilder`)를 컴파일 시점에 자동으로 생성하도록 해보겠습니다.**

우리는 총 2개의 모듈을 만들겁니다. **하나는 애너테이션 프로세서 모듈**이고, **다른 하나는 이 애너테이션 프로세서를 사용하는 모듈**입니다.

가장 먼저 애너테이션 프로세서 모듈을 만들어볼까요?

*참고로 모든 모듈의 빌드 도구는 Gradle을 사용합니다.*

<br/><br/>

## 애너테이션 프로세서 모듈

### 의존성 추가

```groovy
annotationProcessor 'com.google.auto.service:auto-service:1.0.1'
compileOnly 'com.google.auto.service:auto-service:1.0.1'
```

위와 같이 구글의 Auto-Service 를 추가합니다.

Auto-Service는 애너테이션 프로세서를 만들기 위해 필요한 메타 정보를 편리하게 구성할 수 있게 도와주는 라이브러리입니다.

**만든 애너테이션 프로세서를 Javac에 등록할 때, 필요한 설정 파일을 자동으로 작성해주는 역할**을 하게 됩니다.

자세한 것은 아래 공식 Repo를 참고하세요.

[https://github.com/google/auto/tree/main/service](https://github.com/google/auto/tree/main/service)

<br/>

### `@MyBuilder` 애너테이션

먼저, 애너테이션 프로세서가 동작할 애너테이션 `@MyBuilder` 를 만들어봅시다.

```java
@Target(ElementType.TYPE) //클래스, 인터페이스, enum에 적용
@Retention(RetentionPolicy.SOURCE) //컴파일 시점까지 애너테이션 유지
public @interface MyBuilder {

}
```

- `@Target(ElementType.TYPE)`
    - 이 애너테이션을 클래스, 인터페이스, enum 레벨에 적용할 수 있습니다.
- `@Retention(RetentionPolicy.SOURCE)`
    - `@MyBuilder` 애너테이션을 컴파일 시점까지만 유지하고, 런타임 시점에서는 제거합니다.
    - 애너테이션 프로세서는 컴파일 시점에 동작하므로, 런타임 시점까지 애너테이션 정보를 유지할 필요가 없기 때문에 `RetentionPolicy.SOURCE` 으로 설정했습니다.
    
    |||
    | --- | --- |
    | RetentionPolicy.SOURCE | 컴파일러가 해당 애너테이션 정보를 사용한 뒤 제거합니다. |
    | RetentionPolicy.CLASS | 컴파일 이후, 클래스 파일까지 유지합니다. <br/> 런타임 시점에서는 제거됩니다. |
    | RetentionPolicy.RUNTIME | 런타임 시점까지 유지됩니다. |

<br/>

### `MyBuilderProcessor` 클래스

```java
import com.google.auto.service.AutoService;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.Processor;
import javax.annotation.processing.RoundEnvironment;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.ElementKind;
import javax.lang.model.element.TypeElement;
import java.io.PrintWriter;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import static java.util.stream.Collectors.joining;

@AutoService(Processor.class) //구글의 Auto-Service 사용
//@SupportedAnnotationTypes("annotation.processor.MyBuilder")
//@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class MyBuilderProcessor extends AbstractProcessor {
  /**
   * 어떤 애너테이션(Type)에 이 프로세서를 적용할 것인지 결정하는 메서드.
   * @SupportedAnnotationTypes("annotation.processor.MyBuilder") 를
   * 추가해서 생략가능
   * @return 지원 애너테이션(타입) 모음 Set
   */
  @Override
  public Set<String> getSupportedAnnotationTypes() {
    Set<String> supportedAnnotationTypes = new HashSet<>();
    supportedAnnotationTypes.add("annotation.processor.MyBuilder"); //적용할 애너테이션 이름(패키지 포함)
    return supportedAnnotationTypes;
  }

  /**
   * 이 프로세서가 지원할 Java 버전
   * @SupportedSourceVersion(SourceVersion.RELEASE_17) 를
   * 추가해서 생략 가능
   * @return 지원 버전
   */
  @Override
  public SourceVersion getSupportedSourceVersion() {
    return SourceVersion.RELEASE_17;
  }

  /**
   * 실제로 본 애너테이션 프로세서가 어떻게 동작할지 정의
   * @param annotations the annotation interfaces requested to be processed
   * @param roundEnv  environment for information about the current and prior round
   * @return 성공 여부
   */
  @Override
  public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
    annotations.stream().forEach(annotation ->
        roundEnv.getElementsAnnotatedWith(annotation) //애너테이션이 붙은 요소(클래스, 메서드 등의 정보)들을 가져옵니다.
            .stream().forEach(this::generateBuilderFile) //각 요소마다 generateBuilderFile 메서드를 호출합니다.
    );
    return true;
  }

  private void generateBuilderFile(Element element) {
    //애너테이션 붙은 Target의 이름 (우리는 클래스 레벨에 적용했으므로, 클래스 이름이 반환됨)
    String className = element.getSimpleName().toString();

    //애너테이션 붙은 Target의 상위 이름 (우리는 클래스 레벨에 적용했으므로, 패키지 이름이 반환됨)
    String packageName = element.getEnclosingElement().toString();

    //생성할 빌더 클래스의 이름
    String builderName = className + "Builder";

    //생성할 빌더의 이름 (패키지 포함)
    String builderFullName = packageName + "." + builderName;

    //현재 요소(클래스)에 속한 하위 요소(필드, 메서드 등) 중, 필드 요소인 것만 추출하기
    List<? extends Element> fieldElementList = element.getEnclosedElements().stream().filter(
        enclosedElement -> ElementKind.FIELD.equals(enclosedElement.getKind())
    ).toList();

    //빌더 .java 파일 작성 (이 부분은 자세히 보시지 않아도 됩니다!)
    PrintWriter writer;
    try {
      //.java 파일을 작성하기 위해, PrintWriter 객체 생성
      writer = new PrintWriter(
          processingEnv.getFiler().createSourceFile(builderFullName).openWriter()
      );

      //아래부턴 빌더 코드 작성 로직
      writer.println("""
                    package %s;
                         
                    public class %s {
                    """
          .formatted(packageName, builderName)
      );

      fieldElementList.forEach(field ->
          writer.print("""
                                private %s %s;
                            """.formatted(field.asType().toString(), field.getSimpleName())
          )
      );

      writer.println();
      fieldElementList.forEach(field ->
          writer.println("""
                                public %s %s(%s value) {
                                    %s = value;
                                    return this;
                                }
                            """.formatted(builderName, field.getSimpleName(),
              field.asType().toString(), field.getSimpleName())
          )
      );

      writer.println("""
                        public %s build() {
                            return new %s(%s);
                        }
                    """.formatted(className, className,
          fieldElementList.stream().map(Element::getSimpleName).collect(joining(", ")))
      );
      writer.println("}");

      writer.close();
    } catch (Exception e) {
      e.printStackTrace();
    }

  }
}
```

- `getSupportedAnnotationTypes()` 메서드
    - 이 애너테이션 프로세서가 적용될 애너테이션(타입)을 결정하는 메서드입니다.
    - return된 Set에 지원할 애너테이션의 이름(패키지명 포함)이 담기게 됩니다.
    - `@SupportedAnnotationTypes` 으로 교체할 수 있습니다.
- `getSupportedSourceVersion()` 메서드
    - 이 애너테이션 프로세서가 적용될 자바 버전을 설정합니다.
    - `@SupportedSourceVersion` 으로 교체할 수 있습니다.
- `@AutoService(Processor.class)`
    - 구글에서 제공하는 Auto-Service 라이브러리를 사용해서, 애너테이션 프로세서를 정의할 때 필요한 메타정보를 편리하게 추가합니다.
    - 빌드 후 `build/classes/java/main/META-INF/services` 경로에 `javax.annotation.processing.Processor` 라는 파일이 생성된 것을 확인할 수 있습니다.
    - 해당 파일 안에는 애너테이션 프로세서로 등록될 클래스의 이름이 담기게 되는데, 이것을 자동으로 만들어주는 역할을 Auto-Service가 수행해줍니다.
- `process(...)` 메서드
    - 이 애너테이션 프로세서가 어떻게 동작할지 결정하는 메서드
    - 컴파일러에 의해서 호출돼서, 프로세싱을 진행하게 됩니다.
    - 파라미터
        - `Set<? extends TypeElement> annotations` : 적용될 수 있는 애너테이션(의 정보)가 담기게 됩니다.
        - `RoundEnviroment roundEnv` : 현재 수행 중인 프로세싱 라운드 정보가 담기게 됩니다.
    - 반환값
        - `return true` : 만약 프로세싱 작업이 성공적으로 처리되었다면, true를 반환해야 합니다. 이를 통해서, 다른 애너테이션 프로세서가 동일한 애너테이션을 처리하지 않도록 만듭니다.

<br/>

### Jar 빌드

이제 애너테이션 프로세서를 만들기 위한 소스코드는 모두 작성했습니다.

다른 모듈에서 실제로 사용하기 위해, 이제 이 모듈을 빌드하도록 합니다.

```bash
# 명령어 (테스트 없이 Jar 빌드)
gradle build --exclude-task test
```

*참고로 `main` 메서드 없이 빌드해야 합니다.*

위 명령어를 통해, 빌드를 수행합니다.

그리고 프로젝트 디렉토리(`src` 디렉토리와 동일한 레벨)에서 `libs` 디렉토리에 빌드된 jar 파일을 복사합니다.

이 jar 파일을 다른 모듈에 import해서 사용하겠습니다.

<br/><br/>

## 애너테이션 프로세서 사용 모듈

이제 빌드한 애너테이션 프로세서 모듈을 사용할 모듈을 만들어보겠습니다.

새로운 프로젝트를 준비해주세요!

<br/>

### jar 파일 복사

현재 프로젝트 디렉토리(`src` 디렉토리와 동일한 레벨)에 새로운 디렉토리 `libs` 를 만듭니다.

그리고 아까 만들어둔 애너테이션 프로세서 모듈을 해당 디렉토리에 옮겨옵니다.

<br/>

### 의존성 추가

```groovy
implementation files('libs/빌드한_애너테이션_프로세서_모듈.jar')
annotationProcessor files('libs/빌드한_애너테이션_프로세서_모듈.jar')
```

위와 같이, Gradle 의존성을 추가합니다.

> 추가로, IntelliJ IDE를 사용하고 계시면 `Annotation Processor` 옵션을 활성화해주셔야 합니다.
> 

<br/>

### `Employee` 클래스

이제 실제로 빌더 클래스의 목표가 될 `Employee` 클래스를 만들어봅시다.

```java
import annotation.processor.MyBuilder;

@MyBuilder //애너테이션 프로세서 모듈의 애너테이션 사용
public class Employee {
  private String name;
  private int age;

  public Employee(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
```

아까 만들어둔 애너테이션 `@MyBuilder` 를 추가합니다.

<br/>

### 테스트

이제 모든 준비는 끝났습니다!

빌드를 진행해보면, `build/generated/sources` 디렉토리 안에 자동으로 생성된 빌더 클래스를 확인해볼 수 있습니다.

![Untitled](/assets/img/2023-05-23-Tech_AnnotationProcessor/Untitled%201.png)

실제로 실행할 수 있는지 확인해보면 아래와 같습니다.

```groovy
Employee employee = new EmployeeBuilder()
            .age(24)
            .name("James")
            .build();
```

<br/><br/>

## 정리

지금까지 애너테이션 프로세서가 무엇이고, 어떻게 동작하는지 살펴봤습니다.

그리고 직접 애너테이션 프로세서를 만들고, 적용해보는 시간을 가졌습니다.

이번 포스팅을 통해, 모두 애너테이션 프로세서에 대해 깊이 이해할 수 있었으면 좋겠습니다!

감사합니다.