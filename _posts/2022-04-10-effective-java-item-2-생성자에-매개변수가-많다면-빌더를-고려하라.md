---
layout: post
title: "[Effective Java] Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라"
date: 2022-04-10 18:03 +0900
categories: [Book, Java]
tags: [Effective Java, static method]
---



생성자와 정적 팩터리는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 제약이 있다. 본문에서 이를 해결하기 위한 세 가지 대안을 제시하고, 그중 일반적으로 가장 효과적인 **빌더 패턴**에 대해 자세히 알아본다.

<br>

## 대안 1. 점층적 생성자 패턴

필수 매개 변수만 받는 생성자, 선택 매개변수 1개를 함께 받는 생성자, 선택 매개변수 2개를 함께 받는 생성자, ... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.

다음은 필수 매개변수 2개와 선택 매개변수 3개를 받는 User 클래스의 점층적 생성자 패턴 예제이다.

```java
public class User {

		// 필수 매개변수
    private final String name;
    private final int age;

		// 선택 매개변수
    private final String address;
    private final String phone;
    private final String email;

    public User(String name, int age) {
        this(name, age, null, null, null);
    }

    public User(String name, int age, String address) {
        this(name, age, address, null, null);
    }

    public User(String name, int age, String address, String phone) {
        this(name, age, address, phone, null);
    }

    public User(String name, int age, String address, String phone, String email) {
        this.name = name;
        this.age = age;
        this.address = address;
        this.phone = phone;
        this.email = email;
    }
}
```

이 클래스의 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다. 위 예제 코드는 깔끔하고 괜찮아 보이지만 한계가 분명히 존재한다. 매개변수의 개수가 많아질수록 클라이언트 코드를 작성하거나 읽기 어려워진다는 것이다. 코드를 읽을 때 각 값의 이미가 무엇인지 헷갈릴 것이고, 매개변수가 몇 개인지도 주의해서 세어 보아야 할 것이다. 타입이 같은 매개변수가 연달아 늘어져 있으면 찾기 어려운 버그로 이어질 가능성도 있다.

<br>

## 대안 2. 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

다음은 위의 예제와 동일한 User 클래스에 자바빈즈 패턴을 적용한 예제이다.

```java
public class User {

    private String name;
    private int age;
    private String address;
    private String phone;
    private String email;
    
    public User() {}

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

기본 생성자를 통해 인스턴스를 생성하고 setter 메서드를 사용해 값을 넣어줘 인스턴스를 완성한다. 이러한 자바빈즈 패턴은 점층적 생성자 패턴의 단점을 보완했다. 인스턴스를 만들기 쉽고 읽기 쉬운 코드가 되었다.

하지만 자바빈즈 패턴에도 역시 한계가 존재한다. 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 때문에 클래스를 불변으로 만들 수 없으며 스레드의 안정성을 얻기 위해서는 프로그래머의 추가 작업이 필요하다.

<br>

## 대안 3. 빌더 패턴

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개 변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 그다음 빌더 객체가 제공하는 일종의 setter 메서드로 원하는 선택 매개변수들을 설정한다. 마지막으로 매개변수가 없는 build 메서드를 호출해 필요한 객체(보통은 불변)를 얻는다.

다음은 계층적으로 설계된 클래스에서 빌더를 활용한 예제이다. 추상 클래스인 Pizza는 추상 빌더를, 구체 클래스인 ChicagoPizza, HawaiianPizza는 구체 빌더를 선언했다.

```java
public abstract class Pizza {

    public enum Topping { HAM, MUSHROOM, ONION, PERPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {

        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

				// 하위 클래스는 이 메서드를 재정의하여 this를 반환한다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

```java
public class ChicagoPizza extends Pizza {

    public enum Size { SMALL, MEDIUM, LARGE }
    private Size size;

    public static class Builder extends Pizza.Builder<Builder> {

        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public ChicagoPizza build() {
            return new ChicagoPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private ChicagoPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```java
public class HawaiianPizza extends Pizza {

    private final boolean pineapple;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean pineapple = true;

				public Builder() {}				

        public Builder pineappleOut() {
            pineapple = false;
            return this;
        }

        @Override
        public HawaiianPizza build() {
            return new HawaiianPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private HawaiianPizza(Builder builder) {
        super(builder);
        pineapple = builder.pineapple;
    }
}
```

시카고 피자의 빌더는 크기(size) 매개변수를 필수로 받고, 하와이안 피자의 빌더는 필수 매개변수는 없지만 pineappleOut 메서드을 통해 파인애플을 뺄지 결정할 수 있다. 그리고 두 빌더 모두 Pizza.Builder를 상속받았기 때문에 addTopping을 통해 토핑을 추가할 수 있다. 참고로 빌더의 build 메서드는 해당하는 구체 하위 클래스를 반환하고, self 메서드는 메서드 체이닝을 위해 this(빌더)를 반환한다.

<br>

```java
// 빌더 사용 예제
ChicagoPizza chicagoPizza = new ChicagoPizza.Builder(LARGE)
        .addTopping(HAM).addTopping(SAUSAGE)
        .build();

HawaiianPizza hawaiianPizza = new HawaiianPizza.Builder()
        .addTopping(ONION).addTopping(MUSHROOM)
        .pineappleOut()
        .build();
```

빌더 패턴은 점층적 생성자 패턴과 자바빈즈 패턴의 단점을 모두 보완했다. 메서드 체인을 사용해 객체를 생성하여 **일관성을 해치지 않으며 작성하기 쉽고 가독성이 좋다.** 추가적으로 위의 예제처럼 공변 반환 타이핑을 이용하면 형변환에 신경 쓰지 않고 빌더를 사용할 수 있기 때문에 **계층적으로 설계된 클래스와 함께 쓰기에도 좋다.**

하지만 빌더 패턴에 장점만 있는 것은 아니다. **객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수도 있다.** 또한 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 하지만 API의 확장성을 고려하여 애초에 빌더로 시작하는 편이 나을 때가 많다.

<br>

> **공변 반환 타이핑(covariant return typing)** 
>
> 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 것을 의미한다.

