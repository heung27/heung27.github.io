---
layout: post
title: "[Effective Java] Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라"
date: 2022-04-09 23:22 +0900
categories: [Book, Java]
tags: [Effective Java, static method]
---



시작하기 전에 정적 팩터리 메서드(static factory method)의 개념을 잡고가자. 다음은 Boolean 클래스에서 제공하는 정적 팩터리 메서드이다.

```jsx
// 정적 팩터리 메서드 예시
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

해당 메서드는 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해 준다. 정적 팩터리 메서드란 단순히 클래스의 인스턴스를 반환하는 static 메서드 것이다.

이처럼 클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재하는데, 본문에서 간단한 예제를 통해 자세히 살펴본다.

<br>

### 장점 1. 이름을 가질 수 있다.

생성자에 넘기는 매개변수와 생성자 자체 만으로는 반환될 객체의 특성을 제대로 설명하지 못 한다. 반면 정적 팩터리 메서드는 이름만 잘 지으면 **반환될 객체의 특성을 쉽게 묘사할 수 있다.**

다음 예제는 BigInteger 클래스가 제공하는 probablePrime라는 정적 팩터리 메서드이다.

```java
package java.math;

public class BigInteger extends Number implements Comparable<BigInteger> {

			// probablePrime라는 이름을 통해 '값이 소수인 BigInteger를 반환한다'는 의미를 명확히 한다.
	    public static BigInteger probablePrime(int bitLength, Random rnd) {
	      if (bitLength < 2)
	          throw new ArithmeticException("bitLength < 2");
	
	      return (bitLength < SMALL_PRIME_THRESHOLD ?
	              smallPrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd) :
	              largePrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd));
		}
}
```

<br>

### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

이 덕분에 불변 클래스는 **인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.** 비슷한 기법으로 플라이웨이트 패턴이 있다.

반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저하게 통제할 수 있다. 이런 클래스를 통제 클래스라 한다.

그렇다면 인스턴스를 통제하는 이유는 무엇일까?

인스턴스를 통제하면 클래스를 싱글턴 또는 인스턴스화 불가로 만들 수 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐 임을 보장할 수 있다.

```java
public class Developer {

    boolean canBackend;

    boolean canFrontend;

    private Developer() {}

		// 인스턴스를 미리 만들어 놓고, 이를 재활용 한다.
    private static final Developer developer = new Developer();

    public static Developer getBackEnd() {
        developer.canBackend = true;
        return developer;
    }

    public static Developer getFrontEnd() {
        developer.canFrontend = true;
        return developer;
    }

    public static Developer getFullStack() {
        developer.canBackend = true;
        developer.canFrontend = true;
        return developer;
    }

}
```

<br>

> **플라이웨이트 패턴(Flyweight Pattern)**
>
> 어떤 클래스의 인스턴스 한 개만 가지고 여러 개의 가상 인스턴스를 제공하고 싶을 때 사용하는 패턴이다. 즉, 인스턴스를 가능한 대로 공유시켜 쓸데없이 new 연산자를 통한 메모리 낭비를 줄이는 방식이다.

<br>

### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

이 능력은 **반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성'을 선물한다.** API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이다.

자바 컬렉션 프레임워크는 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙이 총 45개의 유틸리티 구현체를 제공하는데, 이 구현체 대부분을 단 하나의 인스턴스 불가 동반 클래스인 java.util.Collections에서 정적 팩터리 메서드를 통해 얻도록 했다.

컬렉션 프레임워크는 이 45개의 클래스를 공개하지 않기 때문에 API 외견을 훨씬 작게 만들 수 있었다. API가 작아진 것은 물론 개념적인 무게, 즉 프로그래머가 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도도 낮췄다. 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지알아보지 않아도 된다. 나아가 정적 팩터리 메서드를 사용하는 클라이언트는 객체를 인터페이스만으로 다루게 된다.

<br>

> **인스턴스화 불가 동반 클래스(companion class)**
>
>  java 8 이전에는 인터페이스에서 정적 메서드를 정의할 수 없기 때문에 인스턴스화 불가 동반 클래스를 두어 관련된 정적 메서드를 제공했다. 대표적인 예로 위에서 언급한 Collection 인터페이스와 Collections 동반 클래스가 있다. java 8 이후에는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸기 때문에, 동반 클래스에 두었던 정적 메서드를 인터페이스 자체에 둘 수 있게 되었다. 따라서 인스턴스화 불가 동반 클래스를 둘 이유가 없어졌다.

<br>

### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다.

다음은 EnumSet 클래스가 제공하는 noneOf라는 정적 팩터리 메서드이다.

```java
package java.util;

public abstract class EnumSet<E extends Enum<E>> {
	...

	public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

				// 원소가 64개 이하면 원소들을 long 변수 하나로 관리하는 RegularEnumSet 인스턴스를 반환 한다.
				// 65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다.
        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }

	...
}
package java.util;

// 위의 noneOf 메서드는 반환형이 EnumSet 인데, RegularEnumSet와 JumboEnumSet를 반환했다.
// 이렇게 할 수 있는 이유는 RegularEnumSet와 JumboEnumSet이 EnumSet의 하위 타입이기 때문이다. (장점 3)
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
	...
}

class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {
	...
}
```

<br>

### 장점 5. 정적 펙터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

다음 예제는 프로젝트 내부에 HelloService의 동작을 구현하는 구현체가 존재하지 않는 상황에서 HelloService 사용하는 예제이다.

```java
public interface HelloService {

    String hello();
}
public class HelloServiceFactory {

    /*
				ServiceLoader를 통해 외부 라이브러리(jar)에 포함된 구현체를 읽어들여 사용한다.
        즉, 어떤 구현체가 올 지 모르지만 해당 구현체가 따르는 인터페이스 기반으로 사용하는 것이다.
        이는 특정 구현체에 의존하지 않기 때문에(import도 하지 않음) 유연한 코드이다.

     */
    public static Optional<HelloService> getService() {
        ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);
        return loader.findFirst();
    }
}
public class HelloMain {

    public static void main(String[] args) {
        HelloServiceFactory.getService()
                .ifPresent(HelloService::hello);
    }
}
```

**이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다.** 대표적인 서비스 제공자 프레임워크로는 JDBC가 있다. 서비스 제공자 프레임워크에서의 제공자는 서비스의 구현체다. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.

서비스 제공자 프레임워크는 3개의 핵심 컴포넌트로 이뤄진다.

- **서비스 인터페이스** : 구현체의 동작을 정의
- **제공자 등록 API** : 제공자가 구현체를 등록할 때 사용
- **서비스 접근 API** : 클라이언트가 서비스의 인스턴스를 얻을 때 사용

클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조건을 명시할 수 있다. 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다. 이 서비스 접근 API가 바로 서비스 제공자 프레임워크의 근간이라고 한 ‘유연한 정적 팩터리'의 실체다.

3개의 핵심 컴포넌트와 더불어 종종 서비스 **제공자 인터페이스**라는 네 번째 컴포넌트가 쓰이기도 한다. 이 컴포넌트는 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다. 서비스 제공자 인터페이스가 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 한다.

서비스 제공자 프레임워크 패턴에는 여러 변형이 있다. 예컨대 서비스 접근 API는 공급자가 제공하는 것보다 더 풍부한 서비스 인터페이스를 클라이언트에 반환할 수 있다. 브리지 패턴이라 알려진 것이다. 의존 객체 주입 프레임워크도 강력한 서비스 제공자라고 생각할 수 있다.

<br>

> **브리지 패턴(Bridge pattern)** 
>
> 구현부에서 추상층을 분리하여 각자 독립적으로 변형 및 확장이 가능하도록 만드는 패턴이다. 즉, 기능과 구현에 대해 두 개의 별도의 클래스로 구현한다.

<br>

### 단점 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

어찌보면 이 제약은 상속보다 컴포지션을 사용 하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.

```java
// 컴파일 에러
// 하위 클래스(JuniorDeveloper)에서 상위 클래스(Developer)의 생성자를 사용할 수 없기 때문에 에러가 발생한다.
public class JuniorDeveloper extends Developer {
	...
}
```

<br>

### 단점 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.

```java
// document 생성
mvn javadoc:javadoc
```

java document에서 생성자의 경우 해당 내용을 정의하는 영역이 있어 한눈에 확인할 수 있지만, 정적 팩터리 메서드의 경우 많은 메서드들 사이에 정의되어 있어 사용자가 직접 찾아봐야하는 번거로움이 존재한다. 이런 불편을 해소하기 위해서는 아래와 같이 document에 관련 설명을 추가해 주도록 하자.

```java
/**
 * 이 클래스는 생성자를 제공하지 않으며, 인스턴스를 얻기 위해 정적 팩터리 메서드를 사용한다.
 * @see #getBackEnd()
 * @see #getFrontEnd()
 * @see #getFullStack()
 */
public class Developer {
	...
}
```
