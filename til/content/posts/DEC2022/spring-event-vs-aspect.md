---
title: "Spring Event vs Aspect"
date: 2022-12-20T20:11:17+09:00
draft: false
author : redjen
---
# Spring Event vs Aspects

## Spring Event

회사 코드를 보던 중 @EventListener 어노테이션을 보게 되었고,
Spring 프레임워크에서도 비동기적으로 이벤트를 invoke할 수 있다는 것을 찾아볼 수 있었다. 

Spring에서는 데이터를 DI를 통해서 전달할 수도 있지만
Bean과 Bean 사이에 데이터를 Event를 통해 전달할 수 있는 방법을 제공해준다. 

### Event 발생하기 (Publish)

ApplicationContext가 상속하는 인터페이스 중 하나인 ApplicationEventPublisher를 사용하여 이벤트를 Publish 한다.

### Event 수신하고 처리하기 (Listen)

ApplicationListener\<T\>를 상속하여 ApplicaionContext 안에서 전달되는 이벤트를 소비할 수 있다.

### @EventListener 어노테이션 기반

@EventListener 어노테이션을 이벤트를 Listen하고 처리할 메소드에 붙여서 파라미터 값으로 이벤트 객체를 받도록 설정할 수 있다.
보내질 이벤트 객체는 어떤 추가 객체 의존 관계 없이 EventListener에게 전달된다는 점이 신기하다

### 처리할 핸들러에 순서 부여하려면?

@Order 어노테이션을 통해 이벤트 리스너 간 우선순위를 지정해줄 수 있다.
Order 어노테이션을 통해 설정되는 우선순위 값이 클수록 우선 순위가 낮아진다 (더 나중에 처리된다)

### 결국 핵심은 비동기

이벤트 핸들러는 기본적으로는 비동기로 실행된다 - 그럼 이벤트 처리를 위한 쓰레드가 홀드 될 수도 있나??
이벤트 처리 메소드에 @Async 어노테이션을 붙여서, 이벤트를 이벤트답게 사용할 수 있도록 설정하자.

## Aspect

이 EventListener와 Spring Event에 대해 공부하다보니, Spring의 Aspect와도 많이 비슷하다는 생각이 들었다.
기본적으로 어떤 코드가 실행되면 - 그 코드 실행 전 후 시점에 추가적인 코드를 실행시킨다는 점에서 특히 비슷하다고 느껴졌다.

[그래서 찾아보니](https://stackoverflow.com/questions/38437389/spring-events-vs-aspects) 다음과 같은 차이점이 있었다.

## 그래서 차이점이 뭔데?

### AOP는 코드 안에 로직을 주입할 수 있게 해준다.

pointcut을 통해 로직이 주입될 위치를 설정한다. Aspect는 로직이 주입될 위치 자체는 달라질 수가 있지만 기본적으로 모든 위치에서 동일한 로직이 실행된다. 
대표적으로 Logback이나 Log4j와 같은 로깅 구현체를 사용하는 것이 그렇다.
로그를 남길 대상과 그 설정은 다르게 할 수가 있지만, 로그를 남긴다는 로직 자체는 전부 동일하다.

### Event Handling은 Listener에게 알려주는 것을 목적으로 한다.

Listener가 로직을 수행하지만, 어플리케이션 전체의 관점에서 바라보았을 때에는 서로 다른 로직을 수행할 수 있다는 점이 장점이다.
하지만 코드 전체에 걸쳐 관심사를 분리할 수 있도록 해주는 Aspect와는 달리 Event Handling은 그 적용 범위가 Application context나 HTTP request에 한정된다는 차이점도 있다.

두 기술의 행동적인 관점에서 보았을 때 (Behavioral Perspective) Aspect와 Event는 정반대이다. 
- Aspects는 로직 실행 시점이 되었을 때 코드 조각을 **주입**한다.
- Event는 로직 실행 시점이 되었을 때의 **응답**으로 코드 조각을 실행한다.

전체 어플리케이션의 관점에서는
- 동일한 Aspect는 여러 번 실행될 수 있다. (로깅처럼)
- 하지만 EventHandler는 특정한 Event가 발생되었을 때에만 실행된다. 동일한 이벤트여도 여러 EventHandler에 의해 처리될 수 있다. 

HTTPRequest에 한정해서 발생하는 Aspect와 Event는 비슷하다고 느껴진다.
하지만 전반적으로 AOP가 Listener 패턴보다 훨씬 더 강력하다. 다만 Listener 패턴이 더 dynamic 하게 비즈니스 로직을 수행할 수 있다고 느껴진다. 

