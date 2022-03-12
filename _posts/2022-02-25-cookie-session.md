---
layout: post
title: Cookie & Session
date: 2022-02-25 23:01 +0900
categories: [Server, Web]
tags: [HTTP, Cookie, Session]
---



웹 서버 개발에 기본적인 개념인 Cookie(쿠키)와 Session(세션)에 대해 정리한다. 

들어가기 전에 웹 통신에 사용되는 프로토콜인 HTTP의 특징을 알아보고 쿠키와 세션이 필요한 이유를 먼저 이해하자.



## HTTP의 특징

#### 비연결성(Connectionless)

클라이언트에서 서버에 요청을 보내면 서버는 클라이언트에 응답을 하고 연결을 끊어 버린다. 

#### 무상태(Stateless)

요청에 응답하고 연결을 끊기 때문에 서버는 클라이언트의 상태를 저장하지 않는다. 

<br>

이러한 특징 때문에 서버는 같은 사용자(클라이언트)가 보내는 요청도 매번 새로운 사용자로 인식하게 되는 문제가 있다. 

그런데 우리가 사용하는 웹사이트를 생각해보자. 로그인을 한 번 하고나면 그 사이트에서는 다시 로그인할 필요가 없이 여러 페이지의 다양한 기능을 이용할 수 있다. 마치 서버가 사용자의 정보를 유지하고 있는 것처럼 보인다. 

이처럼 HTTP의 비연결성과 무상태를 보완하여 Statefule한 서비스를 제공하기위해 사용하는 것이 쿠키와 세션이다.

<br>

#### HTTP 통신을 사용하는 이유

HTTP를 사용하여 Stateful한 서비스를 제공하기 위해 쿠키와 세션을 이용한다고 했다. 그렇다면 굳이 Stateless한 HTTP 통신을 사용하는 이유는 뭘까. 애초에 Stateful한 통신 방법을 사용하면 되지 않나?

이 의문의 답은 **서버의 확장성**에 있다. 

사용자가 늘어나 증가한 트래픽을 분산하기 위해 scale out하는 경우를 생각해보자.

**Stateful 서버**는 사용자의 정보를 메모리에 저장하고 있다. 추가되는 새로운 서버에는 사용자의 정보가 저장되어 있지 않기 때문에 정상적인 응답이 불가능하다. (방법이 없는 것은 아니지만 구현하기 까다롭다고 한다)

**Stateless 서버**는 사용자의 정보를 외부 저장소에 저장하고 있다. 새로운 서버가 추가된다고 하더라도 외부 저장소에서 사용자의 정보를 불러와 사용할 수 있기 때문에 정상적인 응답이 가능하다.

>그러나 Stateless가 장점만 가지고 있는 것은 아니다. 서버로 요청을 보낼때, 매 요청시마다 상태 정보를 전달해야 하기 때문에 그만큼 네트워크 리소스를 소비하게 된다. 
>
>Stateful와 Stateless에 대한 자세한 내용은 추후 다른 포스팅에서 정리한다.

<br>

## Cookie (쿠키)

쿠키는 웹 브라우저에 저장되는 key-value 쌍의 작은 데이터 파일이다. 



### 특징

#### 장점

#### 단점



### 동작 과정



<br>

## Session (세션)





### 특징

#### 장점

#### 단점



### 동작 과정



<br>

## Cookie  vs  Sessoin



<br>

## Reference

- [https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2#](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2#)
- [https://junhyunny.github.io/information/cookie-and-session/](https://junhyunny.github.io/information/cookie-and-session/)
- [https://junshock5.tistory.com/83](https://junshock5.tistory.com/83)
