---
layout: post
title: Item 12. toString을 항상 재정의하라 (미완성)
date: 2022-05-21 11:52 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Override, toString]
---



Effective Java의 열두 번째 아이템 "toString을 항상 재정의하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며



<br>

## 1. 재정의를 해야하는 이유

toString을 잘 구현한 클래스는 사용하기에 훨씬 편하고, 그 클래스를 사용한 시스템은 디버깅하기 쉽습니다.

<br>

## 2. 작성 요령

- toString은 그 객체가 가진 주요 정보 모두를 반환하는 것이 좋다.
- 포맷을 명시하든 아니든 작성자의 의도는 명확히 밝혀야 한다.
- 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

<br>

정적 유틸리티 클래스는 toString을 제공할 이유가 없다. 또한, 대부분의 열거 타입도 자바가 이미 완벽한 toString을 제공하니 따로 재정의하지 않아도 된다. 하지만 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해줘야 한다. 예를 들어 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속해 쓴다.

## 3. 핵심 정리

- 모든 구체 클래스에서 Object의 toString을 재정의하자.
- toString을 재정의한 클래스는 시스템을 디버깅하기 쉽게 해준다.
- toString은 해당 객체의 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.

<br>

## 4. Related Posts

- [정적 유틸리티 클래스 (Item 4)](https://heung27.github.io/posts/item-4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- 열거 타입 (Item 34)

