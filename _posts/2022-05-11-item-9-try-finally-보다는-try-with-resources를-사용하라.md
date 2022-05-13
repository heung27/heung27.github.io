---
layout: post
title: Item 9. try-finally보다는 try-with-resources를 사용하라
date: 2022-05-11 21:45 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, try-finally, try-with-resources]
---



Effective Java의 아홉 번째 아이템 "try-finally보다는 try-with-resources를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 0. 들어가며

Java 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 있습니다. 예를 들어 InputStream, OutputStream, java.sql.Connection 등이 있죠. 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 합니다.

이런 자원 중 상당수가 안정망으로 finalizer를 활용하고 있습니다. 하지만 이전 포스팅에서 알아봤듯이 finalizer는 그리 믿을만하지 못합니다. 본문에서 자원이 제대로 닫힘을 보장하는 전통적인 방법 try-finally와 보다 개선된 방법인 try-with-resources에 대해 알아보겠습니다.

<br>

## 1. try-finally

다음은 입력받은 경로의 파일을 읽어들이는 메서드입니다.

```java
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}
```

위 예제는 try-finally를 사용해 BufferedReader 자원의 닫힘을 보장합니다. 깔끔하고 괜찮아 보이는데, 만약 닫아야 하는 자원이 둘 이상일 때는 어떻게 해야 할까요?

다음은 파일을 읽어 복사하는 메서드입니다.

```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0) {
        out.write(buf, 0, n);
      }
    } finally {
      out.close();
    }
  } finally {
    in.close();
  }
}
```

닫아햐 하는 자원이 InputStream, OutputStream 두 개가 되었습니다. 이때는 try-finally를 중첩으로 사용해야 합니다. 보이는 것과 같이 코드가 많이 지저분해집니다. 자원이 많아질수록 걷잡을 수 없겠죠.

그런데 왜 중첩으로 사용해야 하는지 의문이 들 수 있습니다. 하나의 try-finally로는 안될까요? 아래처럼 말이죠.

```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  OutputStream out = new FileOutputStream(dst);

  try {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  } finally {
    in.close();
    out.close();
  }
}
```

얼핏 보면 문제없어 보입니다. 하지만 finally 블록에서도 예외가 발생할 수 있다는 것을 잊지 말아야 합니다. `in.close()`에서 예외가 발생하면 `out.close()`는 실행되지 않습니다. 자원이 닫히지 않는 것이죠. 물론 위의 예제에서는 OutputStream이 안전망으로 finalizer를 사용하기 때문에 finalize 메서드가 호출되어 자원이 닫힐 것입니다. 하지만 finalizer보다는 직접 닫아주는 게 좋겠죠.

try-finally는 복잡해지는 코드 외에 한 가지 더 문제를 가지고 있습니다. 예외가 try 블록과 finally 블록 모두에서 발생할 수 있다는 것입니다. 예를 들어 try 블록에서 어떤 예외가 발생했고 연쇄적으로 finally 블록에도 예외가 발생했다고 가정하면, try 블록에서 발생한 첫 번째 예외가 finally에서 발생한 두 번째 예외에 가려지게 됩니다. 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어 실제 시스템에서의 디버깅이 어려워집니다.

이러한 문제들은 Java 7부터 추가된 try-with-resources를 사용하면 해결할 수 있습니다. 

<br>

## 2. try-with-resources

먼저 이 구조를 사용하기 위해서는 해당 자원이 AutoCloseable 인터페이스를 구현해야 합니다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

Java 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스은 이미 AutoCloseable을 구현하거나 확장해뒀습니다. 예제에서 사용된 InputStream와 OutputStream 역시 AutoCloseable을 구현하고 있습니다. 

이제 사용법을 알아봐야겠죠. 위에서 보았던 메서드들을 try-with-resources 방식으로 수정해 보겠습니다.

```java
static String firstLineOfFile(String path) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
  }
}
```

```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
       OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  }
}
```

try-with-resources는 코드에서는 보이지 않지만 내부적으로 close 메서드를 호출합니다. try-finally에 비해 훨씬 짧아져 읽기 편합니다. 그리고 문제를 진단하기도 좋습니다. try 블록과 close 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 try 블록에서 발생한 예외가 기록됩니다. 숨겨진 예외들도 그냥 버려지지는 않고, 스택 추적 내역에 '숨겨졌다(suppressed)'는 꼬리표를 달고 출력됩니다. 또한, Java 7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드로 가져올 수도 있습니다.

<br>

## 3. 핵심 정리

꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.

<br>

## 4. Related Posts

- [finalizer (Item 8)](https://heung27.github.io/posts/item-8-finalizer%EC%99%80-cleaner-%EC%82%AC%EC%9A%A9%EC%9D%84-%ED%94%BC%ED%95%98%EB%9D%BC/)
