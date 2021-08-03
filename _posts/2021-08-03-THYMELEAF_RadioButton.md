---
category: Thymeleaf
tags: [타임리프, 스프링통합, 라디오버튼]
title: "[타임리프-스프링] ENUM을 활용한 라디오 버튼 처리"
date:   2021-08-03 18:30:00 
lastmod : 2021-08-03 18:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 라디오 버튼

이번에는 타임리프를 통해 라디오 버튼을 처리하는 방법에 대해 설명하도록 하겠다. 참고로 자바의 ENUM을 활용하여 설명한다.

예시코드로 바로 넘어가자.

<br><br>

## 예시 코드

### 도메인: `ItemType`

```java
public enum ItemType {
	BOOK("도서"), FOOD("식품"), ETC("기타");

	private final String description;

	ItemType(String description) {
		this.description = description;
	}

	public String getDescription() {
		return description;
	}

}
```

<br>

### 도메인: `Item`

```java
@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    private Boolean open; //판매 여부
		private List<String> regions; //등록 지역
		private ItemType itemType; //상품 종류

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

`@ModelAttribute` 애너테이션이 어떻게 사용되었는지 보자.

```java
@Slf4j
@Controller
@RequestMapping("/form/items")
@RequiredArgsConstructor
public class FormItemController {

	private final ItemRepository itemRepository;

	@ModelAttribute("itemTypes")
	public ItemType itemTypes() {
			return ItemType.values(); // 해당 ENUM의 모든 정보를 배열로 반환한다. [BOOK, FOOD, ETC]
	}

// ------------- 상품 객체 추가를 위한 메서드 (위에서 살펴본 메서드들) -----------------

	@GetMapping("/add")
	public String addForm(Model model) {
		model.addAttribute("item", new Item());
		return "form/addForm";
	}

	
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

		<div>판매 여부</div>
		<div>
			<div class="form-check">
				<input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
				<label for="open" class="form-check-label">판매 오픈</label>
			</div>
		</div>

	<div>
		<div>등록 지역</div>
		<div th:each="region : ${regions}" class="form-check form-check-inline">
			<input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
			<label th:for="${#ids.prev('regions')}"
			th:text="${region.value}" class="form-check-label">서울</label>
		</div>
	</div>

	<!--------------------- 주목!!! ------------------------------->
	<div>
		<div>상품 종류</div>
		<div th:each="type : ${itemTypes}" class="form-check form-check-inline">
			<input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
			<label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">
				BOOK
			</label>
		</div>
	</div>
	<!------------------------------------------------------------->

	<button type="submit">상품 등록</button>

</form>

</body>
</html>
```

<br><br>

## 상세 설명

[체크박스를 다룬 이전 글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_CheckBox)에서 설명한 내용과 유사하지만 약간의 차이점이 존재한다. 이 부분을 생각하며 글을 읽도록 하자.

> 체크박스에 대해 다룬 게시글을 읽지 않은 분은 본 게시글을 읽기 전, 해당 게시글을 읽고 오시길 권장한다.

<br>

### 타임리프

- `<div th:each="type : ${itemTypes}" ... >`
    - [이전 글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_Each)에서 다룬 each 문과 동일하다. `${itemTypes}` 는 `ItemType` ENUM의 정보가 담겨있는 배열이다. (컨트롤러로부터 전달받았다.)

<br>

- `<input type="radio" th:field="*{itemType}" th:value="${type.name()}" ... >`
    - `th:field="*{itemType}"` 은 `item` 객체의 `itemType` 프로퍼티를 의미한다.
        - `id="itemType#"` (동적으로 생성된다.), `name="itemType"` 이 생성된다.
        - **Checkbox와는 다르게, 히든필드가 생성되지 않는다. (라디오 버튼은 무조건 하나를 선택해야하기 때문에)**
    - `th:value="${type.name()}"` 은 each문에서 사용된 `type` 객체의 `name` 프로퍼티를 접근하여 `value` 속성의 값으로 설정한다. (`type.name()` 은 ENUM의 이름이다. FOOD 등..)

<br>

- `<label th:for="${#ids.prev('itemType')}" th:text="${type.description}" ... >`
    - `th:for="${#ids.prev('itemType')}"` 은 동적으로 생성된 id를 사용할 수 있게끔 해준다.
    - `th:text="${type.description}"` 은 `type` 객체의 `description` 프로퍼티를 접근하여 `text` 속성의 값으로 설정한다. (`type.description` 은 위에서 작성한 ENUM의 설명이다. 식품 등..)

<br>

### 타임리프의 체크 확인

[이전 글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_CheckBox)에서 설명한 것과 마찬가지로, ‘`th:field="${필드A}"` 의 `필드A`의 값’과 ‘`th:value="${필드B}"` 의 `필드B`의 값’ 을 비교하여, 두 값이 같다면 `checked` 옵션을 추가하게 된다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*