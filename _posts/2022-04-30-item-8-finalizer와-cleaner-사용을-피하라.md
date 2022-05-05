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





<br>

## 1. finalizer와 cleaner란?



<br>

## 2. finalizer와 cleaner의 문제점

- 즉시 수행된다는 보장이 없다.
- 수행 여부를 보장하지 않는다.
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
- 성능 문제를 동반한다.
- finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다.
- 그래서 AutoCloseable, try-with-resource을 사용하자

<br>

## 3. finalizer와 cleaner가 유효한 경우

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
