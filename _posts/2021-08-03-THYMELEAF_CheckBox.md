---
category: Thymeleaf
tags: [Thymeleaf]
title: "[타임리프-스프링] 체크박스의 원리와 타임리프"
date:   2021-08-03 17:30:00 
lastmod : 2021-08-03 17:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# 단일 체크박스

## 스프링에서의 체크박스 작동 원리

스프링에서 체크박스를 처리하는 방식이 약간 특이하다. 먼저 스프링이 체크박스를 어떻게 처리하는지 알아본 뒤, 타임리프를 통해 처리하는 방법을 설명하도록 하겠다.

<br>

### 체크박스 기본 원리

```html
<input type="checkbox" name="myCheckBox"/>
```

위와 같은 체크박스 형식을 통해 데이터를 서버에 전송하면, 서버는 어떻게 받을까? **먼저 서버는 `myCheckBox=on` 이라는 형태로 데이터를 받는다. 이후 스프링은 `on` 이라는 문자를 `true` 타입으로 변환해준다.**

<br>

- **체크가 되어있을 때, 서버측이 받는 형태**
    - `myCheckBox=on`
    - (변환된 형태: `myCheckBox=true`)
- **체크가 되어있지 않을 때, 서버측이 받는 형태**
    - 아무런 값도 전송되지 않는다.
    - `myCheckBox` 필드 자체가 전송되지 않는다!

<br>

여기서 주목해야하는 점이 바로 체크가 되어있지 않을 경우이다. **체크박스에 체크가 되어있지 않으면, 아예 해당 필드를 전송하지 않는다. 즉, `필드명=off` 를 전송하는 것이 아니다! 필드명 자체가 전송이 안된다.**

이 경우, 서버 측에서는 아무런 값도 받지 못했기 때문에, 서버 구현에 따라 값이 오지 않은 것으로 판단하여 논리적 오류가 발생할 수 있다. 이런 문제를 해결하기 위해, 스프링은 특별한 기능을 제공한다.

<br>

### 스프링의 체크박스 처리 원리

스프링은 **히든 필드를 하나 만들어서, 전송하면 체크를 해제했다는 사실을 인식하게 된다.**

```html
<input type="checkbox" name="myCheckBox"/>
<input type="hidden" name="_myCheckBox" value="on"/>
```

위 코드처럼, 'checkbox의 name 속성 값 `myCheckBox` 앞에 언더바(`_`)를 붙인 값'을 name 속성으로 취하는 hidden 필드를 통해 문제를 해결한다.

<br>

- **체크가 되어있을 때, 서버측이 받는 형태**
    - `myCheckBox=on & _myCheckBox=on`
    - 이때, `myCheckBox` 가 `on` 이므로, `_myCheckBox` 는 무시된다.
- **체크가 되어있지 않을 때, 서버측이 받는 형태**
    - `_myCheckBox=on`
    - 스프링MVC가 `_myCheckBox` 만 있는 것을 확인하고, `myCheckBox` 의 값이 체크되지 않았다고 인식한다.

<br>

이제 스프링에서 체크박스를 어떻게 처리하는지 살펴보았으므로, 타임리프를 통해 체크박스를 처리해보자.

<br><br>

## 예시 코드

개발을 할 때마다, 히든필드를 추가하는 것은 매우 번거롭다. 따라서, 타임리프는 적절한 기능을 제공한다. 예시코드를 통해 바로 알아보자.

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

### 타임리프

이번에는 `checkbox` 에 작성된 `th:field` 에 중점을 맞추어 살펴보자.

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

	<!------------------------- 주목!!! --------------------------->
		<div>판매 여부</div>
		<div>
			<div class="form-check">
				<input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
				<label for="open" class="form-check-label">판매 오픈</label>
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

- `<input type="checkbox" id="open" th:field="*{open}" class="form-check-input">`
    - `th:field` 가 `id` , `name` , `value` 속성을 자동으로 생성해준다는 것은 이전 글에서 다뤘다.
    - 그리고 `th:field` 는 해당 속성의 타입이 `checkbox` 이면, **자동으로 히든필드( `<input type="hidden">` ) 까지 생성해준다.**
    - 따라서, 서버측에서 체크박스가 해제되었을 때도, 정확히 인식할 수 있다.

<br><br>

## 타임리프의 체크 확인

타임리프의 `th:field` 를 통해 자동으로 `checked` 속성을 추가할 수도 있다.

```html
<input type="checkbox" th:field="${isChecked}" disabled/>
```

만약, 위와 같은 상태에서 서버가 `isChecked=true` 라는 데이터를 넘겨주었다면 타임리프는 다음과 같이 렌더링한다.

```html
<input type="checkbox" id="isChecked" name="isChecked" value="true" checked="checked" disabled/>
```

`id` , `name` , `value` 속성 뿐만 아니라, `checked` 속성까지 추가된 것을 확인할 수 있다.

<br><br><br>

# 다중 체크박스

위에서 체크박스를 한 개만 사용하는 예시를 알아보았다. 이제 여러개의 체크박스를 사용하는 예시를 알아보자.

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

    private Boolean open; //판매 여부
		private List<String> regions; //등록 지역

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

	@ModelAttribute("regions")
	public Map<String, String> regions() {
	    Map<String, String> regions = new LinkedHashMap<>();
	    regions.put("SEOUL", "서울");
	    regions.put("BUSAN", "부산");
	    regions.put("JEJU", "제주");
			return regions;
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

	<!--------------------- 주목!!! ------------------------------->
	<div>
		<div>등록 지역</div>
		<div th:each="region : ${regions}" class="form-check form-check-inline">
			<input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
			<label th:for="${#ids.prev('regions')}"
			th:text="${region.value}" class="form-check-label">서울</label>
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

### 컨트롤러

`서울, 부산, 제주`라는 체크박스를 여러가지 뷰(타임리프)가 보여주기 위해, 컨트롤러에게 `서울, 부산, 제주`라는 데이터를 받아야 한다. **만약 컨트롤러에게 데이터를 받지 않고, 여러가지 뷰가 자체적으로 `서울, 부산, 제주` 라는 정보를 출력한다면 코드의 중복이 발생한다. 또한, 유지보수가 까다로워지기 때문에 컨트롤러에게 각 뷰들이 데이터를 넘겨받는 방식으로 구현해야한다.**

- `@ModelAttribute`
    - 메서드의 매개변수에 적용하는 것 뿐만 아니라, 메서드 레벨에 적용할 수도 있다.
    - 메서드 레벨에 적용시, 다른 메서드들이 호출될 때마다 해당 반환값을 model 객체에 담아준다.
    - **매개변수에 적용시**
        - 요청 데이터를 객체에 담아준다.
    - **메서드에 적용시**
        - 컨트롤러의 다른 메서드가 호출될 때마다, model 객체에 해당 메서드의 반환값을 담아준다.
    - 즉, `Map` 객체인 `regions` 를 model 객체에 담은 뒤, URL과 매핑된 메서드를 호출한다. (이때, `regions` 가 담긴 model 역시 해당 메서드로 넘겨진다.)

<br>

### 타임리프

- `<div th:each="region : ${regions}" ... >`
    - `th:each="region : ${regions}"` 을 통해, 컨트롤러에게 전달받은 `regions` 객체의 원소 개수만큼 반복하여 생성한다.
    - each 문에 대한 자세한 설명은 [이전 게시글](https://taegyunwoo.github.io/thymeleaf/THYMELEAF_Each)을 참고하자.

<br>

- `<input type="checkbox" th:field="*{regions}" th:value="${region.key}" .. >`
    - `th:field="*{regions}"` 은 컨트롤러에게 전달받은 `item` 객체의 `regions` 프로퍼티를 의미한다.
    - each문 안에서 `th:field` 가 쓰였다.
        - `id` 속성의 값 : `필드명1` , `필드명2` , `필드명3` , ... 으로 각각 설정된다.
        - `name` 속성의 값 : `필드명` 으로 각각 동일하게 설정된다.

            > 같은 `name` 으로 데이터를 서버에 전송하면, 각 데이터들이 묶여서 하나의 `List<String>` 로 변환된다.

    - `th:value="${region.key}"` 을 통해, 체크박스의 `value` 속성의 값을 컨트롤러에게 전달받은 `Map` 객체의 각 원소의 value로 설정한다. (이때 `Map` 객체는 `@ModelAttribute` 로 전달한 객체이다.)
    - `<input type="checkbox" ..>` 의 변환 결과

        ```html
        <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
        <input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions">
        <input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions">
        ```

<br>

- `<label th:for="${#ids.prev('regions')}" th:text="${region.value}" ... >`
    - 체크박스의 `id` 속성이 each문에 의해 자동으로 변화한다. (`regions1` , `regions2` , `regions3`) 따라서, `<label>` 태그의 `for` 속성을 적용하기 어렵다.
    - `th:for="${#ids.prev('regions')}"` 를 통해, 동적으로 생성된 id 정보를 쓸 수 있게 된다.
    - `#ids.prev('regions')` 은 이전에 사용한 id를 가져오는 역할을 한다. (`#ids` 객체는 타임리프가 제공하는 편의 객체이다.)

<br>

- **렌더링 결과**

    ```html
    <!---------렌더링 이전--------------->
    <div>등록 지역</div>
    <div th:each="region : ${regions}" class="form-check form-check-inline">
    	<input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
    	<label th:for="${#ids.prev('regions')}"
    	th:text="${region.value}" class="form-check-label">서울</label>
    </div>
    <!------------------------------------->

    <!---------렌더링 이후--------------->
    <div>등록 지역</div>
    <div class="form-check form-check-inline">
    	<input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
    	<input type="hidden" name="_regions" value="on">
    </div>
    <div class="form-check form-check-inline">
    	<input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions">
    	<input type="hidden" name="_regions" value="on">
    </div>
    <div class="form-check form-check-inline">
    	<input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions">
    	<input type="hidden" name="_regions" value="on">
    </div>

    <!------------------------------------->
    ```

<br>

### 타임리프의 체크 확인

단일 체크박스에서 `th:field="${필드A}"` 에서 `필드A` 의 값을 판단하여 `checked` 옵션을 자동으로 추가하는 것을 기억할 것이다.

`th:field` 을 단독으로 사용하지 않고 `th:value` 와 함께 사용할 수 있다. 이 경우, '`th:field="${필드A}"` 의 `필드A`의 값'과 '`th:value="${필드B}"` 의 `필드B`의 값' 을 비교하여, 두 값이 같다면 `checked` 옵션을 추가하게 된다.

<br>

### 참고: _regions 히든필드

위에서 설명한 것과 같다. 모든 체크박스에 체크를 하지 않으면 서버측이 받는 데이터는 아래와 같다.

- 히든 필드 적용을 했을 때
    - 서버측 저장 타입이 boolean 일 때 : false
    - 서버측 저장 타입이 List 일 때 : 빈 List 객체

<br>

---

<br>

<a href="https://inf.run/YPER"><img src="/assets/img/Inflearn_Spring_MVC2/logo.png" width="400px" height="300px"></a>

- *본 게시글은 김영한님의 강의를 토대로 정리한 글입니다.*
- *더 자세한 내용을 알고 싶으신 분들이 계신다면, 해당 강의를 수강하시는 것을 추천드립니다.*