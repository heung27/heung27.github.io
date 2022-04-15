---
layout: post
title: Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
date: 2022-04-15 21:15 +0900
categories: [Book, Effective Java]
tags: [Java, Effective Java, Constructor]
---



Effective Java의 네 번째 아이템 "인스턴스화를 막으려거든 private 생성자를 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

책에서는 인스턴스화를 막아놓는 클래스의 예시로 유틸리티 클래스를 언급합니다. 본문에서 유틸리티 클래스에 대한 개념과 인스턴스화를 막는 방법을 알아보겠습니다.

<br>

### 유틸리티 클래스

유틸리티 클래스란 인스턴스 메서드와 인스턴스 변수를 일절 제공하지 않고, **정적 메서드와 정적 변수만을 제공하는 클래스**를 의미합니다. 유틸리티 클래스의 존재 의의는 클래스 본래의 목적인 '데이터와 데이터 처리를 위한 로직의 캡슐화'를 실행하는 것이 아닌, '비슷한 기능의 메서드와 상수를 모아서 캡슐화'를 하는 것에 있습니다. 

개발을 하다 보면 종종 유틸리티 클래스를 만들고 싶을 때가 있습니다. java.lang.Math와 java.util.Arrays처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓을 수 있고, java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해 주는 정적 메서드를 모아놓을 수도 있습니다. 또한 final 클래스와 관련한 메서드들을 모아놓을 때도 사용할 수 있습니다.

다음 예제는 org.springframework.util 패키지의 StringUtils의 일부 코드입니다.

```java
package org.springframework.util;

public abstract class StringUtils {
    private static final String[] EMPTY_STRING_ARRAY = new String[0];
    private static final String FOLDER_SEPARATOR = "/";
    private static final String WINDOWS_FOLDER_SEPARATOR = "\\";
    private static final String TOP_PATH = "..";
    private static final String CURRENT_PATH = ".";
    private static final char EXTENSION_SEPARATOR = '.';

    public static boolean hasLength(@Nullable CharSequence str) {
        return str != null && str.length() > 0;
    }
  
    public static boolean hasText(@Nullable CharSequence str) {
        return str != null && str.length() > 0 && containsText(str);
    }
  
    private static boolean containsText(CharSequence str) {
        int strLen = str.length();

        for(int i = 0; i < strLen; ++i) {
            if (!Character.isWhitespace(str.charAt(i))) {
                return true;
            }
        }

        return false;
    }
    
    ...
}
```

이러한 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 것이 아닙니다. 정적 멤버만을 가지고 있기 때문에 인스턴스 없이 모든 기능을 수행할 수 있습니다. 그런데 문제는 java는 **생성자를 명시해 주지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다**는 것입니다. 때문에 사용자는 이 생성자가 자동으로 생성된 것인지 구분할 수 없어 인스턴스를 만들 가능성이 있습니다. 이는 곧 **필요 없는 인스턴스를 생성하게 되어 메모리를 낭비**하는 것입니다.

그런데 위의 예제를 보면 abstract 키워드를 사용하여 추상 클래스로 정의되어 있습니다. 추상 클래스는 인스턴스를 만들 수 없기 때문에 문제가 없는 것처럼 보이는데요. **추상 클래스도 인스턴스화가 가능합니다.** 추상 클래스를 상속받는 하위 클래스를 만들어서 인스턴스화하면 그만입니다. 모든 하위 클래스의 생성자에는 상위 클래스의 생성자를 호출하는 코드가 숨겨져 있기 때문입니다.

<br>

### 해결 방법

그러면 유틸리티 클래스의 인스턴스화를 어떻게 막아야 할까요? 방법은 간단합니다. 컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을 때뿐이니 **private 생성자를 추가**해주면 됩니다. 

```java
public class UtilityClass {
  
    ...

    // 기본 생성자가 만들어지는 것을 막는다. (인스턴스 방지용)
    private UtilityClass() {
        throw new AssertionError();
    }
  
    ...
}
```

명시적 생성자가 private이니 클래스의 외부에서는 접근할 수 없습니다. 그리고 클래스 내부에서 실수로라도 생성자를 호출하는 것을 방지하기 위해 생성자를 호출했을 경우 AssertionError를 던지도록 구현합니다(꼭 에러를 던질 필요는 없습니다). 추가적으로 코드의 가독성을 위해 위의 예제처럼 적절한 주석을 달아주는 것을 권장합니다.

private 생성자를 만들면 **상속을 불가능하게 하는 효과**도 있습니다. 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버리는 것입니다. 

