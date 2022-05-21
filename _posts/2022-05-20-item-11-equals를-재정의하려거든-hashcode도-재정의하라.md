---
layout: post
title: Item 11. equals를 재정의하려거든 hashCode도 재정의하라 (미완성)
date: 2022-05-20 17:00 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Override]
---



Effective Java의 열한 번째 아이템 "equals를 재정의하려거든 hashCode도 재정의하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며



<br>

## 0. 핵심 정리

- equals를 재정의 할 때는 hashCode도 반드시 재정의해야 한다.
- 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 한다.
- 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
- AutoValue 프레임워크를 사용하면 euqals와 hashCode를 자동으로 만들어준다(IDE들도 이런 기능을 일부 제공한다).

<br>

## 0. Related Posts

<br>

- 
