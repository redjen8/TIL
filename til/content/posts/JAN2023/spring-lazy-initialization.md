---
title: "Spring Lazy Initialization"
date: 2023-01-24T20:00:40+09:00
draft: false
author: redjen
---

Spring boot 2에서는 Spring bean에 Lazy Initialization을 적용할 수 있도록 지원한다.
Lazy initialization이 무엇인지, 어떨 때 사용해야 하는지 정리해봤다.

https://www.baeldung.com/spring-lazy-annotation

## 스프링 프레임워크에서의 싱글턴 패턴

스프링은 모든 싱글턴 빈들을 어플리케이션 컨텍스트가 초기화되고 부트스트랩되는 기간에 eager하게 생성하는 것이 default 설정이다. 

런타임에 싱글턴 빈들을 생성하게 된다면 부트스트랩 시간에 미리 예방할 수 있는 에러를 피할 수 없기 때문에 기본값으로는 lazy가 아닌 eager하게 빈들을 생성하는 것이다.

## Lazy initialization vs Eager initialization

싱글턴 패턴은 객체를 단 한번 생성하는 것을 원칙으로 한다.

여기서 문제가 발생하게 되는데, **그럼 객체는 언제 생성해야 할까?**
객체가 꼭 필요하게 되는 시점에 생성하는 것이 Lazy initialization,
객체가 필요하지 않아도 초기에 전부 생성하는 것이 Eager initialization이다.

Lazy initialization을 사용한다면 당장 사용하지 않을 객체에 대해 메모리를 사용하지 않아도 되므로 리소스적인 면에서 경제적이다. 다만 나중에 빈을 생성하게 되면 빈 생성 시점의 초기 설정 값에 이상이 있어도 부트스트랩 시간이 아닌 어플리케이션의 런타임에 이를 알 수 있다는 단점이 있다.

때문에 많은 경우 생성 비용과 관리가 비싼 객체는 Lazy하게 생성하는 것이 이득이다.

## @Lazy 어노테이션

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Lazy.html

스프링 프레임워크에서는 스프링 빈에 대해 편리하게 어노테이션을 통한 Lazy initialization을 지원한다. 즉 `@Component` 나 `@Bean` 어노테이션이 아니라면 스프링이 그 생명 주기를 관리할 수 없기 때문에 해당 객체는 Eager하게 초기화 된다.

`@Bean` 어노테이션이 붙은 스프링 빈 객체는 다른 빈에 의해 참조되거나 명시적으로 `BeanFactory`에 의해 생성되지 않는다면 초기화되지 않는다. 

`@Bean` 어노테이션이 `@Configuration` 클래스와 같이 사용될 경우, 모든 `@Configuration`안에 들어 있는 모든 `@Bean`의 메서드들이 lazy하게 초기화된다. 

컴포넌트의 초기화 시점 설정 외에도 `@Lazy` 어노테이션은 `@Autowired`나 `@Inject` 로 마킹된 의존성 주입 포인트들에서도 사용될 수 있다. 해당 컨텍스트 안에서 `@Lazy` 어노테이션은 모든 의존성에 대한 lazy-resolution 프록시를 생성한다.

이 lazy-resolution 프록시는 항상 의존성이 주입된다. 만약 대상 의존성이 존재하지 않는다면 실행 중에 발생하는 예외를 통해서만 대상을 알 수 있기 때문이다. 

결과적으로 의존성 주입의 포인트들은 선택적 의존성에 대한 직관적이지 않은 행동을 불러 일으킬 수 있다. 만약 프로그래밍적으로 lazy initialization을 통해 동일한 기능적 이득을 얻고자 한다면 `ObjectProvider`를 사용하는 것이 좋다.