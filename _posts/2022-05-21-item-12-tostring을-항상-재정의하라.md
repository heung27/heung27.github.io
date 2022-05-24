---
layout: post
title: Item 12. toString을 항상 재정의하라
date: 2022-05-21 11:52 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Override, toString]
---



Effective Java의 열두 번째 아이템 "toString을 항상 재정의하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없습니다. 이 메서드는 `PhoneNumber@fdsaa` 처럼 단순히 `클래스_이름@16진수_해시코드`를 반환합니다. 때문에 재정의가 필요한 것인데요. 본문에서 toString의 일반 규약과 작성 요령을 알아보겠습니다.

<br>

## 1. 일반 규약

- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
- 모든 하위 클래스에서 재정의하라.

<br>

**toString을 잘 구현한 클래스는 사용하기에 훨씬 편하고, 그 클래스를 사용한 시스템은 디버깅하기 쉽습니다.** 

toString 메서드는 우리가 직접 호출하지 않아도 어딘가에서 쓰일 수 있습니다. 예를 들어 작성한 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때 자동으로 호출할 수 있습니다. 이때 toString을 제대로 재정의 하지 않았다면 쓸모없는 메시지만 로그에 남을 것입니다. 이외에도 객체를 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때 혹은 디버거가 객체를 출력할 때 자동으로 호출됩니다.

<br>

## 2. 예제

PhoneNumber용 toString을 재정의합니다.

```java
public class PhoneNumber {

    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = (short) areaCode;
        this.prefix   = (short) prefix;
        this.lineNum  = (short) lineNum;
  
    ...

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
}

```

이제 print만으로도 객체의 상태를 확인할 수 있습니다.

```java
PhoneNumber phoneNumber = new PhoneNumber(123, 456, 7890);

System.out.println(phoneNumber); // 123-456-7890
```

객체에 어떤 문제가 생겼을 경우 위처럼 간단하게 문제를 진단할 수 있습니다. toString을 재정의하지 않았다면 그다지 쓸모없는 메시지가 출력될 것입니다.

<br>

## 3. 작성 요령

#### toString은 그 객체가 가진 주요 정보 모두를 반환하는 것이 좋다.

PhoneNumber 클래스처럼 전화번호를 나타내는 주요 정보인 prefix, middle, suffix를 반환하는 것입니다. 그런데 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 무리가 있습니다. 이런 상황이라면 `맨해튼 거주자 전화번호부(총 1487536)`나 `Thread[main,5,main]`같은 요약 정보를 담아야 합니다. 이상적으로는 스스로를 완벽히 설명하는 문자열이어야 합니다. 

#### toString을 구현할 때는 반환값의 포맷을 문서화할지 정해야 합니다. 

전화번호나 행렬 같은 값 클래스라면 문서화하기를 권합니다. 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 됩니다. 따라서 그 값 그대로 입출력에 사용하거나 CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수도 있습니다. 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해 주면 좋습니다.

문서화에는 단점도 있습니다. 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다는 것입니다. 클래스를 사용하는 프로그래머들이 그 포맷에 맞춰 파싱하고, 새로운 객체를 만들고, 영속 데이터로 저장하는 코드를 작성할 것입니다. 만약 향후 릴리스에서 포맷을 바꾼다면 이를 사용하던 코드들과 데이터들은 엉망이 될 것입니다.

#### 포맷을 명시하든 아니든 작성자의 의도는 명확히 밝혀야 한다.

주석을 활용하여 toString에 대한 의도를 명시해 줘야 한다는 뜻입니다. 포맷을 명시하기로 했다면 아주 정확하게 작성해야 합니다. 다시 PhoneNumber의 toString을 보겠습니다.

```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override
public String toString() {
  return String.format("%03d-%03d-%04d",
                       areaCode, prefix, lineNum);
}
```

다른 예로, 포맷을 명시하지 않기로 했다면 다음처럼 작성할 수 있습니다.

```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
*
* "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```

이러한 설명을 읽고도 이 포맷에 맞춰 코딩하거나 특정 값을 빼내어 영구 저장한 프로그래머는 나중에 포맷이 바뀌어 피해를 입어도 자기 자신을 탓할 수밖에 없을 것입니다.

#### 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

예를 들어 PhoneNumber 클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 합니다. 그렇지 않으려면 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱할 수밖에 없습니다. 성능이 나빠질 뿐만 아니라 향후 포맷이 바뀌었을 때 시스템이 망가는 결과를 초래할 수 있습니다. 

<br>

## 4. 핵심 정리

- 모든 구체 클래스에서 Object의 toString을 재정의하자.
- toString을 재정의한 클래스는 시스템을 디버깅하기 쉽게 해준다.
- toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.

<br>

## 5. Related Posts

- [정적 유틸리티 클래스 (Item 4)](https://heung27.github.io/posts/item-4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)
- 열거 타입 (Item 34)

