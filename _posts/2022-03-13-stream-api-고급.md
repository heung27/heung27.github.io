---
layout: post
title: Stream API - 고급
date: 2022-03-13 16:50 +0900
categories: [Language, Java]
tags: [Java 8, Stream API, Functional Interface, Lambda, Method Reference]
---



지난번의 [기본편](https://heung27.github.io/posts/stream-api-%EA%B8%B0%EB%B3%B8/)에 이어 고급편을 준비했다. 본편에서는 조금 더 복잡한 Stream API 메서드들과 성능, 주의할 점에 대해 자세히 알아본다. 

<br>

## 1. Parallel Stream

Stream API는 많은 양의 데이터를 처리하는 경우 런타임 성능을 높이기 위해 병렬 스트림(Parallel Stream)을 제공한다. Parallel Stream는 내부적으로 Java 7에서 도입된 Fork & Join을 사용하고 있다.

```java
List<String> strings = Arrays.asList("a", "b", "c", "d");

// 병렬 스트림 생성
Stream<String> parallelStream = strings.parallelStream();

// 병렬 여부 확인
System.out.println(parallelStream.isParallel()); // true
```

Parallel Stream을 생성하기 위해서는 Collection의 메서드 `parallelStream()`을 사용하고  `isParallel()`메서드를 통해 병렬 여부를 확인할 수 있다. 이외에도 일반적인 순차 Stream으로 진행하던 중  `parallel()` 메서드를 사용해 일부 연산만을 병렬로 처리하게 할 수 있다. 

예제를 통해 어떻게 동작하는지 알아보자.

```java
Arrays.asList("a", "b", "c", "d", "e", "f", "g", "h")
  .parallelStream()
  .map(str -> {
    System.out.format("map: %s [%s]\n", str, Thread.currentThread().getName());
    return str.toUpperCase();
  })
  .forEach(str -> {
    System.out.format("forEach: %s [%s]\n", str, Thread.currentThread().getName());
  });

/* 실행 결과
map: f [main]
map: a [ForkJoinPool.commonPool-worker-13]
map: d [ForkJoinPool.commonPool-worker-15]
map: c [ForkJoinPool.commonPool-worker-3]
map: b [ForkJoinPool.commonPool-worker-9]
map: g [ForkJoinPool.commonPool-worker-11]
map: h [ForkJoinPool.commonPool-worker-5]
forEach: H [ForkJoinPool.commonPool-worker-5]
map: e [ForkJoinPool.commonPool-worker-7]
forEach: G [ForkJoinPool.commonPool-worker-11]
forEach: B [ForkJoinPool.commonPool-worker-9]
forEach: C [ForkJoinPool.commonPool-worker-3]
forEach: D [ForkJoinPool.commonPool-worker-15]
forEach: A [ForkJoinPool.commonPool-worker-13]
forEach: F [main]
forEach: E [ForkJoinPool.commonPool-worker-7]
*/
```

실행 결과를 통해 각 Stream 연산을 어느 스레드가 수행했는지 확인할 수 있다. 물론 어떤 스레드가 어떤 작업을 수행할지 비결정적이기 때문에 실행에 따라 출력은 달라 질 수 있다.

Parallel Stream은 내부적으로 공용 ForkJoinPool을 사용하고  `ForkJoinPool.commonPool()`을 통해 사용 가능한 공용의 ForkJoinPool의 갯수를 확인할 수 있다. 해당 값은 사용 가능한 물리적인 CPU 코어 수에 따라 다르게 설정된다.

위 예제는 7개의 ForkJoinPool로 실행되었고 그에 따라 7개의 worker 스레드가 동작한 것을 확인할 수 있다.

```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
System.out.println(commonPool.getParallelism());

/* 실행 결과
7
*/
```

또한 이 값은 다음과 같은 JVM 매개변수를 통해 별도로 설정해 줄 수 있다.

```java
-Djava.util.concurrent.ForkJoinPool.common.parallelism=5
```

<br>

### Parallel Stream의 정렬

Parallel Stream에서 정렬이 어떻게 동작하는지 확인해보기 위해 위의 예제코드에 정렬 메서드 `sorted()`를 추가했다.

```java
Arrays.asList("a", "b", "c", "d", "e", "f", "g", "h")
  .parallelStream()
  .map(str -> {
    System.out.format("map: %s [%s]\n", str, Thread.currentThread().getName());
    return str.toUpperCase();
  })
  .sorted((s1, s2) -> {
    System.out.format("sort: %s ? %s [%s]\n", s1, s2, Thread.currentThread().getName());
    return s1.compareTo(s2);
  })
  .forEach(str -> {
    System.out.format("forEach: %s [%s]\n", str, Thread.currentThread().getName());
  });

/* 실행 결과
map: h [main]
map: a [ForkJoinPool.commonPool-worker-13]
map: e [ForkJoinPool.commonPool-worker-15]
map: f [ForkJoinPool.commonPool-worker-5]
map: c [ForkJoinPool.commonPool-worker-11]
map: b [ForkJoinPool.commonPool-worker-9]
map: d [ForkJoinPool.commonPool-worker-3]
map: g [ForkJoinPool.commonPool-worker-7]
sort: B ? A [main]
sort: C ? B [main]
sort: D ? C [main]
sort: E ? D [main]
sort: F ? E [main]
sort: G ? F [main]
sort: H ? G [main]
forEach: F [main]
forEach: A [ForkJoinPool.commonPool-worker-13]
forEach: C [ForkJoinPool.commonPool-worker-7]
forEach: D [ForkJoinPool.commonPool-worker-5]
forEach: B [ForkJoinPool.commonPool-worker-15]
forEach: E [ForkJoinPool.commonPool-worker-9]
forEach: G [ForkJoinPool.commonPool-worker-11]
forEach: H [ForkJoinPool.commonPool-worker-3]
*/
```

Parallel Stream은 비결정적이기 때문에 실행할 때마다 출력이 다르게 나온다. 하지만 위의 예제를 실행해 보면 map과 forEach는 출력이 바뀌는데 sort는 항상 main 스레드에서 순차적으로 처리되는 것을 확인할 수 있다. 

이러한 이유는 Parallel Stream sort의 내부 동작 방식에 있다. 정렬하고자 하는 배열의 길이가 임계값(프로세스에게 할당 가능한 배열의 길이)보다 작으면 순차적인 정렬 방식인 `Arrays.sort()`를 사용하고, 그보다 크면 `parallelSort()`를 사용하도록 되어있다. 따라서 위의 예제에서는 배열크기가 작아 순차적으로 처리된 것이다.

 <br>

### Parallel Stream에서 고려해야 할 사항

Parallel Stream을 이용하면 임의로 스레드 개수를 조정할 수 있어 작업 처리 속도 향상을 기대할 수 있다. 하지만 여기에도 고려해야 할 사항은 있다.

**1) ForkJoinPool의 특성상 나누어지는 job은 균등하게 처리되어야 한다.**

Parallel Stream은 작업을 분할하기 위해 Spliterator의 trySplit()을 사용하는데, 이 분할되는 작업의 단위가 균등하게 나누어져야 하며 나누어지는 작업에 대한 비용이 높지 않아야 순차적 방식보다 효율적으로 처리할 수 있다. Array,  ArrayList와 같이 정확한 전체 사이즈를 알 수 있는 경우에는 분할 처리가 빠르고 비용이 적게 들지만, LinkedList의 경우에는 별다른 효과를 찾기 어렵다.

**2) 병렬로 처리되는 작업이 독립적이지 않다면 수행 성능에 영향이 있을 수 있다.**

예를 들어 Stream의 중간 연산 중 sorted() 또는 distinct()와 같은 작업을 수행하는 경우에는 내부적으로 상태에 대한 변수를 각 작업들이 공유(synchronized)하게 되어 있다. 이러한 경우에는 순차적으로 실행하는 경우가 더 효과적일 수 있다.

<br>

그렇다면 Parallel Stream은 언제 사용해야 할까?

Parallel Stream은 앞서 설명한 ForkJoinPool 방식을 이용하기 때문에 분할이 잘 이루어질 수 있는 데이터 구조이거나, 작업이 독립적이면서 CPU 사용이 높은 작업에 적합하다고 볼 수 있다.

<br>

## 2. FlatMap

처리해야 하는 데이터가 2차원 배열 또는 2차원 리스트일 경우, 이를 1차원으로 처리해야 한다면 어떻게 해야 할까? map을 이용한다고 해도 2중 Stream의 형태로 처리될 것이다. 이렇게 중첩 구조를 한 단계 제거하기 위해 사용되는 중간 연산이 flatMap이다. 

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

2차원 배열과 2중 리스트를 1차원 Stream으로 Flattening 했다. Stream의 요소는 `["a", "b", "c", "d", "e", "f"]`가 된다. 

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

먼저 Developer List Stream에서 map을 사용해 books(Set)를 꺼낸다. 다음 flatMap을 사용해 Set Stream에서 String Stream으로 변환하고 'python'을 포함하는 책을 필터링한 결과를 Set으로 반환한다. (같은 책 중복 제거)

<br>

## 3. Reduce

누산기(Accumulator)와 연산(Operation)으로 컬렉션에 있는 값을 처리하여 더 작은 컬렉션이나 단일 값을 만드는 작업이다.

reduce는 여러 요소들을 통해 새로운 결과를 만들어내는데, 최대 3가지의 매개변수를 받을 수 있다. 

- Accumulator : 각 요소를 계산한 중간 결과를 생성
- Identity : 계산을 처리하기 위한 초기값
- Combiner : Parlallel Stream에서 나누어 계산한 결과를 하나로 합치기 위한 로직

### 3.1 reduce(accumulator)

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

### 3.2 reduce(identity, accumulator)

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

### 3.3 reduce(identity, accumulator, combiner)

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

새롭게 추가된 BinaryOperator타입의 combiner는 병렬 처리 시에 각 스레드에서 만들어진 결과를 합치는 작업을 수행한다. 때문에 combiner을 추가하여도 ParallelStream으로 실행하지 않으면 combiner은 호출되지 않는다. 

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

위의 예제는 Parallel Stream이 아니기 때문에 `combiner was called`가 출력되지 않는다. 다음 예제를 보자.

```java
int result = Stream.of(1, 2, 3)
  .parallel()
  .reduce(10, Integer::sum, (a, b) -> {
    System.out.println("combiner was called");
    return a + b;
  });

System.out.println(result);

/* 실행 결과
combiner was called
combiner was called
36
*/
```

parallel 메서드를 사용해 스트림이 병렬로 처리되도록 하고 reduce를 실행했다. 결과를 보면 combiner가 두 번 실행된 것을 확인할 수 있다. 그 이유는 Stream의 각 요소(1, 2, 3)가 병렬로 처리되고, 처리된 결과를 하나의 결과값으로 만드는 연산이 두 번 발생하기 때문이다. 

즉, 병렬로 처리된 11(10+1), 12(10+2), 13(10+3)에서 역순으로 13 + 12를 먼저 더하고 25 + 11을 더해 combiner가 총 2번 호출되며 최종적으로 36이라는 결과를 만들어 낸다. 초기값(10) 또한 병렬로 발생하는 것을 주의하자.

```java
int result = 10 + Stream.of(1, 2, 3)
  .parallel()
  .reduce(0, Integer::sum, (a, b) -> {
    System.out.println("combiner was called");
    return a + b;
  });

System.out.println(result);

/* 실행 결과
combiner was called
combiner was called
16
*/
```

초기값이 한 개만 필요할 경우, 위와같이 identity는 0을 넘기고 reduce의 계산된 결과값에 초기값을 더해야 한다.

여기서 또 주목해야 할 것은 병렬 스트림의 경우 combiner라는 추가 연산이 발생한다는 것이다. 때문에 간단한 작업에 병렬 연산을 적용하면 오리혀 처리 속도가 느려질 수 있음을 고려해야 한다. 

<br>

## 4. Null-Safe Stream

Java를 이용해 개발을 하다 보면 NPE(NullPointException)가 자주 발생한다. 물론 NPE를 방지하기 위해 null 여부를 검사하는 로직을 작성해 줄 수 있지만 이러한 코드는 가독성이 떨어진다.

이 문제를 해결하기 위해 Java 8에서부터는 Optional이라는 Wrapper 클래스를 제공한다. Stream API 역시 이 Optional의 도움을 받아 Null-Safe한 Stream을 생성할 수 있다. 

(Optional에 대해 잘 알지 못한다면 이전의 [Optional](https://heung27.github.io/posts/optional/) 포스팅을 참고하길 바란다.)

```java
public <T> Stream<T> collectionToStream(Collection<T> collection) {
  return Optional.ofNullable(collection)
    .map(Collection::stream)
    .orElseGet(Stream::empty);
}
```

위 코드는 컬렉션을 인자로 받아 Optional 객체로 만들고 Stream 생성 후 리턴하는 메서드이다. 만약 컬렉션이 null인 경우 Empty Stream을 반환한다.

```java
List<String> nullList = null;

// NPE 발생
nullList.stream()
  .map(String::length)
  .forEach((System.out::println)); // NPE

```

NPE가 발생하는 상황을 가정해보았다. null의 Stream을 생성하려고 하니 당연히 NPE가 발생한다.

```java
List<String> nullList = null;

// Empty Stream으로 처리
collectionToStream(nullList)
  .map(String::length)
  .forEach(System.out::println); // []
```

우리가 만든`collectionToStream()` 메서드를 사용하면 NPE가 발생하는 대신 Empty Stream으로 작업을 마칠 수 있다.

<br>

## 5. 실행 순서

Stream API는 실행 순서를 고려하지 않고 잘 못 사용하면 처리 속도의 지연을 야기할 수 있다. 때문에 작성한 Stream API 코드가 정확히 어떻게 동작하는지 이해하는 것이 중요하다.

```java
Stream.of("a", "b", "c", "d", "e", "f")
  .filter(str -> {
    System.out.println("filter: " + str);
    return true;
  })
  .forEach(str -> {
    System.out.println("forEach: " + str);
  });

/* 실행 결과
filter: a
forEach: a
filter: b
forEach: b
filter: c
forEach: c
filter: d
forEach: d
*/
```

실행 결과를 보면 모든 요소에 대해 filter가 진행되고 forEach가 실행되는 수평적 구조로 처리하는 것이 아니라, 각각의 요소에 대해 filter와 forEach가 먼저 수행되는 수직적 구조로 처리하는 것을 알 수 있다. 

```java
boolean result = Stream.of("a", "b", "c", "d") 
  .map(str -> {
    System.out.println("map: " + str);
    return str.toUpperCase();
  })
  .anyMatch(str -> {
    System.out.println("anyMatch: " + str);
    return str.startsWith("A");
  });

/* 실행 결과
map: a
anyMatch: A
*/
```

위의 예제는 Stream의 각 요소를 대문자로 변환하고, 변환된 데이터 중 "A"로 시작하는 문자열이 있는지 검사하는 로직이다. 만약 Stream의 연산이 수평적 구조로 처리된다면 위의 결과는 map 4번(a, b, c, d) + anyMatch 1번(a) 실행될 것이다. 하지만 실제로 연산은 수직적 구조로 처리되기 때문에 map 1번 + anyMatch 1번 실행된 것을 확인할 수 있다. 

이러한 처리 방식은 연산의 실행 순서에 따라 전체 연산의 수가 달라지고, 이는 곧 성능에 영향을 미치게 된다. 다음 예제를 통해 자세히 알아보자.

```java
Stream.of("a", "b", "c", "d")
  .map(str -> {
    System.out.println("map: " + str);
    return str.toUpperCase();
  })
  .filter(str -> {
    System.out.println("filter: " + str);
    return str.startsWith("A");
  })
  .forEach(str -> {
    System.out.println("forEach: " + str);
  });

/* 실행 결과
map: a
filter: A
forEach: A
map: b
filter: B
map: c
filter: C
map: d
filter: D
*/
```

실행 결과를 보면, map과 filter는 Stream의 요소 수 만큼(4번) 호출되었고 forEach는 1번만 호출되었다. filter를 통과한 A라는 문자열에 대해서만 실행된 것이다. 그렇다면 filter를 맨 앞으로 당기면 어떻게 될까.

```java
Stream.of("a", "b", "c", "d")
  .filter(str -> {
    System.out.println("filter: " + str);
    return str.startsWith("a");
  })
  .map(str -> {
    System.out.println("map: " + str);
    return str.toUpperCase();
  })
  .forEach(str -> {
    System.out.println("forEach: " + str);
  });

/* 실행 결과
filter: a
map: a
forEach: A
filter: b
filter: c
filter: d
*/
```

위와 같이 수정된 코드는 filter가 5번, map과 forEach가 각각 1번씩 호출되었다. 동일한 입력과 결과에 대해 더 적은 연산으로 처리할 수 있게 된 것이다. 위의 예제는 간단한 예제이기 때문에 연산 횟수에 큰 차이가 없다. 하지만 처리해야 하는 데이터가 많아질수록 그만큼 큰 성능의 차이를 야기할 것이다. 때문에 Stream API를 사용할 때는 반드시 실행 순서를 고려하여 코드를 작성해야 한다.

<br>

## 6. 지연 처리

Stream의 연산은 최종 결과가 만들어 질 때 수행된다. 즉, 결과를 만드는 최종 작업이 수행되지 않는다면 중간 연산은 아무런 동작을 하지 않는다.

다음은 호출 횟수를 카운트하는 예제이다.

```java
private long counter;

private void wasCalled() {
  counter++;
}
```

```java
counter = 0;
Arrays.asList("a", "b", "c", "d", "e")
  .stream()
  .map(str -> {
    wasCalled();
    return str.toUpperCase();
  });
System.out.println(counter);

/* 실행 결과
0
*/
```

위의 예제는 Stream이 5개의 요소를 가지고 있기 때문에 `wasCalled()`가 5번 호출되어 5가 출력될 것으로 예상된다. 하지만 예상과 달리 0이 출력된다. 그 이유는 Stream의 최종 작업이 실행되지 않아 실제로 아무런 동작을 하지 않았기 때문이다.

최종 작업으로 collect 메서드를 넣은 후 실행 결과를 확인해 보자.

```java
counter = 0;
List<String> strings = Arrays.asList("a", "b", "c", "d", "e")
  .stream()
  .map(str -> {
    wasCalled();
    return str.toUpperCase();
  })
  .collect(Collectors.toList());
System.out.println(counter);

/* 실행 결과
5
*/
```

 이제 처음 예상한 대로 5가 출력되는 것을 확인할 수 있다.

<br>

## Related Posts

- [Stream API - 기본](https://heung27.github.io/posts/stream-api-%EA%B8%B0%EB%B3%B8/)
- [Optional](https://heung27.github.io/posts/optional/)
- [Functional Interface](https://heung27.github.io/posts/functional-interface/)

<br>

## Reference

- [https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)
- [https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)
- [https://m.blog.naver.com/tmondev/220945933678](https://m.blog.naver.com/tmondev/220945933678)
- [https://hbase.tistory.com/171](https://hbase.tistory.com/171)
- [https://12bme.tistory.com/461](https://12bme.tistory.com/461)
- [https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API](https://velog.io/@foeverna/Java-513-%EC%8A%A4%ED%8A%B8%EB%A6%BC-API-Stream-API)
- [https://mkyong.com/java8/java-8-flatmap-example/](https://mkyong.com/java8/java-8-flatmap-example/)
