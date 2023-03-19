---
title: "Caching in Spring"
date: 2023-03-19T22:09:32+09:00
draft: false
author: redjen
---

스프링 프레임워크에서의 캐시 추상화는 어떤 방식으로 동작할까?

https://www.baeldung.com/spring-cache-tutorial

## 어노테이션 기반 캐싱

### @Cacheable

캐싱을 활성화하기 위한 방법으로는 `@Cacheable` 어노테이션으로 감싸는 것이다.
- 이 때 결과가 저장될 캐시의 이름을 파라미터화한다.
- `@Cacheable` 어노테이션이 붙은 메서드는 실제로 실행되기 전에 캐시 주소를 먼저 체크한다.
- 대부분의 경우 하나의 캐시로 충분하지만, 여러 개의 캐시를 사용하도록 설정할 수도 있다.
  - `@Cacheable({"addresses", "directory"})`
  - 이 경우 `addresses`, `directory` 캐시 중 하나라도 값이 있다면 결과가 리턴되고 메서드는 호출되지 않는다.

### @CacheEvict

모든 메서드를 `@Cacheable`로 만들면 어떤 문제가 생길까?

문제는 크기이다. 우리는 자주 사용하짖 않는 값에 대해서 캐시로 사용하고 싶지 않다. 캐시의 크기는 엄청 빠르게 성장하며, 그렇게 된다면 이미 값이 유효하지 않거나 이미 사용하지 않는 데이터를 저장할 수도 있다.

이 때 `@CacheEvict` 어노테이션을 사용해서 하나 이상의 값을 삭제하여 최신의 값이 다시 캐시로 들어가게 설정할 수 있다.

```java
@CacheEvict(value="addresses", allEntries=true)
public String getAddress(Customer customer) {...}
```

`allEntries` 파라미터를 같이 사용한다면 어떤 캐시가 초기화될지를 지정할 수 있다. 이 파라미터는 캐시 주소들에 들어 있는 모든 엔트리를 삭제하고 새 데이터를 준비한다.

### @CachePut

`@CacheEvict`는 예전 값이나 사용하지 않는 엔트리를 삭제함으로써 큰 크기의 캐시에 있는 엔트리를 탐색하는 오버헤드를 줄일 수 있다.
    그런데 너무 많은 데이터가 캐시에서 제거되면 어쩌지? 이 경우엔 캐시를 쓰는 의미가 사실상 퇴색된다.

대신 우리는 선택적으로 값을 수정했을 때에만 캐시 엔트리를 선택적으로 업데이트하는 방법을 사용할 수 있다.

`@CachePut` 어노테이션을 통해서 메서드 실행을 방해하지 않고 캐시의 내용을 업데이트 할 수 있다.
```java
@CachePut(value="addresses")
public String getAddress(Customer customer) {...}
```

상기 메서드는 항상 실행되고, 그리고 그 결과가 캐시된다.

> `@Cacheable`과 `@CachePut`의 차이점은 `@Cacheable`은 메서드 실행을 스킵할 수 있다는 점이고, 반면 `@CachePut`은 실제로 메서드를 실행하고 그 결과를 캐시에 저장한다는 점이다.

### @Caching

메서드를 캐싱할 때 동일한 타입의 어노테이션을 많이 사용해야 한다면, 자바는 동일 타입의 다수 어노테이션을 허용하지 않기 때문에 컴파일 오류가 발생한다.

이 때 `@Caching`을 사용한다.

```java
@Caching(evict = { 
  @CacheEvict("addresses"), 
  @CacheEvict(value="directory", key="#customer.name") })
public String getAddress(Customer customer) {...}
```

## 선택적 캐싱 전략

전체 데이터를 매번 캐싱하는 것은 때때로 비효율적일 수 있다.

캐시하기 위한 조건을 걸고 싶다면 어떻게 설정해야 할까?

### condition 파라미터

어노테이션이 활성화되어 있을 때, `@CachePut` 을 파라미터화 해서 SpEL 표현식을 넣어 결과들이 해당 표현식에 기반하여 캐싱될 수 있도록 설정할 수 있다.

```java
@CachePut(value="addresses", condition="#customer.name=='Tom'")
public String getAddress(Customer customer) {...}
```

### Unless 파라미터

캐싱을 입력이 아닌 결과 값에 기반하여 설정할 수도 있다.
```java
@CachePut(value="addresses", unless="#result.length()<64")
public String getAddress(Customer customer) {...}
```

위 코드는 메서드 결과의 길이가 64자 보다 짧지 않다면 캐시에 저장한다.
condition과 unless 파라미터는 `@CachePut` 어노테이션 이외의 다른 캐싱 어노테이션과도 같이 사용될 수 있다.

## 리액티브 환경에서의 캐싱

스프링 웹플럭스에서 캐싱을 사용하기 위해서 그대로 `@Cacheable`을 사용해서는 안된다.

스프링 컨텍스트에서는 `@Cacheable`을 사용했을 떄 결과 객체를 캐싱하게 되는데, `Mono<T>`나 `Flux<T>`의 경우 실제로 캐시하게 되는 것이 `T` 타입 객체가 아닌 `Publisher<T>` 타입이기 때문이다.
- 그리고 `Publisher<T>` 타입은 실제 subscribe가 일어나기 전까지는 단순한 파이프라인의 조립이기 떄문에
- 캐싱된 `Publisher` 객체는 캐시하고자 하는 값을 담고 있지 않다.

스프링 프레임워크에서 리액티브한 타입에 대한 `@Cacheable` 메서드 지원은 공식적으로 이루어지지 않는 것 같다.

https://github.com/spring-projects/spring-framework/issues/17920

웹플럭스 환경에서 어노테이션 기반 캐싱 전략을 사용하려면, 직접 AOP로 커스텀하거나 외부 라이브러리를 사용해야 한다.