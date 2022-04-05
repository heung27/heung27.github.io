---
layout: post
title: Functional Interface
date: 2022-02-27 21:50 +0900
categories: [Language, Java]
tags: [Java 8, Functional Interface, Abstract Method, Lambda, Method Reference]
---



Java는 함수형 프로그래밍 언어가 아닌 객체지향 언어이기 때문에 함수는 일급 객체가 아니다. 그러나 Java 8에서 추가된 Functional Interface(이하 함수형 인터페이스)를 사용하면 일급 객체처럼 다룰 수 있다.

본문에서 함수형 인터페이스에 대해 자세히 알아보자.

> **일급 객체(First-class citizen)란?**
>
> 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체이다. 
>
> 즉, 아래의 조건을 만족하는 객체를 말한다.
>
> 1. 변수에 할당할 수 있다.
> 2. 함수를 인자로 전달받을 수 있다.
> 3. 함수의 결과로서 리턴될 수 있다.

<br>

## 함수형 인터페이스란?

추상 메서드가 하나만 존재하는 인터페이스를 말한다. 예를 들면 아래와 같은 인터페이스가 있다.

```java
public interface FunctionalInterface {
    public abstract int plus(int n1, int n2);
}
```

<br>

함수형 인터페이스를 사용하는 이유는 **자바의 람다식이 함수형 인터페이스로만 접근 가능**하기 때문이다.

```java
public interface FunctionalInterface {
  int plus(int n1, int n2); // public abstract 생략
}

FunctionalInterface func = (n1, n2) -> n1 + n2;
func.plus(1, 3)
```

이렇게 람다식으로 생성된 순수 함수를 변수로 선언할 수 있게 되어 더욱 간결한 코드를 작성할 수 있다.

> **순수 함수란?**
>
> 부수효과가 없는 함수이다. 
>
> 즉, 아래의 조건을 만족하는 함수를 말한다.
>
> - 동일한 인수가 전달되면 언제나 동일한 값을 반환하는 함수
> - 외부 상태에 의존하지 않는 함수
> - 외부의 상태를 변경하지 않는 함수

<br>

## 생성

`@FunctionalInterface`를 사용해 만들 수 있다. 이 어노테이션은 해당 인터페이스가 함수형 인터페이스 조건에 맞는지 검사해 준다.

그런데 위의 예제에서는 `@FunctionalInterface`이 없었지만 제대로 동작했다. `@FunctionalInterface` 이 없어도 조건에 맞는다면 함수형 인터페이스로 동작하고 사용하는 데는 문제없다. 하지만 **함수형 인터페이스 검증과 유지 보수를 위해 꼭 붙여주도록 하자.**

```java
@FunctionalInterface
public interface CustomInterface<T> {
    T call();

    default void printDefault() {
        System.out.println("It is default method");
    }

    static void printStatic() {
        System.out.println("It is static method");
    }
}

```

이 예제에서는 메서드가 여러 개 선언되었다. 함수형 인터페이스는 메서드가 한 개라고 하지 않았나?

오해하지 말자. 

**함수형 인터페이스는 '추상' 메서드가 하나이다.**  default method 또는 static method 는 여러 개 존재해도 상관 없다. 위의 예제에서는 `call()` 이라는 abstract method가 하나 그리고 default method와 static method가 하나씩 선언 되었다.

<br>

## 사용

```java
CustomInterface<String> func = () -> "It is abstract method";

System.out.println(func.call());
func.printDefault();
CustomInterface.printStatic();

/* 실행 결과
It is abstract method
It is default method
It is static method
*/
```

선언했던 `call()`이라는 추상 메서드가 람다식 `() -> "It is abstract method"`과 매핑된다.

주의해야 할 점은 반환형과 매개변수이다. 서로가 매핑되어야 하기 때문에 **추상 메서드와 람다식의 매개변수, 반환형이 같아야 한다.**

<br>

## Java에서 제공하는 함수형 인터페이스

매번 함수형 인터페이스를 만들어서 사용하는 것은 상당히 번거로운 일이다. Java에는 자주 사용될 것 같은 함수형 인터페이스가 이미 정의되어 있다. 

- Supplier\<T>
- Consumer\<T>
- Function\<T, R>
- Predicate\<T>

이외에도 다양한 함수형 인터페이스가 제공된다. 자세한 정보는 패키지 [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)에 정의되어 있다.

<br>

이제 위의 함수형 인터페이스를 하나씩 살펴보자. 아래의 코드들은 실제 구현되어 있는 코드이다.

### 1. Supplier\<T>

```java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {
  
    T get();
}
```

Supplier는 매개변수를 가지지 않고 반환값이 Generic 타입의 객체인 추상 메서드 `get`이 정의되어 있다.



#### 사용 예시

```java
Supplier<String> supplier = () -> "Hello Supplier";

System.out.println(supplier.get());

/* 실행 결과
Hello Supplier
*/
```

공급자라는 이름처럼 아무런 인자를 받지 않고 특정 객체를 리턴한다.

<br>

### 2. Consumer\<T>

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {
  
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

Consumer는 매개변수로 Generic 타입의 객체를 가지고 반환값은 없는 추상 메서드 `accept`가 정의되어 있다. 추가로 `andThen`이라는 default method를 제공하는데 이는 하나의 Consumer가 처리된 후 연쇄적으로 다음 Consumer가 동작하게 한다. 



#### 사용 예시

```java
Consumer<String> consumer = str -> System.out.println(str.split(" ")[0]);
Consumer<String> after = str -> System.out.println(str);

consumer.andThen(after).accept("Hello Consumer");

/* 실행 결과
Hello
Hello Consumer
*/
```

먼저 `accept`를 통해 전달받은 인자로 `consumer`를 처리하고, 다음으로 두 번째 Consumer인 `after`를 처리한다. 

<br>

### 3. Function\<T, R>

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

Function은 매개변수와 반환형으로 Generic 타입의 객체를 가지는 `apply`가 정의되어 있다. 또한 Consumer와 마찬가지로 `andThen`이 제공되고 추가적으로 `compose`와 `identity`가 제공된다. `compose`는 `andThen`과 반대로 첫 번째 Function이 실행되기 이전에 인자로 받은 Function을 먼저 처리한다. `identity`는 자기 자신을 반환하는 static method이다.

#### 사용 예시

```java
Function<String, Integer> function = str -> str.length();
Function<String, String> before = str -> str.split(" ")[0];
Function<Integer, Integer> after = num -> num * num;

int result = function.compose(before).andThen(after).apply("Hello Function");
System.out.println(result);

/* 실행 결과
25
*/
```

`compose`를 통해 첫 번째 `function`이 처리되기 전에 `before`가 먼저 처리된다. 다음으로 `function`이 실행되고, 마지막으로 `andThen`으로 받은 `after`가 처리된다. (Hello Function => Hello => 5 => 25)

<br>

### 4. Predicate\<T>

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }

    @SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```

Predicate는 매개변수로 Generic 타입의 객체를 가지고 반환값으로 `boolean`을 반환하는 `test`가 정의되어 있다. 또한 추가적으로 `and, negate, or, isEqual, not`을 제공한다. 각 메서드는 논리 연산과 비교 연산을 구현하고 있다. 

#### 사용 예시

```java
Predicate<Integer> isBiggerThanFive = num -> num > 5;
Predicate<Integer> isLowerThanFive = num -> num < 5;

boolean result1 = isBiggerThanFive.and(isLowerThanFive).test(10);
boolean result2 = isBiggerThanFive.or(isLowerThanFive).test(10);
System.out.println(result1);
System.out.println(result2);

/* 실행 결과
false
true
*/
```

`test`를 통해 10과 5를 비교하고 `and` 연산과 `or` 연산을 적용해 보았다.

<br>

## Reference

- [https://mangkyu.tistory.com/113](https://mangkyu.tistory.com/113)
- [https://zzang9ha.tistory.com/303](https://zzang9ha.tistory.com/303)
- [https://bcp0109.tistory.com/313](https://bcp0109.tistory.com/313)
- [https://codechacha.com/ko/java8-functional-interface/](https://codechacha.com/ko/java8-functional-interface/)
- [https://jeong-pro.tistory.com/23](https://jeong-pro.tistory.com/23)
