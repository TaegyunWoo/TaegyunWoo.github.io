---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프-스프링] 입력 폼 처리"
date:   2021-08-03 17:00:00 
lastmod : 2021-08-03 17:00:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 입력 폼 처리

이번 게시글에서 다룰 기능은 아래와 같다.

- `th:object`
    - 커맨드 객체를 지정한다.
- `*{...}`
    - 선택 변수식이다.
    - `th:object` 에서 선택한 객체에 접근한다.
- `th:field`
    - HTML 태그의 `id` , `name`, `value` 속성을 자동으로 처리해준다.

예시 코드를 통해 설명하도록 하겠다.

<br><br>

## 예시 코드

### 도메인: `Item`

```java
@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }

}
```

<br>

### 컨트롤러

```java
@Slf4j
@Controller
@RequestMapping("/form/items")
@RequiredArgsConstructor
public class FormItemController {

	private final ItemRepository itemRepository;

	@GetMapping("/add")
	public String addForm(Model model) {
		model.addAttribute("item", new Item()); //th:obejct 가 사용할 수 있도록 객체 넘겨줌
		return "form/addForm";
	}

	//Item 객체를 추가하는 메서드
	@PostMapping("/add")
	public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
		log.info("item.open={}", item.getOpen());

		Item savedItem = itemRepository.save(item);
		redirectAttributes.addAttribute("itemId", savedItem.getId());
		redirectAttributes.addAttribute("status", true);
		return "redirect:/form/items/{itemId}";
	}

}
```

<br>

### 타임리프

`th:object` 와 `th:field` 에 중점을 맞추어 살펴보자.

```html
<!DOCTYPE html>
<html	xmlns:th="http:// www.thymeleaf.org">
<head>
	<title>상품등록 폼</title>
</head>
<body>

	<form action="item.html" th:action th:object="${item}" method="post">

		<div>
			<label for="itemName">상품명</label>
			<input type="text" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
		</div>

		<div>
			<label for="price">가격</label>
			<input type="text" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
		</div>

		<div>
			<label for="quantity">수량</label>
			<input type="text" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
		</div>

	<button type="submit">상품 등록</button>

</form>

</body>
</html>
```

<br>

## 상세 설명

- `th:object="${item}"`
    - `<form>` 태그에서 사용할 객체를 지정한다.
    - 선택 변수 식 ( `*{...}` )을 사용할 수 있다.
    - `${item}` 은 컨트롤러로부터 전달받은 (Model) 객체 `Item` 이다.

<br>

- `th:field="*{itemName}"`
    - `<form>` 태그에서 `th:object` 로 설정한 객체 `${item}` 의 프로퍼티인 `itemName` 에 접근한다.
        - 즉, `th:field="*{itemName}"` == `th:field="${item.itemName}"` 이다.
    - `th:field` 는 `id` , `name` , `value` 속성을 자동으로 생성해준다.
    - `th:field="${item.itemName}"` 일 때,
        - `id` 속성 : `id="itemName"`
        - `name` 속성 : `name="itemName"`
        - `value` 속성 : `value="변수 itemName의 값"`

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*