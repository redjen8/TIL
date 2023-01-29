---
title: "Spring Security 개요"
date: 2023-01-29T16:28:08+09:00
draft: false
author: redjen
---

리액티브 기반 스프링 웹 어플리케이션에서 스프링 시큐리티는 어떻게 동작할까?

그런데 서블릿 기반의 스프링 시큐리티도 잘 모르기 때문에, 먼저 서블릿 기반의 스프링 웹 어플리케이션에서 Spring Security는 어떻게 동작하는지 먼저 알아보고 비교해보기로 했다.

https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html

## SecurityContextHolder

[ThreadLocal](https://redjen8.github.io/posts/jan2023/java%EC%9D%98-threadlocal/)을 공부할 때 곁다리로 잠깐 나왔던 내용이다. 

> `SecurityContextHolder`는 이름처럼 `SecurityContext`를 담는 역할을 한다.

즉, 스프링은 어떤 사람이 인증되었는지에 대한 디테일한 정보들을 `SecurityContextHolder`에 담는다. 하지만 스프링 시큐리티는 `SecurityContextHolder`가 채워지는 방식에 대해서는 상관하지 않는다. 그저 값이 있다면 해당 값을 현재 인증한 사용자로 사용한다.

`SecurityContextHolder`를 직접적으로 설정해서 사용자를 가르킬 수 있다.
```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);
```
`SecurityContext`는 `Authentication` 객체를 가지고, 다시 `Authentication` 객체는 사용자의 인증 정보를 담는다. (`Principal` 등이 `Authentication`에 속한다)

`SecurityContextHolder`는 기본적으로 `ThreadLocal`을 사용해서 사용자의 정보를 저장한다.
이는 곧 `SecurityContext`는 기본적으로 동일한 쓰레드 안에 있는 메서드들이 항상 접근 가능한 상태임을 말한다. (`SecurityContext`가 함수 파라미터로 전달되지 않아도 같아도 됨)

`ThreadLocal`을 사용할 때에는 값을 사용한 다음 값을 비워주는 것에 유의해야 하는데, 스프링 시큐리티의 `FilterChainProxy`가 현재 principal의 요청이 종료되면 항상 `SecurityContext`가 비워진 상태임을 보장해준다.

그런데 어떤 어플리케이션은 쓰레드를 직접 요리조리 다루기 때문에 `ThreadLocal`을 사용하기 부적합 경우도 있다.
- Swing 클라이언트의 경우 동일한 Security context를 사용하기 위해 JVM에 있는 모든 쓰레드를 요구할 수 있다.

이런 경우, `SecurityContextHolder`의 strategy를 수정해서 context가 어떻게 저장될 지 커스텀할 수 있다.
- 스탠드 얼론 어플리케이션의 경우 `SecurityContextHolder.MODE_GLOBAL` strategy를 사용할 수 있다.
- 인증이 완료된 쓰레드에서 spawn된 다른 쓰레드들이 동일한 security context를 가지길 원한다면 `SecurityContextHolder.MODE_INHERITABLETHREADLOCAL`을 사용하면 된다.

## SecurityContext

`SecurityContext`는 `SecurityContextHolder`가 가지는, `Authentication` 객체에 대한 간교 역할을 수행한다.

## Authentication

`Authentication` 인터페이스는 스프링 시큐리티의 두 가지 주요한 목적을 달성시킨다.
1. `AuthenticationManager`의 입력으로써, 사용자가 인증 받기 위해 제공한 credential을 제공한다. 이 경우 `isAuthenticated()`는 false가 된다.
2. 현재 인증된 사용자를 대표하기 위해서 사용된다. 현재 인증 정보는 `SecurityContext`로부터 얻을 수 있다.

`Authentication`은 3가지 정보를 포함한다.
1. `principal`: 사용자를 식별한다. username / password로 인증할 경우 `principal`은 보통 `UserDetails`의 인스턴스이다.
2. `credentials`: 보통 비밀번호에 대한 내용이다. 많은 경우 사용자가 인증을 마친 후에 초기화되어 누출되지 않도록 한다.
3. `authorities`: `GrantedAuthority` 인스턴스은 인가 받은 사용자에 대한 고수준 권한이다. 역할과 스코프를 제공한다.

### GrantedAuthority

`Authentication.getAuthorities()` 메서드를 통해 `GrantedAuthority`를 얻을 수 있다.
해당 메서드는 `GrantedAuthority` 객체의 컬렉션을 제공한다.

`GrantedAuthority` 객체는 principal에 승인된 권한을 나타낸다. 어떤 권한들은 보통 `ROLE_ADMINISTRATOR`나 `ROLE_HR_SUPERVISOR`처럼 역할인 경우가 있다.
이 역할들은 나중에 웹 인증, 메서드 인증, 도메인 객체 인증을 위해 설정될 수 있다.

스프링 시큐리티의 다른 부분은 이 권한을 해석하고 존재하는지를 검사한다. username/password 기반한 인증을 사용할 때에는 `UserDetailsService`에 의해 `GrantedAuthority`가 불려진다.

보통 `GrantedAuthority`는 어플리케이션 전역에 대한 권한 객체다. 어떤 도메인 객체에 특정되지 않기 때문에 `Employee` 54번 객체에 대한 권한을 특정할 수는 없다. 해당 권한은 사용자를 인증하는데 아주 오랜 시간이 걸리더라도 수백, 수천개의 권한 객체가 메모리 안에서 굉장히 빠르게 사라지기 때문이다. 

물론 스프링 시큐리티를 사용하면 이 공통 요구 사항을 잘 핸들링할 수 있지만 특정 도메인의 권한을 핸들링하기 위해서는 프로젝트 도메인 종속 객체 보안을 별도로 구현해야 한다.

## AuthenticationManager

`AuthenticationManager`는 스프링 시큐리티의 가 어떻게 인증을 수행할지에 대한 `Filter`를 정의한다.
그 다음 리턴되는 `Authentication`은  `AuthenticationManager`를 호출한 컨트롤러 (스프링 시큐리티의 Filter 인스턴스)에 의해  `SecurityContextHolder`에 설정된다. 

만약 스프링 시큐리티의 `Filter` 인스턴스를 사용하지 않는다면 `SecurityContextHolder`를 직접 설정해줘야 하는 대신 `AuthenticationManager`를 사용하지 않아도 된다.

`AuthenticationManager`의 구현체는 여러 가지가 있지만, 가장 흔하게 사용되는 것은 `ProviderManager`이다.

### ProviderManager

`ProviderManager`는 가장 흔하게 사용되는 `AuthenticationManager`의 구현체이다. `ProviderManager`는 `AuthenticationManager` 인스턴스들의 리스트를 위임한다.

각각의 `AuthenticationProvider`는 해당 인증이
- 성공하였는지
- 실패하였는지
- 또는 결정할 수 없어서 `AuthenticationProvider`에게 결정을 부탁할지를
 정할 수 있다. 

만약 설정된 `AuthenticationProvider` 인스턴스를 통해서 인증할 수 없다면 `ProviderNotFoundException`과 함께 인증이 실패된다.
`ProviderNotFoundException`은 `AuthenticationException`의 일종으로, `ProviderManager`가 전달 받은 해당 타입의 인증을 지원하도록 설정되지 않았음을 의미한다.

각각의 `AuthenticationProvider`는 특정 타입의 인증을 어떻게 수행할지를 알고 있다.
예를 들어
- 1번 `AuthenticationProvider`는 username/password 형태의 인증을 검증할 수 있다.
- 2번 `AuthenticationProvider`는 SAML 인증을 검증할 수 있다.

이런 형태의 구성은 몇 가지 이점이 있다.
- 각각의 `AuthenticationProvider`가 아주 구체적인 인증을 담당할 수 있다.
- 다양한 종류의 인증을 지원할 수 있다.
- `AuthenticationManager` bean 하나만 노출시킬 수 있다.

`ProviderManager`는 또한 옵션으로 부모인 `AuthenticationManager`를 설정할 수 있다. 좀 전에 설명했던 '인증을 수행할 수 있는 적당한 `AuthenticationProvider`가 존재하지 않을 경우'에 해당된다.
이 때 설정하는 부모는 어떤 타입의 `AuthenticationManager`가 될 수 있지만, 보통은 `ProviderManager`의 인스턴스이다.

사실 다수의 `ProviderManager` 인스턴스들은 동일한 부모의 `AuthenticationManager`를 공유할 수 있다.
- 공통 인증 체계를 가지고 있지만 서로 다른 인증 메커니즘을 가지고 있는 `SecurityFilterChain` 인스턴스를 사용하는 시나리오에서는 흔히 사용하는 방식이다.

기본적으로 `ProviderManager`는 인증 요청이 성공적으로 리턴되면 `Authentication` 객체에 있는 모든 민감한 인증 데이터를 초기화하려 시도한다. 이 행동 패턴은 비밀번호와 같은 정보가 `HttpSession`에 오래 남아 있는 일을 막는다.

그런데 이 초기화하는 행위가 사용자 객체에 대한 캐시를 사용할 때에는 문제가 될 수 있다.

예를 들어 stateless한 어플리케이션의 성능을 향상시키는 시나리오를 생각해보자.
- 만약 `Authentication`가 캐시에 있는 객체에 대한 레퍼런스를 가지고 있고,
- 이 credential이 제거된다면
- 캐시된 값에 대한 인증이 불가능해진다.

그렇기 때문에 캐시를 사용하는 경우 이 점을 고려해야 한다. 해결 방법으로는 캐시의 구현체 또는 반환된 `Authentication` 객체를 생성하는 `AuthenticationProvider`에서 해당 객체의 복사본을 만드는 것이다. 다른 방법으로는 `ProviderManager`의 속성 중 `eraseCredentialsAfterAuthentication`을 비활성화하는 방법이 있다.

## AuthenticationProvider

`ProviderManager` 안으로 여러 개의 `AuthenticationProvider` 인스턴스를 주입할 수 있다.
각각의 `AuthenticationProvider`들은 특정 타입의 인증을 수행한다.
- `DaoAuthenticationProvider`는 username/password 기반 인증을 수행한다.
- `JwtAuthenticationProvider`는 JWT 토큰을 사용한 인증을 지원한다.


## AuthenticationEntryPoint로 credential 요청

`AuthenticationEntryPoint`는 클라이언트로부터 credential들에 대한 HTTP 응답을 보낼 때 사용된다.

가끔은 클라이언트가 리소스를 요청하기 위해 사전에 username/password와 같은 credential을 포함한다. 이런 경우 스프링 시큐리티는 인증 정보가 이미 사전에 포함되어 있기 때문에 클라이언트로부터 credential을 요청하는 HTTP 응답을 제공할 필요가 없다.

다른 경우 클라이언트가 접근할 수 없는 리소스를 요청하는 인증되지 않은 요청을 보낼 수 있다. 이런 경우에는 `AuthenticationEntryPoint`의 구현이 클라이언트에게 credential을 요청하기 위해서 사용될 수 있다. `AuthenticationEntryPoint` 구현체는 로그인 페이지로 리다이렉트하는 동작을 수행하거나, WWW-Authenticate 헤더로 응답하거나 다른 액션을 취할 수 있다.

## AbstractAuthenticationProcessingFilter

`AbstractAuthenticationProcessingFilter`는 사용자의 credential을 인증하기 위한 기본 `Filter`로 사용된다. credential들이 인증되지 전에는 스프링 시큐리티는 보통 credential들을 `AuthenticationEntryPoint`를 통해 요청한다.

그 다음에는 `AbstractAuthenticationProcessingFilter`가 제출된 모든 인증 요청을 핸들링할 수 있다.

### 스프링 시큐리티의 인증 절차 요약

1. 사용자가 credential을 제출할 때, `AbstractAuthenticationProcessingFilter`가 인증 대상이 되는 `HttpServletRequest`로부터 `Authentication`을 생성한다. `Authentication`의 타입은 `AbstractAuthenticationProcessingFilter`의 서브 클래스에 의존한다.

예를 들어, `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`안에 포함된 username과 password를 통해 `UsernamePasswordAuthenticationToken`을 생성한다.

2. 인증 대상이 되는 `Authentication` 객체가 `AuthenticationManager`로 전달된다.
3. 인증이 실패하는 경우.
   1. `SecurityContextHolder`가 초기화 된다.
   2. `RememberMeServices.loginFail`이 호출된다. 만약 remember me가 설정되지 않았다면 이 작업은 수행되지 않는다. (no-op)
   3. `AuthenticationFailureHandler`가 호출된다.
4. 인증이 성공하는 경우.
   1. `SessionAuthenticationStrategy`가 새 로그인에 대한 알림을 받는다.
   2. `Authentication`이 이제는 `SecurityContextHolder`에 담겨 있다. 나중에는 `SecurityContextPersistenceFilter`가 `SecurityContext`를 `HttpSession`에 저장한다.
   3. `RememberMeServices.loginSuccess`가 호출된다. 만약 remember me가 설정되지 않았다면 이 작업은 수행되지 않는다. (no-op)
   4. `ApplicationEventPublisher`가 `InteractiveAuthenticationSuccessEvent`를 발행한다.
   5. `AuthenticationSuccessHandler`가 호출된다.

