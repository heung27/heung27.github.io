---
layout: post
title: "Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라"
date: 2022-04-12 23:45 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, singleton, design pattern]
---



Effective Java의 세 번째 아이템 "private 생성자나 열거 타입으로 싱글턴임을 보증하라"를 읽고 정리한 내용을 포스팅합니다.



## 싱글턴(singleton)

싱글턴이란 어떤 클래스가 최초 한 번만 메모리를 할당하고, 그 메모리의 인스턴스를 만들어 사용하는 디자인 패턴입니다. 즉, 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다. 

싱글턴의 전형적인 예로는 함수와 같은 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있습니다. 

<br>

### 사용하는 이유

인스턴스가 한 개만 생성되도록 하면 어떤 이점이 있을까요?

가장 먼저 떠올릴 수 있는 이점은 아무래도 **메모리**일 것입니다. 최초 한 번의 new 연산자를 통해서 고정된 메모리 영역을 사용하기 때문에 추후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있습니다. 또한 객체의 두 번째 접근부터는 객체 로딩이 줄어 더 빠른 **속도**을 기대할 수 있습니다. 또 다른 이점은 **데이터 공유**가 쉽다는 것입니다. 인스턴스가 전역으로 사용되기 때문에 다른 클래스의 인스턴스들이 접근하여 사용할 수 있습니다.

이외에도 인스턴스가 절대적으로 한 개만 존재하는 것을 보증하고 싶은 경우 사용합니다.

<br>

### 문제점

싱글턴에도 문제점은 있습니다.

첫 번째, **멀티스레드 환경에서 동시성 문제가 발생할 수 있습니다.** 하나의 인스턴스를 여러 스레드에서 공유하고 있기 때문에, 어떤 스레드에서 인스턴스의 상태를 변경한다면 이는 다른 스레드에도 영향이 미치게 됩니다. 결국 일관된 값을 보장할 수 없게 되는 것입니다.

두 번째, **테스트를 수행하기 어렵습니다.** 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없습니다. 때문에 격리된 환경에서 독립적인 테스트를 수행하기 어렵습니다.

세 번째, **의존 관계상 클라이언트가 구체 클래스에 의존하게 됩니다.** new 키워드를 직접 사용하여 클래스 안에서 생성하고 있으므로 객체 지향 설계 SOLID 원칙 중 DIP(의존 역전 원칙)를 위반하게 되고, OCP(개방 폐쇄 원칙) 또한 위반하게 될 가능성이 높습니다.

<br>

### 싱글턴 구현 방식

위에서 싱글턴을 사용하는 이유와 문제점을 알아봤습니다. 이제 인스턴스가 오직 한 개만 존재하도록 하는 이 싱글턴을 구현하는 세 가지 방식을 알아보겠습니다.

#### 1) public 필드

**생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public 필드를 제공하는 방식**입니다.

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public void leaveTheBuilding() { ... }
}
```

private 생성자는 public static final 필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출됩니다. public이나 protected 생성자가 없기 때문에 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됩니다. 

```java
Elvis elvis1 = Elvis.INSTANCE;
Elvis elvis2 = Elvis.INSTANCE;

System.out.println(elvis1 == elvis2);
/* 실행 결과
true
*/
```

public 필드 방식의 장점은 해당 클래스가 **싱글턴임이 API에 명백히 드러난다**는 것입니다. public static 필드가 final이니 절대로 다른 객체를 참조할 수 없습니다. 두 번째 장점은 **간결함**입니다. 구현이 쉬워 코드가 깔끔하고 가독성도 좋습니다.

<br>

그런데 한 가지 예외 상황이 있습니다. 권한이 있는 클라이언트는 **리플렉션 API인 `AccessibleObject.setAccessible`을 사용하면 private 생성자를 호출할 수 있습니다.** 

```java
try {
  Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
  defaultConstructor.setAccessible(true); // private 생성자 호출 허용
  
  Elvis elvis1 = defaultConstructor.newInstance();
  Elvis elvis2 = defaultConstructor.newInstance();
  
  System.out.println(elvis1 == elvis2);
} catch (NoSuchMethodException | InvocationTargetException | InstantiationException | IllegalAccessException e) {
  e.printStackTrace();
}
/* 실행 결과
false
*/
```

먼저 `Elvis.class.getDeclaredConstructor` 메서드를 통해 Constructor 객체를 얻습니다. 그리고 `setAccessible(true)`를 사용해 private 생성자 호출을 허용하면 `newInstance()`를 통해 새로운 인스턴스를 생성할 수 있습니다. 이제 elvis1과 elvis2 인스턴스를 비교해 보면 false가 출력되는 것을 확인할 수 있습니다.

이러한 공격을 방어하기 위해서는 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지도록 해야 합니다.

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();
  
    private static boolean created;

    private Elvis() {
        // 인스턴스가 이미 만들어졌을 경우 예외 throw
        if (created) {
            throw new UnsupportedOperationException("can't be created by constructor.");
        }
        created = true;
    }

    public void leaveTheBuilding() { ... }
}
```

이제 다시 테스트를 해 보겠습니다.

```java
// 수정된 Elvis 테스트
try {
  Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
  defaultConstructor.setAccessible(true);
  
  Elvis elvis1 = defaultConstructor.newInstance(); // 에러
  Elvis elvis2 = defaultConstructor.newInstance(); // 에러
  
  System.out.println(elvis1 == elvis2);
} catch (NoSuchMethodException | InvocationTargetException | InstantiationException | IllegalAccessException e) {
  e.printStackTrace();
}
/* 실행 결과
java.lang.reflect.InvocationTargetException
Caused by: java.lang.UnsupportedOperationException: can't be created by constructor.
...
*/
```

Elvis 클래스의 `public static final Elvis INSTANCE = new Elvis();`에서 이미 인스턴스를 만들었기 때문에 보이는 것과 같이 UnsupportedOperationException가 발생했습니다. 이제는 리플렉션을 통해 새로운 인스턴스를 생성할 수 없어졌습니다.

<br>

한 가지 더 문제점이 있습니다. **역직렬화를 할때 새로운 인스턴스가 생길 수 있다**는 것입니다. 먼저 Elvis 클래스의 직렬화를 위해 Serializable 인터페이스를 구현합니다.

```java
public class Elvis implements Serializable {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public void leaveTheBuilding() { ... }
}
```

다음은 Elvis 클래스를 직렬화한 후 다시 역직렬화해 기존의 Elvis 인스턴스와 동일한지 비교하는 예제입니다.

```java
try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
  out.writeObject(Elvis.INSTANCE);
} catch (IOException e) {
  e.printStackTrace();
}

try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
  Elvis elvis = (Elvis) in.readObject();
  System.out.println(elvis == Elvis.INSTANCE);
} catch (IOException | ClassNotFoundException e) {
  e.printStackTrace();
}
/* 실행 결과
false
*/
```

실행 결과에서 알 수 있듯이 동일하지 않은 인스턴스가 생성되었습니다. 이를 해결하기 위해서는 Elvis 클래스에 readResolve 메서드를 추가해 주어야 합니다.

```java
public class Elvis implements Serializable {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public void leaveTheBuilding() { ... }
    
    private Object readResolve() {
        return INSTANCE;
    }
}
```

이제 다시 위의 테스트 코드를 실행시켜 보면 true를 출력하는 것을 확인할 수 있습니다. 역직렬화를 할 때 내부적으로 readResolve라는 메서드를 호출하도록 설계되어 있기 때문입니다. 

<br>

#### 2) 정적 팩터리

**생성자와 필드는 private으로 감춰두고, 유일할 인스턴스를 접근할 수 있는 수단으로 정적 팩터리 메서드를 제공하는 방식**입니다.

```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() { ... }
}
```

다음은 정적 팩터리 메서드를 사용하여 두 객체를 만들고 인스턴스를 비교하는 예제입니다.

```java
Elvis elvis1 = Elvis.getInstance();
Elvis elvis2 = Elvis.getInstance();

System.out.println(elvis1 == elvis2);
/* 실행 결과
true
*/
```

Elvis.getInstance 메서드는 항상 같은 객체의 참조를 반환하기 때문에 제2의 인스턴스는 결코 만들어지지 않습니다. 

<br>

정적 팩터리 방식의 첫 번째 장점은 **API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다**는 점입니다.

```java
Elvis elvis1 = Elvis.getInstance();
Elvis elvis2 = Elvis.getInstance();

System.out.println(elvis1);
System.out.println(elvis2);
/* 실행 결과
example.item3.Elvis@6aaa5eb0
example.item3.Elvis@6aaa5eb0
*/
```

위와 같은 클라이언트 코드가 있습니다. Elvis가 싱글턴이니 elvis1과 elvis2는 같은 인스턴스를 참조하고 있을 것입니다. 하지만 어떤 이유로 인해 싱글턴이 아니게, 즉 서로 다른 인스턴스를 만들고 싶다면 어떻게 해야 할까요? 

```java
public static Elvis getInstance() {
    return new Elvis();
}
```

정적 팩터리 방식에서는 클라이언트 코드를 수정하지 않고 정적 팩터리 메서드를 수정하면 간단하게 변경할 수 있습니다.

```java
Elvis elvis1 = Elvis.getInstance();
Elvis elvis2 = Elvis.getInstance();

System.out.println(elvis1);
System.out.println(elvis2);
/* 실행 결과
example.item3.Elvis@6aaa5eb0
example.item3.Elvis@3498ed
*/
```

<br>

두 번째 장점은 **정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다**는 점입니다. 다음은 Elvis 클래스를 제네릭 싱글턴 팩터리 패턴으로 만든 예제입니다.

 ```java
 public class Elvis<T> {
 
     private static final Elvis<Object> INSTANCE = new Elvis<>();
 
     private Elvis() {}
 
     @SuppressWarnings("unchecked")
     public static <T> Elvis<T> getInstance() {
         return (Elvis<T>) INSTANCE;
     }
 
     public void leaveTheBuilding() { ... }
 }
 ```

제네릭 싱글턴 팩터리는 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리 입니다. 제네릭 싱글턴 팩터리를 사용하면 동일한 인스턴스를 참조하지만 서로 다른 타입의 객체를 만들 수 있습니다.

```java
Elvis<String> elvis1 = Elvis.getInstance();
Elvis<Integer> elvis2 = Elvis.getInstance();

System.out.println(elvis1);
System.out.println(elvis2);
/* 실행 결과
example.item3.Elvis@6aaa5eb0
example.item3.Elvis@6aaa5eb0
*
```

실행 결과를 보면 elvis1과 elvis2는 같은 인스턴스를 참조하고 있습니다. 하지만 둘은 서로 다른 타입을 가지고 있습니다. 

<br>

세 번째 장점은 **정적 팩터리의 메서드 참조를 함수형 인터페이스 Supplier로 사용할 수 있다**는 점입니다. 

다음은 Java에서 제공하는 함수형 인터페이스 Supplier의 선언 부분입니다.

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

Elvis의 getInstance 메서드는 인자는 받지 않고 인스턴스를 리턴합니다. 그 구조가 Supplier의 get 메서드와 동일합니다. 따라서 Elvis getInstance 메서드의 참조를 Supplier로 사용할 수 있습니다.

```java
Supplier<Elvis> elvisSupplier = Elvis::getInstance; // 물론 람다식도 가능합니다. () -> Elvis.getInstance();

Elvis elvis1 = elvisSupplier.get();
Elvis elvis2 = elvisSupplier.get();

System.out.println(elvis1);
System.out.println(elvis2);
/* 실행 결과
example.item3.Elvis@1a407d53
example.item3.Elvis@1a407d53
*/
```

Supplier의 get 메서드와 Elvis의 getInstance가 매핑됩니다. 실행 결과를 보면 get 메서드가 동일한 Elvis 인스턴스를 리턴하는 것을 알 수 있습니다.

<br>

하지만 **정적 팩터리 방식에도 public 필드 방식과 같이 리플렉션과 직렬화에서 예외 상황이 발생할 수 있습니다.** 예제 코드는 public 필드 방식과 거의 동일하니 생략하겠습니다.

<br>

#### 3) 열거 타입

**원소가 하나인 열거 타입을 선언하는 방식**입니다. 

```java
public enum Elvis {

    INSTANCE;
  
    public void leaveTheBuilding() { ... }
}
```

열거 타입은 상수 하나당 인스턴스를 하나씩 만들어 public static final 필드로 공개합니다. 열거 타입의 인스턴스는 클라이언트가 직접 생성할 수 없고 런타임에 한 번만 생성됩니다. 

```java
Elvis elvis1 = Elvis.INSTANCE;
Elvis elvis2 = Elvis.INSTANCE;

System.out.println(elvis1 == elvis2);
/* 실행 결과
true
*/
```

public 필드 방식과 비슷하지만, 더 간결하고 추가 노력 없이 직렬화 할 수 있고 심지어 **아주 복잡한 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아줍니다.**

조금 부자연스러워 보일 수는 있으나 **대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법입니다.** 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없습니다. 다른 인터페이스를 구현하는 것은 가능합니다.
