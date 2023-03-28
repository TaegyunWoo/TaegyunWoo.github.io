---
category: Tech
tags: [Tech]
title: "Lombok 의 @Builder 사용시, MapStruct 버그"
date:   2022-09-18 21:15:00 
lastmod : 2022-09-18 21:15:00
sitemap :
  changefreq : daily
  priority : 1.0
---

MapStruct 를 사용하여 프로젝트를 진행하던 중, 이상한 버그를 발견하여 포스팅으로 해결 방법을 남긴다.

<br/>

## 무슨 버그?

Lombok 에서 빌더패턴을 쉽게 적용할 수 있는 애너테이션을 제공한다.

`@Builder` , `@SuperBuilder` 를 통해, 특정 클래스에 빌더 패턴을 간편하게 적용할 수 있다.

이렇게 빌더 패턴을 적용한 클래스를 MapStruct의 `target` 으로 사용하면, 아래 버그가 발생한다.

<br/>

**MapStruct 의 `@AfterMapping` 이 적용되지 않는 문제**

<br/>

위 문제는 MapStruct 1.3 버전부터 발생된 것으로 보인다.

자세한 것은 아래를 참고하자.

- [https://stackoverflow.com/questions/59028797/aftermapping-is-not-called-from-mapper-interface](https://stackoverflow.com/questions/59028797/aftermapping-is-not-called-from-mapper-interface)
- [https://github.com/mapstruct/mapstruct/issues/1556#issuecomment-1011493330](https://github.com/mapstruct/mapstruct/issues/1556#issuecomment-1011493330)`

## 해결방법

MapStruct 가 Lombok 으로 생성한 빌더를 사용하지 않도록 설정하면, 정상적으로 `@AfterMapping` 이 동작한다.

적용 방법은 아래와 같다.

```java
@Mapper(builder = @Builder(disableBuilder = true)) //빌더패턴 비활성화
public interface MyMapper {
	//...

	@AfterMapping
	default void callAfterMapping(...) {
		//...
	}
}
```