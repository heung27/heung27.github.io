---
layout: post
title: Item 8. finalizer와 cleaner 사용을 피하라 (미완성)
date: 2022-04-30 14:37 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, finalizer, cleaner]
---



Effective Java의 여덟 번째 아이템 "finalizer와 cleaner 사용을 피하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Java는 finalizer와 cleaner라는 두 가지 **객체 소멸자**를 제공합니다. 본문에서 finalizer와 cleaner가 무엇이고 어떤 문제점을 가지고 있는지, 그리고 어떤 쓰임새가 있는지 자세히 알아보겠습니다. 

<br>

## 1. finalizer와 cleaner란?

### finalizer

finalize 메서드는 Object 클래스에서 제공하는 기본 메서드입니다. 어떤 클래스든지 오버라이드 할 수 있고, 해당 클래스의 객체가 GC의 대상이 될 때 호출됩니다. 보통 객체가 없어지기 전 다른 연관 자원을 정리하는 용도로 사용합니다.

```java
public class Room {
  
    ...

    @Override
    protected void finalize() throws Throwable {
        System.out.println("run finalizer");
    }
}
```

Room 클래스는 finalize 메서드를 오버라이드 했습니다. 이제 객체가 GC의 대상이 되어 메모리가 회수될 때 finalize 메서드가 호출 됩니다. 아래는 실제로 finalize 메서드가 호출되는지 확인하기 위해 작성한 클라이언트 코드입니다.

```java
new Room();
System.gc();

/* 실행 결과
run finalizer
*/
```

Room 클래스의 객체를 만들고 GC를 동작시켰습니다(`System.gc()`는 실제로 GC의 동작을 보장하지는 않습니다). 예상했던 것 처럼 finalize 메서드가 호출되는 것을 확인할 수 있습니다.

그러나 finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요합니다. 나름의 쓰임새가 몇 가지 있긴 하지만 기본적으로 쓰지 말아야 합니다. 그래서 Java 9에서는 finalizer를 deprecated API로 지정하고 cleaner를 그 대안으로 소개했습니다. 



### Cleaner

Java 9에서 도입된 소멸자로, 생성된 Cleaner가 더 이상 사용되지 않을 때 등록된 스레드에서 정의된 클린 작업을 수행합니다.

```java
public class Room {

    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("run cleaner");
            numJunkPiles = 0;
        }
    }

    public Room(int numJunkPiles) {
        State state = new State(numJunkPiles);
        cleaner.register(this, state);
    }
}
```

State 클래스의 numJunkPiles는 Room 클래스의 객체가 회수될 때 반드시 수거해야하는 자원입니다. State는 Runnable을 구현하여 run 메서드를 오버라이드 하였습니다. 그리고 Room의 객체가 생성될 때 State 객체가 Cleaner에 등록됩니다.  이제 Room의 객체가 GC의 대상이 되어 메모리가 회수될 때 Cleaner에 등록된 State 객체의 run 메서드가 호출될 것입니다.

```java
new Room(99);
System.gc();

/* 실행 결과
run cleaner
*/
```

Room 클래스의 객체를 만들고 GC를 동작시켰습니다(보장하지는 않습니다). 역시 예상했던 것 처럼 run 메서드가 호출된 것을 확인할 수 있습니다.

cleaner은 finalizer 보다 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요합니다. 구체적으로 어떤 문제가 있길래 사용을 피하라는 걸까요? 아래에서 finalizer와 cleaner의 문제점을 하나하나 알아보겠습니다.

<br>

## 2. finalizer와 cleaner의 문제점

### 2.1 즉시 수행된다는 보장이 없다.

finalizer와 cleaner는 즉시 수행된다는 보장이 없습니다. 객체에 접근할 수 없게 된 후 finalizer와 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없습니다. 즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없습니다.

예를들어 파일 닫기를 finalizer와 cleaner에 맡기면 중대한 오류를 일으킬 수 있습니다. 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문입니다. 시스템이 finalizer나 cleaner 실행을 게을리해서 파일을 계속 열어 둔다면 새로운 파일을 열지 못 해 프로그램이 실패할 수 있습니다.

finalizer와 cleaner가 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며, 이는 GC 구현마다 천차만별입니다. finalizer나 cleaner 수행 시점에 의존하는 프로그램의 동작 또한 마찬가지입니다.

finalizer 스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아서 실행될 기회를 제대로 얻지 못 할 수 있습니다. 자바 언어 명세는 어떤 스레드가 finalizer를 수행할지 명시하지 않으니 이 문제를 예방할 보편적인 해법은 없습니다. 한편, cleaner는 자신을 수행할 스레드를 제어할 수 있다는 면에서 조금 낫습니다. 하지만 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있으니 즉각 수행되리라는 보장은 없습니다.

<br>

### 2.2 수행 여부를 보장하지 않는다.

자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않습니다. 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있습니다. 따라서 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 됩니다. 예를 들어 데이터베이스 같은 공유 자원의 영구 락(lock) 해제를 finalizder나 cleaner에 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 것입니다.

<br>

### 2.3 finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.

finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료됩니다. 잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있습니다. 그리고 다른 스레드가 이처럼 훼손된 객체를 사용하려 한다면 어떻게 동작할지 예측할 수 없습니다. 보토으이 경우엔 잡지 못한 예외가 스레드를 중단시키고 스택 추적 내역을 출력하겠지만, 같은 일이 finalizer에서 일어난다면 경고조차 출력하지 않습니다.

그나마 cleane를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않습니다.

<br>

### 2.3 성능 문제를 동반한다.

finalizer와 cleaner는 심각한 성능 문제를 동반합니다.

<br>

### 2.4 finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다.

finalizer 공격 원리는 간단합니다. 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되자 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 됩니다. 있어서는 안 될 일입니다. 이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있습니다. 이렇게 일그러진 객체가 만들어지고 나면, 이 객체의 메서드를 호출해 애초에는 허용되지 않았을 작업을 수행하는 건 일도 아닙니다. 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지도 않습니ㅏㄷ. 이러한 공격은 끔찍한 결과를 초래할 수 있스빈다. final 클래스들은 그 누구도 하위 클래스를 만들 수 없으니 이 공격에서 안전합니다. final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선업해야 합니다.

<br>

## 3. finalizer와 cleaner의 대체제 - AutoCloseable

파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에서 finalizer나 cleaner를 대신해줄 묘안은 무엇일까요? 그저AutoCloseable을 구현(implement)해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 됩니다(일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 합니다). 구체적인 구현법과 관련하여 알아두면 좋을 것이 하나 있습니다. 각 인스턴스는 자신이 닫혔는지를 추적하는 것이 좋습니다. 다시 말해, close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 IllegalStateException을 던지는 것입니다.

<br>

## 3. finalizer와 cleaner의 쓰임새

finalizer와 cleaner는 대체 어디에 쓰는 것일까요? 적절한 쓰임새가 (아마도) 두 가지 있습니다.

### 3.1 안전망

자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할입니다. finalizer나 cleaner가 직시 (혹은 끝까지) 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안 하는 것보다는 낫습니다. 이런 안정망 역할의 finalizer를 작성할 때는 그럴만한 값어치가 있는지 심사숙고하는 것이 좋겠습니다. 자바 라이브러리 일부 클래스는 안정망 역할의 finalizer를 제공합니다. FileInputStream, FileOutputStream, ThreadPoolExecutor가 대표적입니다.

### 3.2 네이티브 피어

finalizer나 cleaner를 적절히 활용하는 두 번째 예는 네이티브 피어(native peer)와 연결된 객체에서 입니다. 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말합니다. 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못합니다. 그 결과 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못합니다. finalizer나 cleaner가 나서서 처리하기에 적당한 작업입니다. 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당됩니다. 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 합니다. 

<br>

## 4. 사용 예제

- 예제

```java
public class Room implements AutoCloseable {

    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {

        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    private final State state;

    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

방(room) 자원을 수거하기 전에 반드시 청소(clean)해야 한다고 가정해보겠습니다. Room 클래스는 AutoCloseable을 구현합니다. static으로 선언 된 중첩 클래스인 State는 cleaner가 방을 청소할 때 수거할 자원들을 담고 있습니다. 이 예에서는 단순히 방 안의 쓰레기 수를 뜻하는 numJunkPiles 필드가 수거할 자원에 해당합니다. 더 현실적으로 만들려면 이 필드는 네이티브 피어를 가리키는 포인터를 담은 final long 변수여야 합니다. State는 Runnable을 구현하고, 그 안의 run 메서드는 cleanable에 의해 딱 한 번만 호출될 것 입니다. 이 cleanable 객체는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻습니다. 

run 메서드가 호출되는 상황은 둘 중 하나 입니다. 보통은 Room의 close 메서드를 호출할 때 입니다. close 메서드에서 Cleanable의 clean을 호출하면 이 메서드 안에서 run을 호출합니다. 혹은 GC가 Room을 회수할 때까지 클라이언트가 close를 호출하지 않는다면, cleaner가 State의 run 메서드를 호출해줄 것입니다. 

- 중첩 클래스 - 바깥 객체 참조 주의

State 인스턴스는 절대로 Room 인스턴스를 참조 해서는 안 됩니다. Room 인스턴스를 참조할 경우 순환참조가 생겨 GC가 Room 인스턴스를 회수해갈 (자동 청소될) 기회가 오지 않습니다. State가 정적 중첩 클래스인 이유가 여기에 있습니다. 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문입니다. 이와 비슷하게 람다 역시 바깥 객체의 참조를 갖기 쉬우니 사용하지 않는 것이 좋습니다.

- 예제 비교

앞서 이야기 한 대로 Room의 cleaner은 단지 안전망으로 쓰였습니다. 클라이언트가 모든 Room 생성을 try-with-resources 블록으로 감쌌다면 자동 청소는 전혀 필요하지 않습니다. 다음은 클라이언트 코드 예제입니다. 

```java
try (Room myRoom = new Room(7)) {
  System.out.println("안녕");
}

/* 실행 결과
안녕
방 청소
*/
```

기대한 대로 "안녕"을 출력한 후, 이어서 "방 청소"를 촐력합니다. try-with-resources를 사용하지 않았다면 어떻게 될 까요?

```java
new Room(7);
System.out.println("안녕");

/* 실행 결과
안녕
*/
```

"방 청소"는 한 번도 출력되지 않습니다. 앞서 "예측할 수 없다"고 말한 상황입니다. 

```java
new Room(7);
System.out.println("안녕");
System.gc();

/* 실행 결과
안녕
방 청소
*/
```



<br>

## 5. 핵심 정리

cleaner(Java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.

<br>

## 6. Related Posts

- try-with-resource (Item 9)
- 중첩 클래스 (Item 24)
