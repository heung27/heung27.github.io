---
layout: post
title: Item 9. try-finally보다는 try-with-resources를 사용하라 (미완성)
date: 2022-05-11 21:45 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, finalizer, cleaner]
---



Effective Java의 아홉 번째 아이템 "try-finally보다는 try-with-resources를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Java 라이브러리에는 close 메서드를 호출해 직접 닫아줘야하는 자원이 있습니다. 예를 들어 InputStream, OutputStream, java.sql.Connection 등이 있죠. 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 합니다. 이런 자원 중 상당수가 안정망으로 finalizer를 활용하고는 있지만 finalizer는 그리 믿을만하지 못합니다. 

<br>

## 1. try-finally

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였습니다. 

```java
```



<br>

## 2. try-with-resources





<br>

## 3. 핵심 정리

꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.

<br>

## 4. Related Posts

- finalizer (Item 8)
