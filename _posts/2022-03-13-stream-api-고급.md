---
layout: post
title: Stream API - 고급
date: 2022-03-13 16:50 +0900
categories: [Language, Java]
tags: [Java 8, Stream API, Functional Interface, Lambda, Method Reference]
---



지난번의 기본편에 이어 고급편을 준비했다. 본편에서는 조금 더 복잡한 Stream API 메서드들과 성능, 주의할 점에 대해 자세히 알아본다. 



## 1. FlatMap

처리해야 하는 데이터가 2차원 배열 또는 2차원 리스트일 경우, 이를 1차원으로 처리해야 한다면 어떻게 해야할까? map을 이용한다고 해도 2중 Stream의 형태로 처리될 것이다. 이렇게 중첩 구조를 한 단계 제거하기 위해 사용되는 중간 연산이 flatMap이다. 

인자로 반환형이 Stream인 함수형 인터페이스 Function을 받는다. 즉, flatmap은 중첩된 Stream 구조에서 한 단계 제거하여 새로운 Stream을 생성하고 반환하는 것이다. 이런 작업을 '플래트닝(Flattening)'이라고 한다. 

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

```java
// 2차원 배열
String[][] strings = {
  {"a", "b", "c"},
  {"d", "e", "f"}
};

Stream<String> stringStream1 = Arrays.stream(strings)
  .flatMap(Arrays::stream);

// 2중 리스트
List<List<String>> listList = Arrays.asList(
  Arrays.asList("a", "b", "c"),
  Arrays.asList("d", "e", "f")
);

Stream<String> stringStream2 = listList.stream()
  .flatMap(Collection::stream);
```

2차원 배열과 2중 리스트를 1차원 Stream으로 Flattening했다. Stream의 요소는 `["a", "b", "c", "d", "e", "f"]`가 된다. 

<br>

### FlatMap 활용 예제

이름과 책 리스트를 가지는 Developer 클래스가 있다. 

```java
public class Developer {
  
  private String name;
  private Set<String> books;
  
  public void addBook(String book) {
    if (this.books == null) {
      this.books = new HashSet<>();
    }
    this.books.add(book);
  }
  
  // Getter, Setter, ToString 생략
}
```

Developer 객체들에 저장된 모든 book 리스트에서 'python'을 포함하지 않는 책들을 반환하고자 한다. 

```java
Developer o1 = new Developer();
o1.setName("heung");
o1.addBook("Java 8 in Action");
o1.addBook("Spring Boot in Action");
o1.addBook("Effective Java (3nd Edition)");

Developer o2 = new Developer();
o2.setName("tack");
o2.addBook("Learning Python, 5th Edition");
o2.addBook("Effective Java (3nd Edition)");

List<Developer> list = Arrays.asList(o1, o2);

Set<String> collect = list.stream() // Stream<Developer>
  .map(Developer::getBook) // Stream<Set<String>>
  .flatMap(Collection::stream) //Stream<String>
  .filter(x -> !x.toLowerCase().contains("python"))
  .collect(Collectors.toSet());

collect.forEach(System.out::println);

/* 실행 결과
Spring Boot in Action
Effective Java (3nd Edition)
Java 8 in Action
*/
```

먼저 Developer List Stream에서 map을 사용해 books(Set)를 꺼낸다. 다음 flatMap을 사용해 Set Stream에서 String Stream으로 변환하고 'python'을 포함하는 책을 필터링한 결과를 Set으로 반환한다.(같은 책 중복 제거)

<br>

## 2. Reduce

누산기(Accumulator)와 연산(Operation)으로 컬렉션에 있는 값을 처리하여 더 작은 컬렉션이나 단일 값을 만드는 작업이다.

reduce 메서드는 여러 요소들을 통해 새로운 결과를 만들어내는데, 최대 3가지의 매개변수를 받을 수 있다. 

- Accumulator : 각 요소를 계산한 중간 결과를 생성
- Identity : 계산을 처리하기 위한 초기값
- Combiner : Parlallel Stream에서 나누어 계산한 결과를 하나로 합치기 위한 로직



### 2.1 reduce(accumulator)

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
```

BinaryOperator 타입의 accumulator를 매개변수로 받는다. BinaryOperator\<T>는 같은 타입의 인자 2개를 받아 동일한 타입의 결과를 반환하는 함수형 인터페이스이다. 

accumulator를 수행하고 Optional 타입의 계산 결과를 반환한다. Stream이 비어있을 경우 값을 계산할 수 없기 때문에 Optional을 반환하는 것이다. 

```java
IntStream intStream = IntStream.range(1, 5); // 1, 2, 3, 4

OptionalInt optionalInt = intStream.reduce(Integer::sum);
System.out.println(optionalInt.orElse(0));

/* 실행 결과
10
*/
```

<br>

### 2.2 reduce(identity, accumulator)

```java
T reduce(T identity, BinaryOperator<T> accumulator);
```

Generic 타입의 identity와 BinaryOperator를 매개변수로 받는다. identity는 계산을 처리하기 위한 초기값을 의미한다.

매개변수 1개를 갖는 reduce와 달리 초기값이 세팅되기 때문에 Optional이 아닌 Generic 타입의 결과를 반환한다.

```java
IntStream intStream = IntStream.range(1, 5); // 1, 2, 3, 4

int result = intStream.reduce(10, Integer::sum);
System.out.println(result);

/* 실행 결과
20
*/
```



<br>

### 2.3 reduce(identity, accumulator, combiner)

```java
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
```

identity와 BiFunction 타입의 accumulator, BinaryOperator타입의 combiner를 매개변수로 받는다. 

BiFunction은 2개의 파라미터 타입과 1개의 반환형 모두 서로 다른 Generic 타입을 가질 수 있다. 

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
  R apply(T t, U u);
  
  default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
  }
}
```

하지만 BinaryOperator는 BiFunction을 구현(implements)하여 2개의 파라미터 타입과 1개의 반환형 모두 같은 Generic 타입을 갖는다.

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
  
  public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
    Objects.requireNonNull(comparator);
    return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
  }

  public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
    Objects.requireNonNull(comparator);
    return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
  }
}
```

새롭게 추가된 BinaryOperator타입의 combiner는 병렬 처리 시에 각 쓰레드에서 만들어진 결과를 합치는 작업을 수행한다. 때문에 combiner을 추가하여도 ParallelStream으로 실행하지 않으면 combiner은 호출되지 않는다. 

```java
int result = Stream.of(1, 2, 3)
  .reduce(10, Integer::sum, (a, b) -> {
    System.out.println("combiner was called");
    return a + b;
  });

System.out.println(result);

/* 실행 결과
16
*/
```

위의 예제는 Parallel Stream이 아니기 때문에 `combiner was called` 가 출력되지 않는다. 그러면 이제 

```java
int result = Stream.of(1, 2, 3)
  .parallel()
  .reduce(10, Integer::sum, (a, b) -> {
    System.out.println("combiner was called");
    return a + b;
  });

System.out.println(result);

/* 실행 결과
36
*/
```



```java
int result = 10 + Stream.of(1, 2, 3)
  .parallel()
  .reduce(0, Integer::sum, (a, b) -> {
    System.out.println("combiner was called");
    return a + b;
  });

System.out.println(result);

/* 실행 결과
16
*/
```



<br>

## 3. Null-safe Stream



<br>

## 4. Parallel Stream



<br>

## 5. 실행 순서



<br>

## 6. 지연 처리



<br>

## 7. 줄여쓰기



<br>

## Reference

- [https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)
- [https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)
- [https://hbase.tistory.com/171](https://hbase.tistory.com/171)
- [https://12bme.tistory.com/461](https://12bme.tistory.com/461)
- [https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API](https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API)
- [https://mkyong.com/java8/java-8-flatmap-example/](https://mkyong.com/java8/java-8-flatmap-example/)
