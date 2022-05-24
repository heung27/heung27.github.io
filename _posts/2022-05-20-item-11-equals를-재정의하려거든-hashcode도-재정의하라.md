---
layout: post
title: Item 11. equals를 재정의하려거든 hashCode도 재정의하라
date: 2022-05-20 17:00 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Override, hashCode]
---



Effective Java의 열한 번째 아이템 "equals를 재정의하려거든 hashCode도 재정의하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

equals를 재정의한 클래스가 hashCode를 재정의하지 않으면 일반 규약을 어기게 되어, 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것입니다. 때문에 반드시 equals를 재정의한 클래스는 hashCode도 재정의해야 합니다. 본문에서 hashCode의 일반 규약과 작성 요령, 주의할 점을 알아보겠습니다.

<br>

## 1. 일반 규약

다음은 Object 명세서에서 발췌한 규약입니다. 

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했다면, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다. 

<br>

**hashCode 재정의를 잘못했을 때 크게 문제가 되는 것은 두 번째 조항입니다.** 논리적으로 같은 객체는 같은 해시코드를 반환해야 하는 것인데요. 이전 포스팅에서 다루었듯이 equals는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수 있습니다. 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여, 규약과 달리 서로 다른 값을 반환합니다. 

다음은 equals를 재정의한 PhoneNumber 클래스입니다.

```java
public class PhoneNumber {

    private final short prefix, middle, suffix;

    public PhoneNumber(int prefix, int middle, int suffix) {
        this.prefix = (short) prefix;
        this.middle   = (short) middle;
        this.suffix  = (short) suffix;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.prefix == prefix && pn.middle == middle && pn.suffix == suffix;
    }
}
```

PhoneNumber 클래스를 사용하는 클라이언트 코드입니다.

```java
Map<PhoneNumber, String> map = new HashMap<>();

PhoneNumber p1 = new PhoneNumber(123, 456, 7890);
map.put(p1, "heung");

PhoneNumber p2 = new PhoneNumber(123, 456, 7890);

System.out.println(p1.equals(p2));

System.out.println(map.get(p1));
System.out.println(map.get(p2));

/* 실행 결과
true
heung
null
*/
```

p1이라는 PhoneNumber 객체 key로 하여 HashMap에 heung를 넣었습니다. 동일한 번호를 가지는 새로운 인스턴스 p2를 만들고 이전의 p1과 equals 메서드를 사용해 비교했습니다. PhoneNumber 클래스에서 equals를 재정의했기 때문에 true가 반환됩니다. 하지만 HashMap에서 p2에 해당하는 값을 꺼냈을 때는 null이 반환되는 것을 확인할 수 있습니다. 이유는 Object의 기본 hashCode 메서드가 이 둘이 전혀 다르다고 판단하여 서로 다른 값을 반환하기 때문입니다. 

이 문제는 PhoneNumber에 적절한 hashCode 메서드만 작성해 주면 해결됩니다. 올바른 hashCode 메서드는 어떤 모습이어야 할까요? 올바르지 않게 작성하려면 아주 간단합니다. 예를 들어 다음 코드는 적법하게 구현했지만, 절대 사용해서는 안 됩니다.

```java
@Override
public int hashCode() {
  return 42;
}
```

이 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법합니다. 하지만 모든 객체에서 똑같은 값을 반환하기 때문에 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작하게 됩니다. 그 결과 시간 복잡도가 O(1)인 해시테이블이 O(n)으로 느려져서 객체가 많아지면 도저히 쓸 수 없게 됩니다. 올바른 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환합니다. 이것이 바로 hashCode의 세 번째 규약이 요구하는 속성입니다. 

<br>

## 2. 작성 요령

이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 합니다. 이상을 완벽히 실현하기는 어렵지만 비슷하게 만들기는 그다지 어렵지 않습니다. 다음은 올바른 hashCode를 작성하는 간단한 요령입니다.

1. **int 변수 result를 선언한 후 값 c로 초기화한다.** 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2-1 방식으로 계산한 해시코드다. 여기서 핵심 필드란 equals 비교에 사용되는 필드를 말한다.
2. **해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.**
   1. 해당 필드의 해시코드 c를 계산한다.
      1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
      2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2-2 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다. 
   2. 계산한 해시코드 c로 result를 갱신한다. (`result = 31 * result + c;`)
3. **result를 반환한다.**

파생 필드는 해시 코드 계산에서 제외해도 됩니다. 즉, 다른 필드로부터 계산해 낼 수 있는 필드는 모두 무시해도 됩니다. 또한 equals 비교에 사용되지 않은 필드는 '반드시' 제외해야 합니다. 그렇지 않으면 hashCode 규약 두 번째를 어기게 될 위험이 있습니다.

단계 2-2의 곱셈 `31 * result`는 필드를 곱하는 순서에 따라 result 값이 달라지게 합니다. 그 결과 클래스에 비슷한 필드가 여러 개 일 때 해시 효과를 크게 높여줍니다. 곱할 숫자를 31로 정한 이유는 31이 홀수이면서 소수여서 최적화에 효과적이기 때문입니다. 

<br>

## 3. 예제

위의 요령을 적용해 PhoneNumber 클래스를 개선했습니다.

```java
public class PhoneNumber {

    private final short prefix, middle, suffix;

    public PhoneNumber(int prefix, int middle, int suffix) {
        this.prefix = (short) prefix;
        this.middle   = (short) middle;
        this.suffix  = (short) suffix;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.prefix == prefix && pn.middle == middle && pn.suffix == suffix;
    }

    @Override
    public int hashCode() {
        int result = Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(middle);
        result = 31 * result + Short.hashCode(suffix);
        return result;
    }
}
```

PhoneNumber 인스턴스의 핵심 필드 3개만을 사용해 해시코드를 계산합니다. 그 과정에 비결정적 요소는 전혀 없으므로 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가질 것이 확실합니다. 

```java
Map<PhoneNumber, String> map = new HashMap<>();

PhoneNumber p1 = new PhoneNumber(123, 456, 7890);
map.put(p1, "heung");

PhoneNumber p2 = new PhoneNumber(123, 456, 7890);

System.out.println(map.get(p2)); // heung
```

위에서 봤던 테스트 코드를 다시 실행시켜 보겠습니다. 이전에는 null이 출력됐었는데요. 이제는 heung가 출력되는 것을 확인할 수 있습니다.

소개한 해시 함수 작성 요령은 자바 라이브러리가 사용한 방식과 견주어도 손색이 없으며 대부분의 쓰임에도 문제가 없습니다. 단, 해시 충돌이 더욱 적은 방법을 꼭 써야 한다면 구아바의 com.google.common.hash.Hashing을 참고하는 것이 좋습니다.

```java
@Override
public int hashCode() {
  return Hashing.goodFastHash(32)
    .hashObject(this, PhoneNumberFunnel.INSTANCE)
    .hashCode();
}

private static class PhoneNumberFunnel implements Funnel<PhoneNumber> {

  private static final PhoneNumberFunnel INSTANCE = new PhoneNumberFunnel();

  @Override
  public void funnel(PhoneNumber from, PrimitiveSink into) {
    into.putShort(from.prefix).putShort(from.middle).putShort(from.suffix);
  }
}
```

<br>

## 4. 주의할 점

hashCode를 재정의할 때는 몇 가지 주의해야 할 사항이 있습니다.

#### Object 클래스가 제공하는 hash 메서드는 성능이 민감하지 않은 상황에서만 사용해야 합니다.

Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해 주는 정적 메서드인 hash를 제공합니다. 이 메서드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한 줄로 작성할 수 있습니다. 하지만 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문에 속도가 느립니다. 

#### 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 합니다.

이 타입의 객체가 주로 해시의 key로 사용될 것 같다면 인스턴스가 만들어질 때 해시 코드를 계산해둬야 합니다. 해시의 key로 사용되지 않는 경우라면 hashCode가 처음 호출될 때 계산하는 지연 초기화 전략을 고려해야 합니다. 필드를 지연 초기화하려면 그 클래스를 thread-safe하게 만들어야 합니다. PhoneNumber 클래스는 굳이 이렇게까지 할 이유는 없지만, 예시를 위해 만들었습니다.

```java
private volatile int hashCode;

@Override 
public int hashCode() {
  if (this.hashCode != 0) {
    return hashCode;
  }

  synchronized (this) {
    int result = hashCode;
    if (result == 0) {
      result = Short.hashCode(prefix);
      result = 31 * result + Short.hashCode(middle);
      result = 31 * result + Short.hashCode(suffix);
      this.hashCode = result;
    }
    return result;
  }
}
```

#### 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 됩니다.

속도는 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있습니다. 특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을지도 모릅니다. 하필 이런 필드를 생략한다면 해당 영역의 수많은 인스턴스가 단 몇 개의 해시코드로 집중되어 해시테이블의 속도가 선형으로 느려질 것입니다.

#### hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말아야 합니다.

자세한 생성 규칙을 공표하면 클라이언트는 이 값에 의지하게 됩니다. 이는 향후 해시 기능을 개선할 여지를 없애버리는 일입니다.  자세한 규칙을 공표하지 않는다면, 해시 기능에서 결함을 발견했거나 더 나은 해시 방식을 알아낸 경우 다음 릴리스에서 수정할 수 있습니다.

<br>

## 5. 핵심 정리

- equals를 재정의 할 때는 hashCode도 반드시 재정의해야 한다.
- 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 한다.
- 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
- Lombok 라이브러리 또는 AutoValue 프레임워크를 사용하면 euqals와 hashCode를 자동으로 만들어준다.

<br>

## 6. Related Posts

- [equals (Item 10)](https://heung27.github.io/posts/item-10-equals%EB%8A%94-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EC%9D%84-%EC%A7%80%EC%BC%9C-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/)
- 지연 초기화 (Item 83)
