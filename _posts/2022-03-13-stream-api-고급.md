---
layout: post
title: Stream API - 고급
date: 2022-03-13 16:50 +0900
categories: [Language, Java]
tags: [Java 8, Stream API, Functional Interface, Lambda, Method Reference]
---



지난번의 기본편에 이어 고급편을 준비했다. 본편에서는 조금 더 복잡한 Stream API 메서드들과 성능, 주의할 점에 대해 자세히 알아본다. 



### 1. FlatMap

처리해야 하는 데이터가 2차원 배열 또는 2차원 리스트일 경우, 이를 1차원으로 처리해야 한다면 어떻게 해야할까? map을 이용한다고 해도 2중 Stream의 형태로 처리될 것이다. 이렇게 중첩 구조를 한 단계 제거하기 위해 사용되는 중간 연산이 flatMap이다. 



```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```



```java
```



<br>

### 2. Reduce



<br>

### 3. Null-safe Stream



<br>

### 4. Parallel Stream



<br>

### 5. 실행 순서



<br>

### 6. 지연 처리



<br>

### 7. 줄여쓰기



<br>

### Reference

- [https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)
- [https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)
- [https://hbase.tistory.com/171](https://hbase.tistory.com/171)
- [https://12bme.tistory.com/461](https://12bme.tistory.com/461)
- [https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API](https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API)
