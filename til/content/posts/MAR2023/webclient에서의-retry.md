---
title: "Webclient에서의 Retry"
date: 2023-03-14T21:09:22+09:00
draft: false
author: redjen
---

webclient를 사용하면 다른 서버로의 웹 요청에 대한 응답을 비동기적으로 수행할 수 있다.

그런데, 요청이 실패하면 어떻게 될까? 실패했을 때의 요청에 대해 재시도 정책을 구성하려면 어떻게 해야 할까?

https://www.baeldung.com/spring-webflux-retry

## retry

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
        .uri(PATH_BY_ID, stockId)
        .retrieve()
        .bodyToMono(String.class)
        .retry(3);
}
```
사익와 같이 시도할 횟수를 정책으로 추가하여 실패한 요청에 대한 재시도를 시도하도록 구성할 수 있다.

이 때 주의해야 할 점은 WebClient는 리턴받는 에러의 종류와는 관계 없이 재시도에 진입한다는 점이다.

## retryWhen

`retry()`는 너무 딱딱하다. 최대 시도에 대해 더 많은 옵션을 구성하기 위해서는 어떻게 해야할까?

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
        .uri(PATH_BY_ID, stockId)
        .retrieve()
        .bodyToMono(String.class)
        .retryWhen(Retry.max(3));
}
```

`retryWhen()`을 사용하면 각각의 실패에 대한 재시도는 eager하게 발생한다는 점에 유의해야 한다.

## 딜레이를 추가하는 방법

하지만 이런 요청에 대한 실패는 서버 어플리케이션 자체의 문제일 수도 있지만 네트워크 레벨에서의 장애일 수도 있다.

학부 네트워크 시간에 배웠던 것처럼 TCP는 동일 네트워크 내에서의 요청 횟수를 조절할 수 있던 기능이 포함되어 있는 프로토콜이다.

[TCP의 혼잡 제어](https://evan-moon.github.io/2019/11/26/tcp-congestion-control/) 과는 관계 없이 어플리케이션에서 어플리케이션 간의 통신에서 재시도 정책을 구성하되, 재시도로 인해 네트워크를 더 혼잡하게 만드는 것을 방지하지 위해서 WebClient 설정에도 여러 구성을 끼워 넣을 수 있다.

### fixedDelay

`fixedDelay()`는 아주 단순하다.
각각의 재시도 사이에 고정된 딜레이 값을 삽입하여 그 동안은 재시도를 시도하지 않는 정책이다.

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(2)));
}
```

### backOff

`backOff()`는 `fixedDelay()`보다 더 최악의 상황을 가정한다.

특정 딜레이 경과 뒤에도 네트워크나 어플리케이션의 상태가 좋아지지 않는다면, exponential하게 딜레이를 구성해서 더 많은 재시도가 실패했다는 점은 곧 그만큼 서버나 네트워크 상태가 좋지 않다는 기본적인 생각을 반영한다.

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(2)));
}
```

상기의 코드는 2초, 4초, 8초의 딜레이를 가지고 다음 요청을 재시도하게 한다. 

### jitter

`jitter()`를 활용하면 가장 보수적인 정책을 설정할 수 있다.

`jitter()`는 클라이언트 간 충돌을 피하면서도 exponential 하게 delay를 구성한다.  jitter 구성에 대해 자세히 설명된 글은 [이 곳](https://www.baeldung.com/resilience4j-backoff-jitter#jitter) 에 잘 정리되어 있었다.

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .accept(MediaType.APPLICATION_JSON)
      .retrieve()
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(2)).jitter(0.75));
}
```

## 특정 에러에만 재시도 구성하기

`retryWhen()`에 `filter()`를 걸어서 발생하는 `throwable`에 대해 모두 `Exception`을 던지는 것이 아니라 필터링하고 싶은 특정 예외 상황에서만 `Exception`을 던지도록 구성할 수 있다.

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .onStatus(HttpStatus::is5xxServerError, 
          response -> Mono.error(new ServiceException("Server error", response.rawStatusCode())))
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(5))
          .filter(throwable -> throwable instanceof ServiceException));
}
```

## 만약 모든 재시도에도 실패하는 경우에는

이렇게 노력했음에도 불구하고 끝끝내 요청이 실패할 수 있다. 절망적이다.

하지만 그럼에도 불구하고 우리는 모든 재시도가 실패했다는 것을 인지시켜야 한다. 이럴 때 사용하는 것이 `onRetryExhaustedThrow()` 이다.

```java
public Mono<String> getData(String stockId) {
    return webClient.get()
      .uri(PATH_BY_ID, stockId)
      .retrieve()
      .onStatus(HttpStatus::is5xxServerError, response -> Mono.error(new ServiceException("Server error", response.rawStatusCode())))
      .bodyToMono(String.class)
      .retryWhen(Retry.backoff(3, Duration.ofSeconds(5))
          .filter(throwable -> throwable instanceof ServiceException)
          .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
              throw new ServiceException("External Service failed to process after max retries", HttpStatus.SERVICE_UNAVAILABLE.value());
          }));
}
```

이러한 정책을 커스텀하게 구성하면 서비스의 사양과 디테일에 맞춰서 보다 세세한 MSA 구성이 가능하다.