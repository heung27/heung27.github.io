---
layout: post
title: Item 8. finalizer와 cleaner 사용을 피하라 (미완성)
date: 2022-04-30 14:37 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, finalizer, cleaner]
---



Effective Java의 여덟 번째 아이템 "finalizer와 cleaner 사용을 피하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Java는 finalizer와 cleaner라는 두 가지 **객체 소멸자**를 제공합니다. 객체 소멸자란 ~~입니다. 본문에서 finalizer와 cleaner가 무엇이고 어떤 문제점을 가지고 있는지, 그리고 어떤 쓰임새가 있는지 자세히 알아보겠습니다. 



<br>

## 1. finalizer와 cleaner란?



### finalizer

```java
public class Room {
  
    ...

    @Override
    protected void finalize() throws Throwable {
        System.out.println("run finalizer");
    }
}
```



```java
new Room();
System.gc();
```



finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요합니다. 나름의 쓰임새가 몇 가지 있긴 하지만 기본적으로 쓰지 말아야 합니다. 그래서 Java 9에서는 finalizer를 deprecated API로 지정하고 cleaner를 그 대안으로 소개했습니다. 



### Cleaner

```java
public class RoomCleaner {

    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("run cleaner");
            numJunkPiles = 0;
        }
    }

    public RoomCleaner(int numJunkPiles) {
        State state = new State(numJunkPiles);
        cleaner.register(this, state);
    }
}
```



```java
new Room(99);
System.gc();
```



cleaner은 finalizer 보다 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요합니다.

<br>

## 2. finalizer와 cleaner의 문제점

- 즉시 수행된다는 보장이 없다.
- 수행 여부를 보장하지 않는다.
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
- 성능 문제를 동반한다.
- finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다.
- 그래서 AutoCloseable, try-with-resource을 사용하자

<br>

## 3. finalizer와 cleaner의 쓰임새

- 쓰임새 1 - 안전망
- 쓰임새 2 - 네이티브 피어



<br>

## 4. 사용 예제

- 예제
- 중첩 클래스 - 바깥 객체 참조 주의
- 예제 비교



<br>

## 5. 핵심 정리

cleaner(Java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.

<br>

## 6. Related Posts

- try-with-resource (Item 9)
- 중첩 클래스 (Item 24)
