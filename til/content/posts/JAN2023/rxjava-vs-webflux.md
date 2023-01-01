---
title: "Rxjava vs Webflux"
date: 2023-01-01T21:48:24+09:00
draft: false
author: redjen
---

# RxJava vs Spring Webflux

리액티브 프로그래밍 == 프로그래밍 패러다임.
객체 지향 프로그래밍, 함수형 프로그래밍, 절차형 (procedural) 프로그래밍처럼 리액티브 프로그래밍 역시 하나의 패러다임에 불과하다.

Reactive Stream은 specification.
Reactive Stream은 4가지 인터페이스를 제공하는 API. 
1. Publisher: 길이에 제한이 없는 sequence element들의 공급자.
2. Subscriber: publisher가 전달한 element를 수신하고 소비하는 주체
3. Subscription: publisher-subscriber의 1대1 라이프사이클을 의미
4. Processor: subscriber와 publisher가 계약을 맺는 processing stage를 대변. 

이렇게 정의해 놓은 reactive stream을 구현하는 JVM 기반 프레임워크에는
- Akka Streams Framework
- Ratpack
- Vert.x
- ReactiveX (RxJava 2.x + Reactor)
- Java 1.9 Flow 클래스들

ReactiveX는 Observer, Iterator 패턴과 함수형 프로그래밍의 핵심 아이디어로부터 조합되어 탄생된 프레임워크.
Observer 패턴의 연장선 상에서 데이터 / 이벤트들의 시퀀스나 시퀀스들을 명시적으로 합칠 때 low-level threading, synchronization, thread safety, non-blocking IO 에 대한 걱정 없이 사용할 수 있는 연산자를 지원한다.

RxJava 2.0과 Reactor는 ReactiveX 프로젝트에 기반한 결과물.
그리고 스프링 Webflux가 Reactor를 내부적으로 사용하는 것.
둘 다 Reactive Stream의 구현체인 것은 동일하다.

https://stackoverflow.com/questions/56461260/java-spring-webflux-vs-rxjava

