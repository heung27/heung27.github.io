---
layout: post
title: Item 14. Comparable을 구현할지 고려하라
date: 2022-05-28 01:05 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Override, Comparable, Comparator]
---



Effective Java의 열네 번째 아이템 "Comparable을 구현할지 고려하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

이번 포스팅은 정렬과 관련 있습니다. 개발을 하다 보면 정렬이 필요한 순간이 정말 많은데요. 그만큼 확실히 알아두면 유용한 내용이 내용일 될 것 같습니다. 본문에서 Comparable 인터페이스와 compareTo 메서드 작성 요령, 그리고 비슷한 기능을 하는 Comparator 인터페이스에 대해 알아보겠습니다.

<br>

## 1. Comparable

```java
package java.lang;
import java.util.*;

public interface Comparable<T> {
  
    public int compareTo(T o);
}

```

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻합니다. 그래서 Comparable을 구현한 객체들의 배열은 Arrays.sort 메서드를 통해 손쉽게 정렬할 수 있습니다. 또한 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 역시 쉽게 할 수 있습니다. 예를 들어 String이 Comparable을 구현한 덕분에 다음 코드는 문자열 배열을 알파벳순으로 출력합니다. 

 ```java
 String[] values = new String[]{"heung", "bae", "hoon", "na"};
 
 Set<String> s = new TreeSet<>();
 Collections.addAll(s, values);
 System.out.println(s);
 
 /* 실행 결과
 [bae, heung, hoon, na]
 */
 ```

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입은 Comparable을 이미 구현하고 있습니다. Comparable을 구현하면 Comparable을 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있습니다. 따라서 **알파벳, 숫자같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현해야 합니다.**

<br>

## 2. 일반 규약

compareTo 메서드의 일반 규약은 equals의 규약과 비슷합니다.

>이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>
>다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)을 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
>
>- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
>- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
>- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
>- (x.compareTo(y) == 0) == (x.equals(y))여야 한다. 
>  - 이번 권고가 필수는 아니지만 꼭 지키는 것이 좋다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.

모든 객체에 대해 전역 동치관계를 부여하는 equals 메서드와 달리, compareTo는 타입이 다른 객체를 신경쓰지 않아도 됩니다. 타입이 다른 객체가 주어지면 간단히 ClassCastException을 던져도 되며, 대부분 그렇게 합니다.

compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못합니다. 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 TreeSet과 TreeMap, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있습니다. 

세 규약은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻합니다. 그래서 주의사항도, 우회법도 같습니다. 

마지막 규약은 필수는 아니지만 꼭 지키길 권합니다. 간단히 말하면 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것입니다. 이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 됩니다. 

<br>

## 3. 작성 요령

compareTo 메서드 작성 요령은 equals와 비슷합니다. 몇 가지 차이점만 주의하면 됩니다.

**1) Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다.**

입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻입니다. 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않습니다. 

**2) compareTo 메서드는 각 필드가 동치인지를 비교하는 것이 아니라 그 순서를 비교한다.**

객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출합니다. Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용합니다.

```java

public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {

    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    
    @Override
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s); // 자바가 제공하는 기본 Comparator
    }
  
    ...
}
```

**3) compareTo 메서드에서 관계 연산자 \<와 \>를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니 추천하지 않는다.**

자바 7 이전에는 정수 기본 타입 필드를 비교할 때 관계 연산자인 \<와 \>를 사용하라고 권고했습니다. 하지만 이제는 박싱된 기본 타입 클래스에 새로 추가된 정적 메서드인 compare을 사용하는 것을 추천합니다.

**4) 클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다.**

가장 핵심적인 필드부터 비교해 나아가야 합니다. 비교 결과가 0이 아니라면, 즉 순서가 결정되면 거기서 끝내고 곧장 결과를 반환합니다. 가장 핵심이 되는 필드가 똑같다면, 똑같지 않은 필드를 찾을 때까지 그다음으로 중요한 필드를 비교해 나갑니다. 

다음은 PhoneNumber 클래스용 compreTo 메서드를 이 방식으로 구현한 모습니다.

```java
public class PhoneNumber implements Comparable<PhoneNumber> {

    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = (short) areaCode;
        this.prefix   = (short) prefix;
        this.lineNum  = (short) lineNum;
    }

    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
    
    ...
}

```

<br>

## 4. Comparator

자바 8에서는 Comparator 인터페이스가 일련의 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 Comparator 인스턴스를 생성할 수 있게 되었습니다. 그리고 이 Comparator는 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는 데 활용할 수 있습니다. 

다음은 PhoneNumber용 compareTo 메서드에 이 방식을 적용한 모습입니다.

```java
import java.util.Comparator;

import static java.util.Comparator.comparingInt;

public class PhoneNumber implements Comparable<PhoneNumber> {

    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = (short) areaCode;
        this.prefix   = (short) prefix;
        this.lineNum  = (short) lineNum;
    }

    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    @Override
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
  
   ...
}
```

Comparator는 수많은 보조 생성 메서드들을 가지고 있습니다. 자바의 숫자용 기본 타입을 모두 커버할 뿐 아니라 객체 참조용 Comparator 생성 메서드도 준비되어 있습니다. 이처럼 Comparator은 매우 간결하게 쓸 수 있다는 장점이 있지만, 약간의 성능 저하가 뒤따른다는 단점도 가지고 있습니다. 

<br>

## 5. 주의할 점

종종 '값의 차'를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo나 compare 메서드와 마주하게 됩니다. 예를 들면 다음과 같은 코드가 있습니다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    @Override
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

이 방식은 사용하면 안 됩니다. **정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있습니다.** 그 대신 다음의 두 방식 중 하나를 사용하는 것을 권장합니다.

**1) 정적 compare 메서드를 활용한 Comparator**

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    @Override
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

**2) 생성 메서드를 활용한 Comparator**

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

<br>

## 6. 핵심 정리

- 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자.
- compareTo 메서드에서 필드의 값을 비교할 때 \<와 \> 연산자는 쓰지 말아야 한다.
- 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 생성 메서드를 사용하자.

<br>

## 7. Related Posts

- 열거 타입 (Item 34)
- [equals 규약 (Item 10)](https://heung27.github.io/posts/item-10-equals%EB%8A%94-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EC%9D%84-%EC%A7%80%EC%BC%9C-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/)
