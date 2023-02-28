---
title: "Jdk Proxy vs Cglib Proxy"
date: 2023-02-28T19:02:20+09:00
draft: false
author: redjen
---

스프링 프레임워크는 AOP의 장점을 아주 잘 활용하고 있는 프레임워크이다.

AOP에 의해 생성된 객체는 Aspect Contract를 실행하기 위해 사용된다.
스프링에서 사용하는 수많은 프록시 객체들은 JDK 동적 프록시나 CGLIB 프록시가 될 수 있다.

두 프록시 방법에는 어떤 차이가 있고, 스프링 프레임워크에서는 어떤 것을 언제 사용할까?

https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch08s06.html

https://stackoverflow.com/questions/10664182/what-is-the-difference-between-jdk-dynamic-proxy-and-cglib

## JDK Dynamic Proxy

JDK Dynamic Proxy는 다음의 특징을 가지고 있다.
- 인터페이스를 통해서만 프록시할 수 있다.
- 때문에 타깃 클래스는 인터페이스를 구현해야 한다.
	- 구현한 타깃 클래스는 다시 프록시 클래스에 의해 구현된다.

## CGLIB Proxy

CGLIB 프록시는 다음의 특징을 가지고 있다.
- 서브 클래스를 통해 프록시 객체를 만든다.
- 이 때 프록시는 타깃 클래스의 서브클래스가 된다.
	- 때문에 인터페이스를 사용하지 않아도 프록시를 사용할 수 있다.

## 둘의 차이점은?

CGLIB의 서브 클래스 생성 과정에서는
- CGLIB이 슈퍼 클래스에 대해 충분히 알아야 한다는 것이 전제된다. 
	- 올바른 파라미터로 올바른 생성자를 호출할 수 있어야 한다.
- 이는 CGLIB의 프록시가 JDK 프록시에 비해 조금 더 신경써야 할 것이 많다는 것을 의미한다.
	- CGLIB은 final 메서드들을 무시한다. (Exception도 던지지 않는다)

- JDK Dynamic 프록시는 항상 호출 당 추가 스택 프레임을 필요로 한다. (내부적으로 리플렉션 API를 쓰기 때문이다.)
- 하지만 CGLIB은 추가 스택 프레임을 필요로 할 수도 있고, 아닐 수도 있다.

어플리케이션이 복잡해지면 이 차이는 유의미하게 커진다고 한다.
- 스택 크기가 커질 수록, 쓰레드가 소모하는 메모리가 커지기 때문이다.
- CGLIB 프록시가 따라서 메모리 소모량에서 좀 더 효율적이다.

## 언제 실행될까?

- `public class Foo implements iFoo` 는 JDK Dynamic Proxy가,
- `public class Foo`는 CGLIB이 프록시할 수 있다.

### Spring Data JPA

Spring Data JPA는 인터페이스로 선언된 Repository만으로도 동작한다.

그리고 방금 알아봤던 것처럼, 인터페이스로 선언된 녀석들은 JDK Dynamic Proxy가 맡아서 처리한다.
- 내부적으로는 `InvocationHandler`의 구현체를 호출해서 프록시가 생성된다.

### Spring Boot에서

https://www.springcloud.io/post/2022-01/springboot-aop/#gsc.tab=0

Spring Boot 2.0 과 그 전으로 기본 프록시 정책 (별도의 프록시 정책을 설정하지 않았을 때)이 다르다.
- Spring Boot 2.0 이전에는: 
	- `spring.aop.proxy-target-class` 가 false : JDK Dynamic 프록시 사용
	- `spring.aop.proxy-target-class`가 true: CGLIB 프록시 사용
	- `spring.aop.proxy-target-class`를 설정하지 않았을 경우: JDK Dynamic 프록시 사용
- Spring Boot 2.0 + 이후에는 : 
	- 나머지 설정은 동일하지만, 별도의 설정이 없을 경우 CGLIB 프록시가 기본적으로 사용된다.
	- 그런데 아까 말했듯이 Spring Data JPA의 경우 인터페이스를 기반으로 하기 때문에, JDK Dynamic Proxy가 사용된다. 
