---
title: "Java의 Threadlocal"
date: 2023-01-27T18:42:46+09:00
draft: false
author: redjen
---

Java의 `ThreadLocal`에 대해 알아보았다.
https://www.baeldung.com/java-threadlocal

## ThreadLocal API란?

`ThreadLocal`은 특정 쓰레드에서만 접근할 수 있는 데이터를 저장할 수 있도록 도와준다.
아래와 같이 인스턴스화 할 수 있다.
```java
ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
```

### 값 저장
`threadLocal.set(someValue)` 으로 값을 저장한다.

### 값 획득
`threadLocal.get()`으로 저장된 값을 가져온다.

### 값 제거
`threadLocal.remove()`로 저장된 값을 제거한다.

## 특징

일반 변수는 scope 안에서만 수명을 가지고, scope가 종료되는 순간 gc의 대상이 된다.
`ThreadLocal`을 사용하면 특정 쓰레드에 값을 저장할 수 있기 때문에 특정 쓰레드가 실행하는 모든 작업에서 저장한 값을 전역처럼 사용할 수 있다.

주의해야 할 점은, 일반적인 웹 어플리케이션 (스프링 부트에 내장된 톰캣 등)에서는 쓰레드 풀을 사용한다.
쓰레드 풀과 함께 `ThreadLocal`을 사용할 때에는 저장한 데이터의 사용을 마치고 `remove()`를 통해 제거해줘야 한다.
쓰레드 재사용 시 유효하지 않은 데이터를 참조하여 의도치 않은 동작을 할 수 있기 때문이다.

## Spring에서의 사용

서블릿 기반 스프링 웹 어플리케이션에서는 일반적으로 사용자의 요청 하나 당 쓰레드 풀에서 유휴 상태인 쓰레드 하나를 할당하여 요청을 처리한다.

Spring Security에서 제공하는 `SecurityContextHolder`는 `SecurityContext`를 제어한다.
그리고 기본적으로 이 떄 `ThreadLocal`을 사용한다. 이 때 `ThreadLocal`을 사용함으로써 얻는 이점으로는
- 한 쓰레드 내에서 공용으로, 전역으로 저장하기 때문에 데이터에 접근이 가능한 `ThreadLocal`의 특성 이용
- `Authentication`을 한 쓰레드 내에서 파라미터의 전달 없이 공유할 수 있다

찾아보니 여러 strategy 중 선택하여 `SecurityContext`를 저장할 수 있는 듯 하다.

https://stackoverflow.com/questions/45725888/why-does-spring-security-store-securitycontext-in-thread-local-variable

1. ThreadLocalSecurityContextHolderStrategy : `ThreadLocal`에 `SecurityContext`를 저장한다. (앞서 설명했던 방법)
2. InheritableThreadLocalSecurityContextHolderStrategy : `InheritableThreadLocal`에 `SecurityContext`를 저장한다.
3. GlobalSecurityContextHolderStrategy : `SecurityContext`를 static하게 저장하고 관리한다. 서버의 용도로는 부적격.

기본적으로는 `ThreadLocal`을 사용하고, 특정 경우에 `InheritableThreadLocal`에 저장한다. 

둘의 차이는 무엇일까? 다음에 알아보는 것으로~

[SecurityContextHolder 공식 문서](https://docs.spring.io/spring-security/site/docs/3.0.x/apidocs/org/springframework/security/core/context/SecurityContextHolder.html)