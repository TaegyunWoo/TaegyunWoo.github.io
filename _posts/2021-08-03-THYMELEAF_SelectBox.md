---
category: Thymeleaf
tags: [타임리프, 스프링통합, 셀렉트박스]
title: "[타임리프-스프링] 셀렉트 박스 처리"
date:   2021-08-03 18:45:00 
lastmod : 2021-08-03 18:45:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# 셀렉트 박스

이번 게시글에서는 타임리프를 통해 셀렉트 박스를 처리하는 방법에 대해 알아보자.

<br><br>

## 예시 코드

### 도메인: `DeliveryCode`

```java
/**
 * FAST: 빠른 배송  * NORMAL: 일반 배송  * SLOW: 느린 배송
 */
@Data
@AllArgsConstructor
public class DeliveryCode {
	private String code;
	private String displayName;
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
		private String deliveryCode; //배송 방식

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

	@ModelAttribute("deliveryCodes")
	public List<DeliveryCode> deliveryCodes() {
	    List<DeliveryCode> deliveryCodes = new ArrayList<>();
	    deliveryCodes.add(new DeliveryCode("FAST", "빠른 배송"));
	    deliveryCodes.add(new DeliveryCode("NORMAL", "일반 배송"));
	    deliveryCodes.add(new DeliveryCode("SLOW", "느린 배송"));
			return deliveryCodes;
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

`@ModelAttribute` 는 다른 메서드가 호출될 때마다, 본인 메서드를 실행하여 반환값을 model 객체에 바인딩한다. 따라서 `List<DeliveryCode> deliveryCodes = new ArrayList<>();` 를 통해, 다른 메서드가 호출될 때마다 `ArrayList` 객체가 생성된다. 그러므로, 다른 곳에서 미리 생성해둔 뒤 반환하는 구조로 설계해야 더 효율적이지만, 예시의 간소화를 위해 그대로 진행하겠다.

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

	<div>
		<div>상품 종류</div>
		<div th:each="type : ${itemTypes}" class="form-check form-check-inline">
			<input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
			<label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">
				BOOK
			</label>
		</div>
	</div>

	<!--------------------- 주목!!! ------------------------------->
	<div>
		<div>배송 방식</div>
		<select th:field="*{deliveryCode}" class="form-select">
			<option value="">==배송 방식 선택==</option>
			<option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}"
				th:text="${deliveryCode.displayName}">FAST</option>
		</select>
	</div>
	<!------------------------------------------------------------->

	<button type="submit">상품 등록</button>

</form>

</body>
</html>
```

<br><br>

## 상세 설명

### 타임리프

- `<select th:field="*{deliveryCode}" ... >`
    - `th:field="*{deliveryCode}"` 은 컨트롤러로부터 전달받은 `item` 객체의 `deliveryCode` 프로퍼티를 의미한다.
    - `id="itemType#"` (동적으로 생성된다.), `name="itemType"` 이 생성된다.

<br>

- `<option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}" th:text="${deliveryCode.displayName}">`
    - `th:each="deliveryCode : ${deliveryCodes}"` 을 통해, 컨트롤러로부터 전달받은 `deliveryCodes` 객체를 하나씩 꺼내며 반복한다.

<br>

### 타임리프의 선택 확인

`<option>` 태그의 `th:value="${deliveryCode.code}"` 와, `<select>` 태그의 `th:field="*{deliveryCode}"` 의 값을 비교하여 `selected` 옵션을 자동적으로 추가해준다.

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*