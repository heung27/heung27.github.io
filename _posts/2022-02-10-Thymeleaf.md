---
layout: post
title: Thymeleaf
date: 2022-02-10 14:41 +0900
categories: [Server, Spring]
tags: [Thymeleaf, Spring, Backend, Web]
---



타임리프는 템플릿 엔진의 한 종류이다. 타임리프에 대해 알아보기 전에 템플릿 엔진에 대한 개념을 잡고 가겠다. 

<br>

## 템플릿 엔진(Template Engine)이란?

템플릿 엔진이란 템플릿 양식과 특정 데이터 모델에 따른 입력 자료를 합성하여 결과 문서를 출력하는 소프트웨어(또는 소프트웨어 컴포넌트)를 말한다. 다시 말해 **HTML과 데이터를 합성하여 결과물을 만들어 주는 도구**이다.

<br>

그러면 템플릿 엔진의 종류에는 어떤 것들이 있을까?

템플릿 엔진의 종류에는 크게 **서버 사이드 템플릿 엔진**과 **클라이언트 사이드 템플릿 엔진** 두 가지가 있다. 

- **서버 사이드 템플릿 엔진**

  서버에서 DB 혹은 API로 가져온 데이터를 미리 정의된 Template에 넣어 HTML을 그리는 역할을 수행한다. 즉, HTML 코드에서 고정적으로 사용되는 부분은 템플릿으로 만들어두고 동적으로 생성되는 부분만 템플릿의 특정 장소에 끼워 넣는 방식으로 동작할 수 있도록 한다.

  Ex) Thymeleaf, Freemarker, Groovy, Mustache, JSP, Velocity, jade4j, Handlebars(Handlebars.java), EJS(Embedded JavaScript Templates)  등

- **클라이언트 사이드 템플릿 엔진**

  HTML 형태로 코드를 작성할 수 있으며, 데이터를 받아와 DOM을 동적으로 그릴 수 있도록 한다. 

  Ex) Mustache, Squirrelly, Handlebars(Handlebars.js) 등

<br>

이런 템플릿 엔진을 사용하는 이유는 세 가지가 있다. 

1. 기존의 HTML에 비해 문법이 간단하여 많은 코드를 줄일 수 있다.
2. 데이터만 바뀌는 경우가 많이 때문에 재사용성이 높다.
3. 유지 보수가 용이하다.

<br>

## 타임리프(Thymeleaf)란?

타임리프는 템플릿 엔진의 한 종류이다.  JSP, Freemarker와 같이 **백엔드 쪽에서 클라이언트에게 응답할 브라우저 화면을 만들어 주는 역할**을 한다. 

<br>

아래와 같은 세 가지의 **특징**을 가지고 있다.

1. **서버 사이드 HTML 렌더링(SSR)**

   타임리프는 백엔드 서버에서 HTML을 동적으로 렌더링 하는 용도로 사용된다.

2. **네츄럴 템플릿**

   타임리프는 순수 HTML을 최대한 유지하는 특징이 있다. 타임리프로 작성한 파일은 HTML을 유지하기 때문에 웹 브라우저에서 파일을 직접 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다.

   JSP를 포함한 다른 뷰 템플릿들은 해당 파일을 열면, 예를 들어서 JSP 파일 자체를 그대로 웹 브라우저에서 열어보면 JSP 소스코드와 HTML이 뒤죽박죽 섞여서 웹 브라우저에서 정상적인 HTML 결과를 확인할 수 없다. 오직 서버를 통해서 JSP가 렌더링 되고 HTML 응답 결과를 받아야 화면을 확인할 수 있다.

   반면에 타임리프로 작성된 파일은 해당 파일을 그대로 웹 브라우저에서 열어도 정상적인 HTML 결과를 확인할 수 있다. 물론 이 경우 동적으로 결과가 렌더링 되지는 않는다. 하지만 HTML 마크업 결과가 어떻게 되는지 파일만 열어도 바로 확인할 수 있다.

   이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿(natural templates)이라 한다.

3. **스프링 통합 지원**

   타임리프는 스프링과 자연스럽게 통합되고, 스프링의 다양한 기능을 편리하게 사용할 수 있게 지원한다. 

<br>

지금까지 템플릿 엔진과 타임리프의 기본 개념을 알아봤다. 사실 타임리프의 사용법까지 정리하려고 했지만 글이 너무 길어질 것 같아 공식 문서 링크로 대체한다. 정리가 정말 잘 되어있으니 필요할 때마다 접속해 보시는 것을 추천한다.

- 공식 사이트: [https://www.thymeleaf.org/](https://www.thymeleaf.org/)
- 공식 메뉴얼 - 기본 기능: [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)
- 메뉴얼 - 스프링 통합: [https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)

<br>

## Reference

- [백엔드 웹 개발 활용 기술 - 인프런](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2#)
- [Thymeleaf란 - MAENCO](https://maenco.tistory.com/entry/Thymeleaf-Thymeleaf%EB%9E%80)
