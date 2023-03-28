---
category: Spring-MVC
tags: [Spring-MVC]
title: "[스프링 - MVC] 검증 - Form 전송 객체의 분리"
date:   2021-08-14 18:30:00 
lastmod : 2021-08-14 18:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

- 이전 게시글
  1. [검증 - 기초](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_Validation)
  2. [검증 - 오류코드와 메시지처리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndMessage)
  3. [검증 - 검증 로직과 컨트롤러 분리](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_ValidationAndController)
  4. [검증 - Bean Validation](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_BeanValidation)

<br/><br/>

# Form 전송 객체의 분리

## 개요

- [이전 글](https://taegyunwoo.github.io/spring-mvc/SPRING_MVC_BeanValidation)에서 Bean Validation 기능을 통해 폼 객체를 검증하는 방법에 대해 살펴보았다.
- 만약 **"`Product 객체` 를 등록하는 Form"** 과 **"`Product 객체` 를 수정하는 Form"** 두가지가 존재할 때, 우리는 `Product 객체` 를 어떻게 다뤄야할까?

<br/>

### 공용 `Product 객체` 사용시 한계점

- 서로 다른 두가지 Form에서 "Bean Validation이 적용된 `Product 객체` "를 공용으로 사용한다면, 각 Form에서 서로 다른 검증 조건을 적용할 수 없다.
- 또한, 서로 다른 두가지 Form에서 사용하는 객체는 정확히 똑같지 않다.
- **따라서 각 Form 마다 다루는 객체를 따로 분리하여 관리해야한다.**

<br/><br/>

## 예시 웹 애플리케이션

- 지금부터 예시를 통해 Form 객체를 어떻게 분리하는지 알아보자.
- 본 예시는 상품을 등록하고 수정하는 웹 애플리케이션이다.
- 포스팅 글에서는 자세한 코드는 제공하지 않는다. 전체 소스코드는 아래를 참고하자.
    - [**예시 애플리케이션 다운로드**](https://github.com/TaegyunWoo/TaegyunWoo.github.io/blob/master/ExampleProjects/FormValidation.zip)

> **[Point!]**  
  `Product` 가 `ProductAddForm` 와 `ProductUpdateForm` 로 분리되었다는 사실에 주목하자.

<br/>

### 요구사항

- 상품 정보
    - 상품 Id
    - 상품 이름
    - 상품 가격

<br/>

- 상품을 등록하는 기능
    - 상품 id, 상품 이름, 상품 가격을 설정해야한다.

<br/>

- 상품을 수정하는 기능
    - 상품 id는 수정할 수 없다.

<br/>

- 상품 등록시 검증 조건
    - 상품 Id는 비어있을 수 없다.
    - 상품 이름은 비어있을 수 없다.
    - 상품 가격은 1000 이상이다.

<br/>

- 상품 수정시 검증 조건
    - 상품 이름은 비어있을 수 없다.
    - 상품 이름 뒤에 ".Updated" 라는 단어가 붙어야한다.
    - 상품 가격은 0 이상이다.

<br/>

### 상품 객체: `Product` 클래스

```java
/**
 * 상품도메인
 */
public class Product {
	private Long productId;
	private String productName;
	private Integer productPrice;
	
	// 생성자, getter, setter 생략
}
```

<br/>

### 상품 등록 Form 객체: `ProductAddForm` 클래스

```java
public class ProductAddForm {
  @NotNull
  private Long productId;

  @NotBlank
  private String productName;
  
  @Min(1000)
  private Integer productPrice;

	//생성자, getter, setter 생략
}
```

<br/>

### 상품 수정 Form 객체: `ProductUpdateForm` 클래스

```java
public class ProductUpdateForm {
	//검증조건이 없다.
  private Long productId;

  @NotBlank
  @Pattern(regexp = "^.*\\.Updated")
  private String productName;

  @Min(0)
  private Integer productPrice;
	
	//생성자, getter, setter 생략
}
```

<br/>

### 컨트롤러: `ProductController` 클래스

```java
import ...

@Controller
public class ProductController {

    private final ProductRepository productRepository;

    @Autowired
    public ProductController(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @GetMapping("/product/{productId}")
    public String viewProduct(@PathVariable Long productId, Model model) {
        model.addAttribute(productRepository.findById(productId));
        return "product";
    }

    @GetMapping("/product/add")
    public String viewAddForm(@ModelAttribute Product product) {
        return "addProduct";
    }

    @PostMapping("/product/add")
    public String addProduct(@Validated @ModelAttribute("product") ProductAddForm form,
                             BindingResult bindingResult,
                             RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            return "addProduct";
        }

        //product 추출
        Product product = new Product();
        product.setProductId(form.getProductId());
        product.setProductName(form.getProductName());
        product.setProductPrice(form.getProductPrice());

        //product 추가
        productRepository.addProduct(product);

        //PRG 패턴
        redirectAttributes.addAttribute("productId", product.getProductId());
        return "redirect:/product/{productId}/update";
    }

    @GetMapping("/product/{productId}/update")
    public String viewUpdateForm(@PathVariable Long productId, Model model) {
        Product product = productRepository.findById(productId);
        model.addAttribute(product);
        return "updateProduct";
    }

    @PostMapping("/product/{productId}/update")
    public String updateProduct(@PathVariable Long productId,
                                @Validated @ModelAttribute("product") ProductUpdateForm form,
                                BindingResult bindingResult,
                                RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            return "updateProduct";
        }

        //product 추출
        Product product = new Product();
        product.setProductId(productId);
        product.setProductName(form.getProductName());
        product.setProductPrice(form.getProductPrice());

        //product 수정
        productRepository.updateProduct(productId, product);

        redirectAttributes.addAttribute("productId", product.getProductId());
        return "redirect:/product/{productId}";
    }
}
```

<br/>

### 상품 등록 뷰 템플릿: `addProduct.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <style>
    .error-class {
      background-color: red;
    }
  </style>
</head>
<body>
<h1>상품 등록 폼</h1>
<form th:object="${product}" method="post">

  상품ID: <input type="text" th:field="*{productId}" placeholder="상품ID" th:errorclass="error-class">
  <div th:errors="*{productId}">
    상품 ID 오류시, 출력되는 태그
  </div> <br/>

  상품이름: <input type="text" th:field="*{productName}" placeholder="상품이름" th:errorclass="error-class">
  <div th:errors="*{productName}">
    상품 이름 오류시, 출력되는 태그
  </div> <br/>

  상품가격: <input type="text" th:field="*{productPrice}" placeholder="상품가격" th:errorclass="error-class">
  <div th:errors="*{productPrice}">
    상품 가격 오류시, 출력되는 태그
  </div> <br/>

  <input type="submit"/>
</form>
</body>
</html>
```

<br/>

### 상품 수정 뷰 템플릿: `updateProduct.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <style>
    .error-class {
      background-color: red;
    }
  </style>
</head>
<body>
<h1>상품 수정 폼</h1>
<form th:object="${product}" method="post">

  상품ID: <input type="text" th:field="*{productId}" placeholder="상품ID" th:errorclass="error-class" disabled>
  <div th:errors="*{productId}">
    상품 ID 오류시, 출력되는 태그
  </div> <br/>

  상품이름: <input type="text" th:field="*{productName}" placeholder="상품이름" th:errorclass="error-class">
  <div th:errors="*{productName}">
    상품 이름 오류시, 출력되는 태그
  </div> <br/>

  상품가격: <input type="text" th:field="*{productPrice}" placeholder="상품가격" th:errorclass="error-class">
  <div th:errors="*{productPrice}">
    상품 가격 오류시, 출력되는 태그
  </div> <br/>

  <input type="submit"/>
</form>
</body>
</html>
```

<br/>

### 상품 정보 뷰 템플릿: `product.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>

</head>
<body>
<h1>상품 정보</h1>
상품 ID: <p th:text="${product.productId}">상품ID</p>
상품 이름: <p th:text="${product.productName}">상품이름</p>
상품 가격: <p th:text="${product.productPrice}">상품가격</p>
</body>
</html>
```

<br/>

### 설명

- 각 Form( `addProduct.html`, `updateProduct.html` )에 따라 사용하는 객체(기존 객체: `Product`)가 분리되었다. 그리고 검증 조건도 다르게 설정되었다.
    - `ProductAddForm`
    - `ProductUpdateForm`

<br/>

- 컨트롤러에서 분리된 Form 객체를 다음과 같이 가져온다.
    - `@Validated @ModelAttribute("product") ProductAddForm form`
    - `@Validated @ModelAttribute("product") ProductUpdateForm form`
    - `@ModelAttribute("product")` 를 통해, Model에 product 라는 이름으로 바인딩한다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*