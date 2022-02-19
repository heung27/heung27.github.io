---
layout: post
title: Bean Validation
date: 2022-02-17 23:09 +0900
categories: [SERVER, Spring]
tags: [Spring Boot, Java, Validation]
---



개발에서 가장 중요한 요소 중 하나가 Validation이다. 사용자로부터 어떤 값이 입력될지 모르기 때문에 검증하는 과정이 필수적이다. 

검증 로직은 클라이언트와 서버에서 수행하게 된다. 클라이언트에서만 검증하게 되면 postman과 같은 툴을 이용해 데이터를 조작할 수 있어 보안에 취약하며, 서버에서만 검증하게 되면 즉각적인 반응이 어려워 고객 사용성이 부족해진다. 

이 글에서는 Spring Boot에서 Bean Validation을 활용한 서버에서의 검증 방법을 살펴본다.

<br>

## Bean Validation 이란?

Bean Validation은 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준이다. 쉽게 이야기해서 검증 어노테이션과 여러 인터페이스의 모음이다. 마치 JPA가 표준 기술이고 그 구현체로 Hibernate 있는 것과 같다. 

Bean Validation을 구현한 기술 중에 일반적으로 사용하는 구현체는 Hibernate Validator이다. 

참고로 여기서의 Hibernate는 ORM과 관련이 없다.

<br>

## 사용법

### Dependency 추가

```gradle
  implementation 'org.springframework.boot:spring-boot-starter-validation'
```

spring-boot-starter-validation 의존관계를 추가하면 관련 라이브러리가 모두 추가 된다.

<br>

### Validation Annotaion

대표적인 어노테이션을 몇 가지 소개한다.

- @NotBlank : 빈 값, 공백을 허용하지 않는다.
- @NotNull : null을 허용하지 않는다.
- @Range(min = 1, max = 999) : 값의 범위를 지정한다.
- @Max(999) : 최댓값을 지정한다.
- @Min(1) : 최솟값을 지정한다.

<br>

더 많은 정보는 아래 링크의 공식 문서에 잘 정리되어 있다. 정말 다양한 어노테이션이 있으니 개발을 하면서 필요할 때마다 찾아 쓰면 될 것 같다.

validation annotation 모음 : [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

<br>

### 예제

~~~java
package hello.itemservice.domain.item;

import lombok.Data;
import org.hibernate.validator.constraints.Range;

import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

@Data
@ToString
public class Item {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}

~~~

Item 클래스의 멤버 변수에 validation annotaion을 적용했다. 

상품명은 빈칸과 공백을 허용하지 않고 가격은 1000이상 10000이하, 수량은 최대 9999로 제한된다.

<br>

import 코드를 보면 Range 어노테이션과 나머지 어노테이션의 위치가 다르다. 

javax.validation으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이고, org.hibernate.validator로 시작하면 hibernate validator 구현체를 사용할 때만 제공되는 검증 기능이다. 실무에서 대부분 hibernate validator를 사용하므로 자유롭게 사용해도 된다.

<br>

~~~java
package hello.itemservice.web.validation;

import hello.itemservice.domain.item.Item;
import hello.itemservice.web.validation.form.ItemSaveForm;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated Item item, BindingResult bindingResult) {

        log.info("API 컨트롤러 호출");
      	
        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }
      	
      	log.info(item.toString());
				log.info("성공 로직 실행");
        return item;
    }
}
~~~

Item 클래스를 테스트하기 위한 간단한 컨트롤러를 만들었다. 이제 API 요청이 오면 reqeust body에 담긴 itemName, price, quantity의 값을 검증하게 된다.

<br>

addItem 메서드의 매개 변수를 보면, 검증할 클래스(Item)에 **@Validated**가 적용된 것을 볼 수 있다. 해당 어노테이션은 스프링 전용 검증 어노테이션으로, 검증을 하기 위해서 반드시 적용되어야 한다.

같은 역할을 하는 **@Valid**도 존재한다. 자바 표준 검증 어노테이션으로, 사용하기 위해서는 build.gradle에 dependency 추가가 필요하다. 앞서 이미 추가한 'org.springframework.boot:spring-boot-starter-validation' 이다.

<br>

@Validated와 @Valid 둘 다 동작하지만 @Validated는 내부에 groups라는 기능을 포함하고 있다. 

groups는 하나의 클래스로 서로 다른 검증 조건을 제공하는 기능이다. 예를 들어 회원정보를 저장하고 수정하는 로직을 생각해 보자. 저장할 때는 회원번호를 입력받지 않지만(아직 회원이 아니므로 회원번호가 없다) 수정할 때는 회원번호가 필요하다. 따라서 한 클래스에서 서로 다른 검증 조건이 생긴 것이다. 

이런 경우 groups 기능을 활용해서 처리할 수 있다. 하지만 groups는 실무에서 잘 사용되지 않는다. 앞에서 간단한 회원번호로 예시로 들었지만 실무에서 저장과 수정에서 필요한 데이터는 크게 상이하다. 따라서 저장 폼, 수정 폼 두 개의 클래스로 분리하여 사용하는 것이 유지보수 측면에서 유리하다.

<br>

### 주의할 점

검증에 실패하면 스프링은 FieldError를 생성하면서 사용자가 입력한 값을 넣어둔다. 그리고 해당 오류를 BindingResult에 담아서 컨트롤러를 호출한다. 따라서 검증 실패 시에도 사용자의 오류 메시지를 정상 출력할 수 있다. 위의 예제에서는 "API 컨트롤러 호출"가 출력된 후, "검증 오류 발생 errors={발생된 에러}"가 출력된다. 

하지만 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면, 이후 단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다. 위의 예제에서는 "API 컨트롤러 호출" 로그가 출력되지 않고 Exception이 발생한다.

<br>

## Reference

- [스프링 MVC 2편 - 백엔드 웹 개발 활용 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2#)
- [Java Bean Validation 제대로 알고 쓰자](https://kapentaz.github.io/java/Java-Bean-Validation-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90/#)
- [Spring Boot Bean Validation 제대로 알고 쓰자](https://kapentaz.github.io/spring/Spring-Boo-Bean-Validation-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90/#)
