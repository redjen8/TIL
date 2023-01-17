---
title: "Mockito의 Mock, Mockbean 어노테이션의 차이"
date: 2023-01-17T22:09:25+09:00
draft: false
author: redjen
---

junit 테스트 수행 도중 `@MockBean` 어노테이션이 없어서 테스트에 필요한 서비스 의존성 주입에 실패한 트러블 슈팅을 하였다.

Mockito 라이브러리의 `@Mock`과 `@MockBean`은 어떤 차이가 있을까?

https://stackoverflow.com/questions/44200720/difference-between-mock-mockbean-and-mockito-mock

## `@Mock`과 `Mockito.mock(someService.class)`

`@Mock`과 `Mockito.mock(someService.class)`는 모두 Mockito 라이브러리의 기능이다.
클래스나 인터페이스를 mocking하여 행동을 검증하고 기록하기 위한 용도로 사용된다.
어노테이션인 `@Mock`은 단지 더 짧기 때문에 더 많이 사용되는 경향이 있다.

테스트 실행을 위해 Mockito의 `@Mock` 어노테이션을 사용한다면 테스트 도중에 `MockitoAnnotation.initMocks(this)` 정적 메서드가 실행된다.

테스트 간 충돌하는 것을 막기 위해서 테스트 실행 전에 `@Before` 를 통해서 Mockito 객체의 정적 메서드를 전역으로 사전에 실행하는 것이 권고 된다.

Mockito 어노테이션을 활성화하는 다른 방법으로는 테스트 클래스를 `@Runwith` 어노테이션으로 `MockitoJUnitRunner`로 지정하여 상기의 정적 메서드 외의 다른 유용한 작업도 동시에 할 수 있게 할 수 있다.

## 스프링 부트의 `@MockBean`에 대하여

`@MockBean`에 대한 내용은 spring-boot-test 라이브러리 안에 포함되어 있다.
Mockito의 컴포넌트 mocking을 Spring의 ApplicationContext에 추가할 수 있게 해주는 기능이다.
추가하려는 대상이 어플리케이션의 컨텍스트에 있다면 해당 빈을 Mock으로 replace(!) 하는 마법과도 같은 작업을 한다.
만약 추가하려는 대상이 어플리케이션 컨텍스트에 없다면 해당 mock을 컨텍스트에 빈으로 등록한다.

## 차이점 정리 : 언제, 무엇을 써야 할까?

유닛 테스트는 다른 컴포넌트에 영향을 받지 않는 환경에서 컴포넌트를 테스트하기 위한 목적을 가진다.
이를 위해서는 개발자의 개인 환경에서 얼마든지 여러 번 거리낌 없이 반복적으로 실행할 수 있도록 빠른 테스트 시간이 충족되어야 한다.

스프링 부트의 컨테이너의 어떠한 의존성도 필요로 하지 않는 테스트를 작성한다면 클래식하고 plain한 Mockito를 사용하면 된다. 이렇게 된다면 두 가지 장점이 있다.
1. 필요 없는 인스턴스 생성과 의존성 주입이 일어나지 않기 때문에 테스트를 굉장히 빠르게 만들 수 있다.
2. 테스트 대상 컴포넌트의 기능에 대한 테스트가 가능하다.

하지만 만약 스프링 부트 컨테이너에 의존한 테스트를 수행하거나 테스트 도중에 다른 컨테이너 빈을 Mocking해야 한다면 `@MockBean`을 사용해야 한다.

`@MockBean`을 사용해야 하는 대표적인 케이스가 `@WebMvcTest`를 사용한 컨트롤러 컴포넌트 테스트를 수행할 때이다. 테스트 대상 컨트롤러 뿐만 아니라 웹 클라이언트에 대한 의존성도 주입받아야 하기 때문이다. 사실 이 경우에는 컨트롤러 단일 컴포넌트에 대한 완벽한 격리 테스트 환경을 만들기 불가능한 케이스에 보는 것이 맞다는 생각이 든다. 

나는 `WebTestClient`를 사용한 컨트롤러 테스트를 수행하려 했었는데, `@MockBean` 을 사용하지 않고 `@Mock`을 사용했기 때문에 스프링 어플리케이션의 컨텍스트에서 의존성을 찾을 수 없다는 에러 메시지와 함께 테스트가 실행되지 못한 것이었다.