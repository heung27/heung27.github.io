---
layout: post
title: Stream API - 기본
date: 2022-02-23 22:55 +0900
categories: [Language, Java]
tags: [Java 8, Stream API]
---



## Stream API란?

Stream API는 Java 8에서 추가되었으며, 데이터를 추상화하고 처리하는데 자주 사용되는 함수들을 정의해둔 API이다. 

여기서 데이터를 추상화하였다는 것은 데이터의 종류와 관계없이 같은 방식으로 데이터를 처리할 수 있다는 것을 의미한다. 

<br>

Java 8 이전에는 배열 또는 컬렉션 인스턴스를 다루기 위해 for 또는 while 문을 돌려 요소를 하나씩 꺼내 연산을 수행했다. 간단한 경우라면 문제가 되지 않지만, 로직이 복잡해질수록 코드의 양도 많아지고 여러 로직이 섞여 가독성도 떨어진다.

Stream API는 반복 문법을 메소드 내부에 숨긴 내부 반복을 사용해 이를 보완한다. 또한 배열 또는 컬렉션 인스턴스에 여러 개의 함수를 조합해 필터링하고 가공된 결과를 얻을 수 있다. 즉, 배열과 컬렉션을 함수형으로 처리할 수 있게 되는 것이다. 

<br>

정리하면 다음과 같은 **장점**을 가지고 있다.

- 유연하다.
  - 데이터를 추상화하고 함수를 조합하는 방식으로 데이터를 처리하기 때문에 변하는 요구사항에 쉽게 대응할 수 있다.
- 가독성이 높다.
  - 람다식 또는 메서드 참조를 이용해 간결하고 명확하게 표현할 수 있다.
- 병렬 처리가 쉽다.
  - 멀티 스레드 코드를 직접 구현하지 않고 내부 반복을 통해 간단하게 처리할 수 있다.

<br>

## 특징

### 1. 데이터 원본을 유지한다.

Stream API는 원본 데이터를 조회하여 원본 데이터가 아닌 별도의 요소들로 Stream을 생성한다. 그렇기 때문에 원본 데이터로부터 읽기만 할 뿐, 정렬이나 필터링 등의 작업은 별도의 Stream 요소들에서 처리 된다.

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> sortedList = list.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());

System.out.println(sortedList);
System.out.println(list);

/* 실행 결과
[5, 4, 3, 2, 1]
[1, 2, 3, 4, 5]
*/
```

Stream을 통해 원본 데이터[1, 2, 3, 4, 5]를 정렬하고 sortedList에 저장했다.

실행 결과를 보면 정렬이 잘 수행 되었고, 원본 데이터는 변경되지 않는 것을 확인할 수 있다.

<br>

### 2. 일회용이다.

Stream API는 일회용이기 때문에 한번 사용이 끝나면 재사용이 불가능하다. 

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

Stream<Integer> integerStream = list.stream();

int result = integerStream.reduce(0, Integer::max);

System.out.println(result);
System.out.println(integerStream.reduce(0, Integer::max));

/* 실행 결과
5
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
...
*/
```

list의 Stream인 integerStream를 통해 list의 최댓값을 추출했다. 그리고 다시 integerStream를 사용해 같은 연산을 수행했다.

실행 결과로 list의 최댓값 5가 출력된 후 IllegalStateException이 발생했다. integerStream이 이미 사용되고 닫힌 후 재사용하려고 했기 때문이다. 같은 Stream이 필요하다면 다시 생성해주어야 한다.

<br>

### 3. 내부 반복을 통해 작업을 처리한다.

Stream API에서는 반복 문법을 메소드 내부에 숨겨 내부 반복을 통해 작업을 처리한다. 때문에 보다 간결한 코드의 작성이 가능하다.

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

list.stream().forEach(System.out::println);

/* 실행 결과
1
2
3
4
5
*/
```

list의 요소들을 내부 반복을 통해 하나씩 출력했다.

위의 예제는 아주 단순하기 때문에 기존의 반복문과의 큰 차이를 느낄 수 없을지도 모른다. 하지만 복잡한 로직을 함수 조합과 내부 반복을 통해 간결하게 표현했을 때, 비로소 그 편리함에 희열을 느낄 수 있다. (나는 그렇다)

<br>

## 사용

간단한 예제를 통해 Stream API의 개념과 특징을 알아보았다. 이제 또다른 예제를 통해 사용법을 알아보자. 

Stream API의 연산은 크게 세 단계로 나눌 수 있다. Stream을 생성하는 단계, 데이터를 가공하는 단계, 마지막으로 결과 데이터를 만드는 단계가 있다. 

### 1. 생성

Stream 연산을 하기 위해서는 먼저 Stream 객체를 생성해주어야 한다. 배열, 컬렉션, 임의의 수, 파일 등 거의 모든 것을 가지고 스트림을 생성할 수 있다. 

#### 배열 Stream

Stream.of 또는 Arrays.stream 메서드를 사용해 생성한다. 

```java
String[] arr = new String[] {"a", "b", "c", "d"};

Stream<String> stringStream1 = Stream.of(arr);
Stream<String> stringStream2 = Arrays.stream(arr);
Stream<String> stringStream3 = Arrays.stream(arr, 1, 3); // ["b", "c"]
```



#### 컬렉션 Stream

Collection 인터페이스에는 default method로 stream()이 정의되어 있기 때문에, Collection 인터페이스를 구현한 객체들(List, Set 등)은 모두 이 메소드를 이용해 Stream을 생성할 수 있다.

```java
List<String> list = Arrays.asList("a", "b", "c", "d");

Stream<String> stringStream = list.stream();
```



#### 원시 Stream

위와 같이 객체를 위한 Stream 외에 원시 타입(int, long, double)을 다루기 위한 특수한 Stream도 있다. range 메서드를 통해 값의 범위를 지정해 생성한다.

```java
IntStream intStream = IntStream.range(1, 10);
LongStream longStream = LongStream.range(1, 10);
```

추가적으로 Java 8 의 `Random` 클래스를 사용하면 난수를 가지있는 원시 스트림을 만들 수 있다.

```java
LongStream longStream = new Random().longs(5); // 난수 5개 생성
```



#### 파일 Stream

자바 NIO 의 `Files` 클래스의 `lines` 메소드를 사용하면 해당 파일의 각 라인을 String 타입의 Stream으로 만들 수 있다.

```java
Stream<String> fileStream = Files.lines(Paths.get("test.txt"), StandardCharsets.UTF_8);
```



<br>

### 2. 가공

원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산의 단계이다. 어떤 객체의 Stream을 원하는 형태로 처리할 수 있으며, 중간 연산의 반환값은 Stream이기 때문에 필요한 만큼 중간 연산을 연결하여 사용할 수 있다. 

#### filter

Stream에서 조건에 맞지 않는 데이터를 걸려내 데이터를 정제하는 메서드이다. 인자로 boolean을 리턴하는 함수형 인터페이스 Predicate를 받는다.

```java
Stream<T> filter(Predicate<? super T> predicate);
```

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

integers.stream()
  .filter(num -> num >= 3)
  .forEach(System.out::println);

/* 실행 결과
3
4
5
*/
```

1부터 5까지의 숫자 리스트에서 3이상의 숫자를 필터링했다.



#### map

Stream의 각 요소들에 동일한 연산을 적용하는 메서드이다. 인자로 각 요소들에 적용할 연산인 함수형 인터페이스 Function을 받는다.

Stream에 들어가 있는 값이 특정 로직을 거친 후 새로운 스트림에 담기게 되는데, 이러한 작업을 맵핑(*mapping*)이라고 한다.

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

integers.stream()
  .map(num -> num * 2)
  .forEach(System.out::println);

/* 실행 결과
2
4
6
8
10
*/
```

1부터 5까지의 숫자 리스트에서 각 요소에 * 2를 적용했다.



#### sorted

Stream의 요소들을 정렬하는 메서드이다. 인자로 정렬 방법을 지정하는 함수형 인터페이스 Comparator를 받는다. 그리고 인자를 받지 않을 수도 있는데, 이때 정렬은 오름차순이 된다.

```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

````java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

integers.stream()
  .sorted(Comparator.reverseOrder())
  .forEach(System.out::println);

/* 실행 결과
5
4
3
2
1
*/
````

1부터 5까지 오름차순으로 정렬되어 있는 리스트를 준비했다. 여기에 `Comparator.reverseOrder()`를 사용하여 내림차순으로 다시 정렬했다.



#### distinct

Stream의 요소들에 중복된 데이터가 존재하는 경우, 중복을 제거하는 메서드이다. 

```java
Stream<T> distinct();
```

```java
List<Integer> integers = Arrays.asList(1, 1, 1, 5, 5);

integers.stream()
  .distinct()
  .forEach(System.out::println);

/* 실행 결과
1
5
*/
```

1이 세 개, 5가 두 개 가지고 있는 리스트를 준비하고, distinct를 수행해 중복 데이터를 제거했다.

여기서 주의해야 할 점이 하나 있다. distinct는 중복 데이터를 검사하기 위해 Object의 equals()를 사용한다. 때문에 우리가 생성한 클래스를 Stream으로 사용한다고 하면, equals()와 hashCode()를 오버라이드 해야만 distinct를 제대로 적용할 수 있다.



#### peek

Stream에 영향을 주지 않고 각 요소들에 특정 연산을 수행하는 메서드이다. 인자로 함수형 인터페이스 Consumer를 받는다. 

peek라는 단어가 '확인해본다'라는 뜻을 가지고 있는 것처럼, peek 함수는 Stream의 각 요소에 특정 연산을 수행할 뿐 결과에 영향을 주지 않는다. 예를 들어 어떤 Stream의 요소들을 중간에 출력해서 확인하고 싶을때 사용할 수 있다.

```java
Stream<T> peek(Consumer<? super T> action);
```

```java
IntStream intStream = IntStream.range(1, 5); // 1 ~ 4

int sum = intStream
  					.peek(System.out::println)
  					.sum();

System.out.println(sum);

/* 실행 결과
1
2
3
4
10
*/
```

1부터 4를 가지는 IntStream의 합계를 계산하는 과정이다. peek 함수를 적용하여 중간값을 출력해 확인하였고, 마지막으로 sum을 적용해 합계를 계산했다.



#### mapToObj

작업을 하다 보면 일반적인 Stream 객체를 원시 Stream으로 바꾸거나 그 반대의 작업이 필요한 경우가 있다. 이를 위해 일반적인 Stream 객체는 mapToInt(), mapToLong(), mapToDouble()이라는 특수한 Mapping 연산을 지원하고 있으며, 그 반대로 원시객체는 mapToObject를 통해 일반적인 Stream 객체로 바꿀 수 있다.

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

IntStream intStream1 = integers.stream()
                				.mapToInt(Integer::intValue); // List<Integer> => IntStream
```



<br>

### 3. 결과

가공 단계를 거쳐 만들어진 데이터를 최종적으로 원하는 결과 데이터로 만드는 단계이다. Stream의 요소들을 소모하면서 연산이 수행되기 때문에 한 번만 처리 가능하며, 연산이 끝난 뒤 Stream이 닫히게 된다.

#### calculate

max / min / sum / average / count

```java
IntStream intStream = IntStream.range(1, 5);

OptionalInt optionalInt = intStream.max();
```



``` java
int max = intStream.max().orElse(0); // 4
int min = intStream.min().orElse(0); // 1
double average = intStream.average().orElse(0); // 2.5
```



```java
int sum = intStream.sum();
long count = intStream.count();
```



#### collect

```java
List<User> userList = Arrays.asList(
    		new User("heung", 28),
    		new User("woneee", 24),
    		new User("sun", 28),
  			new User("bun", 31)
		);
```

##### 1. Collectors.toList

```java
List<String> nameList = userList.stream()
  	.map(User::getName)
  	.collect(Collectors.toList());
```

##### 2. Collectors.joining

```java
String listToString = userList.stream()
  	.map(User::getName)
  	.collect(Collectors.joining(", ", "<", ">"));
```

##### 3. Collectors.averagingInt / Collectors.summingInt / Collectors.summarizingInt

```java
Double average = userList.stream()
  	.collect(Collectors.averagingDouble(User::getAge));

Integer sum = userList.stream()
  	.collect(Collectors.summingInt(User::getAge));

IntSummaryStatistics statistics = userList.stream()
  	.collect(Collectors.summarizingInt(User::getAge));
```

##### 4. Collectors.groupingBy

```java
Map<Integer, List<User>> group = userList.stream()
  	.collect(Collectors.groupingBy(User::getAge));
```

##### 5. Collectors.partitioningBy

```java
Map<Boolean, List<User>> partition = userList.stream()
  	.collect(Collectors.partitioningBy(user -> user.getAge() > 25));
```

##### 6. Collectors.collectingAndThen

```java
Set<User> unmodifiableSet = userList.stream()
		.collect(Collectors.collectingAndThen(
      	Collectors.toSet(),
				Collections::unmodifiableSet
		));
```

##### 7. Collector.of

```java
Collector<User, ?, LinkedList<User>> toLinkedList = Collector.of(
      	LinkedList::new,
      	LinkedList::add,
      	(first, second) -> {
        		first.addAll(second);
      	  	return first;
    	  }
		);

LinkedList<User> userLinkedList = userList.stream()
		.collect(toLinkedList);
```



#### match

```java
List<Integer> integers = Arrays.asList(2, 4, 6, 8);

boolean anyMatch = integers.stream().anyMatch(num -> num == 2); // true
boolean allMatch = integers.stream().allMatch(num -> num % 2 == 0); // true
boolean noneMatch = integers.stream().noneMatch(num -> num % 2 == 1); // true
```



#### forEach

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

list.stream().forEach(System.out::println);
```



<br>continue 고급편

<br>

## Refference

- [https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)
- [https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)
- [https://hbase.tistory.com/171](https://hbase.tistory.com/171)
- [https://12bme.tistory.com/461](https://12bme.tistory.com/461)
