---
layout: post
title: Functional Interface
date: 2022-02-27 21:50 +0900
categories: [Language, Java]
tags: [Java 8, Functional Interface, Abstract Method, Lambda]
---



Java는 함수형 프로그래밍 언어가 아닌 **객체지향 언어**이기 때문에 함수는 **일급 객체**가 아니다. 그러나 **Java 8에서 추가된 Functional Interface(이하 함수형 인터페이스)를 사용하면 일급 객체처럼 다룰 수 있다.** 본문에서 함수형 인터페이스에 대해 자세히 알아보자.

> **일급 객체(First-class citizen)란?**
>
> 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 말한다.
>
> 쉽게 말해 아래의 조건을 만족하는 객체이다.
>
> 1. 변수에 할당할 수 있다.
> 2. 함수를 인자로 전달 받을 수 있다.
> 3. 함수의 결과로서 리턴될 수 있다.

<br>

## 함수형 인터페이스란?

추상 메서드가 하나만 존재하는 인터페이스를 말한다. 예를 들면 아래와 같은 인터페이스가 있다.

```java
public interface FunctionalInterface {
     public abstract void doSomething(String text);
}
```

<br>

함수형 인터페이스를 사용하는 이유는 **자바의 람다식이 함수형 인터페이스로만 접근 가능**하기 때문이다.

```java
public interface FunctionalInterface {
     public abstract void doSomething(String text);
}

FunctionalInterface func = text -> System.out.println(text);
func.doSomething("do something");
// 실행 결과
// do something
```

이렇게 함수형 람다식으로 생성된 **순수 함수**를 변수로 선언할 수 있게되어 더욱 간결하게 코드를 작성할 수 있다.

> 순수 함수란?
>
> 

<br>

## 생성

@FunctionalInterface를 사용해 만들 수 있다. 이 어노테이션은 해당 인터페이스가 함수형 인터페이스 조건에 맞는지 검사해줍니다.

`@FunctionalInterface` 어노테이션이 없어도 함수형 인터페이스로 동작하고 사용하는 데 문제는 없지만, 인터페이스 검증과 유지보수를 위해 붙여주는 게 좋습니다.

```java
@FunctionalInterface
interface CustomInterface<T> {
    // abstract method 오직 하나
    T myCall();

    // default method 는 존재해도 상관없음
    default void printDefault() {
        System.out.println("Hello Default");
    }

    // static method 는 존재해도 상관없음
    static void printStatic() {
        System.out.println("Hello Static");
    }
}
```

여기서 메서드가 여러개 선언되었는데 왜 함수형 인터페이스라고 하는 거지라는 의문이 들 수 있다. 

추상 메서드가 하나라는 거지  `default method` 또는 `static method` 는 여러 개 존재해도 상관 없다. 위의 예제에서는myCall 이라는 abstract method가 하나 그리고 default method와 static method가 하나씩 선언 되었다.

<br>

## 사용

```java
CustomInterface<String> customInterface = () -> "Hello Custom";

// abstract method
String s = customInterface.myCall();
System.out.println(s);

// default method
customInterface.printDefault();

// static method
CustomFunctionalInterface.printStatic();
```

선언했던 myCall 이라는 abstract method가 람다식 `() -> "Hello Custom"`과 매핑된다.

주의해야 할 점은 abstract method의 반환형과 매개변수이다. ~~ 에러가 발생한다. 

<br>

## Java에서 제공하는 함수형 인터페이스

매번 함수형 인터페이스를 만들어서 사용하는 것은 상당히 번거로운 일이다. Java에는 자주 사용될 것 같은 함수형 인터페이스가 이미 정의되어 있다. 

- Supplier\<T>
- Consumer\<T>
- Function\<T, R>
- Predicate\<T>

이외에도 다양한 함수형 인터페이스가 제공된다. [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) 패키지에 정의되어있으니 참고하자.

<br>

이제 위의 함수형 인터페이스를 하나씩 살펴보자.

### 1. Supplier\<T>

```java

```



### 2. Consumer\<T>

```java
```



### 3. Function\<T, R>

```java
```



### 4. Predicate\<T>

```java
```



<br>

## Refference

- https://mangkyu.tistory.com/113
- https://zzang9ha.tistory.com/303
- https://bcp0109.tistory.com/313
- https://codechacha.com/ko/java8-functional-interface/
