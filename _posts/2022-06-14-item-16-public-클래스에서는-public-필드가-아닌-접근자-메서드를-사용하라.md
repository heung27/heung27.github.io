---
layout: post
title: Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
date: 2022-06-14 17:12 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Getter]
---



Effective Java의 열여섯 번째 아이템 "public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

이번 주제는 이전 포스팅과 상당히 밀접한 내용입니다. 이전에 public 클래스의 인스턴스는 되도록 public이 아니어야 한다고 했는데요. 여기에 초점을 맞춰 자세히 알아보겠습니다.

<br>

## 1. public 클래스의 필드

종종 인스턴스 필드를 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있습니다. 예를 들어 java.awt 패키지의 Point 클래스가 있습니다. 

```java
package java.awt;

public class Point extends Point2D implements java.io.Serializable {
  
    public int x;
    public int y;
  
    ...
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못합니다. API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없습니다.

이를 해결하기 위해 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가합니다.

```java
public class Point {

    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }
}
```

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있습니다. 

<br>

만약 public 클래스의 필드가 불변이라면 어떨까요? 직접 노출될 때의 단점이 조금은 줄어들겠지만, 여전히 좋은 생각은 아닙니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전합니다. 단, 불변식은 보장할 수 있게 됩니다.

<br>

## 2. 예외 상황

위에서는 public 클래스에 대해서만 이야기했습니다. package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없습니다. 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 됩니다.

이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔합니다. 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐입니다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있습니다.

<br>

## 3. 핵심 정리

- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
- package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.

<br>

## 4. Related Posts

- 캡슐화 (Item 15)
