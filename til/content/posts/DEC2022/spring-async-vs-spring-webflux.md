---
title: "Spring Async vs Spring Webflux"
date: 2022-12-29T16:21:15+09:00
draft: false
author: redjen
---

# Spring Async vs Spring Webflux

https://www.baeldung.com/spring-mvc-async-vs-webflux

기존 스프링 MVC에서는 Blocking 방식을 사용하고 있었고, Java 8에서 Future, CompletableFuture 와 같은 비동기 지원을 통해 한정적으로 비동기 요청 처리를 할 수 있었다.

MVC에서는 Servlet이 요청의 비동기적 dispatch를 맡는다. 기존 Servlet Filter들도 동일하게 동작한다. 

## Spring Webflux와의 차이점

Spring Async는 서블릿 3.0 스펙을 지원한다.
Spring Webflux는 서블릿 3.1 + 스펙을 지원한다.

- 스프링 Async IO 모델은 클라이언트가 블로킹 상태를 유지한채로 통신한다. 때문에 클라이언트의 성능에 영향을 받을 수 있다. 스프링 Webflux는 논브로킹 IO 모델을 제공한다.
- Spring Async에서 Request body나 request parts와 같이 요청에 딸려들어오는 데이터들을 읽는 동작이 blocking이다. Webflux에서는 non-blocking이다.
- Spring Async에서 Filter와 Servlet들은 동기적으로 동작한다. Webflux에서는 전체 통신 과정에 대해 비동기적으로 수행한다.
- Webflux는 Spring Async보다 더 넓은 웹 어플리케이션 서버에 대한 지원을 제공한다. (Netty, Undertow와 같은)

https://itembase.com/resources/blog/tech/spring-boot-2-spring-webflux

## Webflux의 단점

- 많은 현대의 라이브러리들이 reactive 스택을 상정하고 만들어지지 않는다. reactive web stack을 위한 새로운 라이브러리가 필요한데 공식 라이브러리가 아닌 경우도 있고 아예 없는 경우도 있다 (R2DBC도 최근 들어 공식이 되었으며, Reactive feign도 현재 공식이 아님)
- reactive 스택 프로그래밍은 난이도가 높다. 전통적인 명령형 프로그래밍 방식에 익숙한 사람들에게는 이해에 필요한 충분한 시간이 필요하다.
	- 디버깅하기 까다롭다는 단점도 있다. stack trace들이 엄청 길어지고, 문제를 일으키는 코드를 정확히 찾기 어렵다.
- 대부분의 문제는 비동기 non blocking 방식으로 바꾼다고 해서 해결되는 문제가 아닌 경우가 많다. 현실 세계의 많은 문제는 이미 Spring MVC 모델로도 충분히 풀 수 있는 경우가 많지만 webflux가 이들을 마법처럼 해결해주지는 않는다. 오히려 문제를 더 복잡하고 심각하게 꼬게 할 수도 있다. 

개인적으로는 Project Loom이 실현되어 Java에서도 경량 쓰레드를 자유롭게 사용할 수 있다면 자바 생태계에 또 다른 혁명이 오지 않을까 생각한다.
