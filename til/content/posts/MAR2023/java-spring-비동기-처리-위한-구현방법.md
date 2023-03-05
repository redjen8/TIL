---
title: "Java, Spring의 비동기 처리 위한 구현방법"
date: 2023-03-05T19:29:30+09:00
draft: false
author: redjen
---

자바의 여러 비동기 처리를 위한 구현체들과, 그 차이점을 알아보자.

## Future

https://stackoverflow.com/questions/38744943/listenablefuture-vs-completablefuture

자바의 `Future` 클래스는 JDK6부터 도압되었으며, 호출자에게 비동기 액션이 완료되면 실행할 콜백 함수를 동작할 수 있도록 허용한다.

### ListenableFuture

`ListenableFuture`는 JDK6 `Future` 클래스로부터 파생된 Google Guava 라이브러리의 한 기능이다. 

https://medium.com/pramod-biligiris-blog/how-is-google-guava-listenablefuture-better-than-java-future-f27a31a86192

`ListenableFuture`는 자바의 `Future`과 비교해서 몇 가지 이점을 가지고 있는데,
1. 명시적으로 `Future.get()`을 호출해서 결과에 대한 예외 처리나 핸들링을 하는 것이 아니라 `Future`의 성공 또는 실패 결과에 대한 리스너들을 추가할 수 있다.
2. 동일한 코드를 반복적으로 작성하는 대신 제공된 유틸리티 함수들을 사용해서 여러 비동기적인 코드의 조각들을 연결할 수 있다.

`ListenableFuture` 클래스를 사용한다면 콜백 함수를 아래와 같은 방법으로 등록할 수 있다.

```java
ListenableFuture listenable = service.submit(...);
    Futures.addCallback(listenable, new FutureCallback<Object>() {
        @Override
        public void onSuccess(Object o) {
            //handle on success
        }

        @Override
        public void onFailure(Throwable throwable) {
            //handle on failure
        }
    })
```

### CompletableFuture

Java 진영에서는 JDK8 버전부터 `CompletableFuture`의 도입으로 비동기처리를 표준 API로 가져오게 하였다.

`CompletableFuture` 클래스는 `CompletionStage` 인터페이스와 `Future` 인터페이스를 결합한 방식으로 도입되게 되었다.

`CompletionStage`는 다음과 같은 특징을 가진다.
- 동기 / 비동기 로직을 동시에 사용할 수 있도록 지원한다.
- 때문에 인터페이스가 지원하는 메서드들이 엄청 많아지는 원인이 되었다. (`thenAccept`, `thenAcceptAsync`, `thenApply`, `thenApplyAsync`...)

`CompletableFuture`는 `CompletionStage` 인터페이스가 가지고 있던 불편함을 해소하기 위해 특정 기능들을 비활성화한 서브클래스를 생성해서 현재에는 JDK 자체의 유일한 `CompletionStage`의 서브 클래스가 되었다.

`CompletableFuture` 클래스를 사용한다면 콜백 함수를 아래와 같은 방법으로 등록할 수 있다.

```java
CompletableFuture completableFuture = new CompletableFuture();
    completableFuture.whenComplete(new BiConsumer() {
        @Override
        public void accept(Object o, Object o2) {
            //handle complete
        }
    }); // complete the task
    completableFuture.complete(new Object())
```

`CompletableFuture` 클래스는 태스크가 완료되었을 떄 실행될 콜백을 등록하는 것은 동일하지만, `ListenableFuture`와는 구별되는 차이점을 가지고 있다.

> `CompletableFuture`는 완료되기를 원하는 어떤 쓰레드에서도 작업이 완료되는 것을 알 수 있다.

https://nurkiewicz.com/2013/05/java-8-definitive-guide-to.html

## DeferredResult

스프링에서는 `DeferredResult` 클래스를 제공한다. 

https://www.baeldung.com/spring-deferred-result

그리고 이 `DeferredResult` 클래스는 단순히 '결과에 대한 컨테이너'에 불과하기 때문에 요청에 대한 처리를 비동기적으로 실행하기 위해서 특정 종류의 쓰레드 풀(`ForkJoinPool` 등)을 명시적으로 사용해야 한다는 특징이 있다.

서블릿 기반 스프링 어플리케이션에서 `@Controller` 단에서 `ResponseEntity<?>` 를 반환하는 단순한 API는 요청이 완전히 처리될 때까지 요청을 처리하는 쓰레드는 블락된다.

동일한 서블릿 기반 스프링 어플리케이션에서 `DeferredResult<T>`를 쓰게 된다면
- `@Controller`가 물고 있는 쓰레드 이외의 별개의 분리된 쓰레드를 사용한다.
- 작업이 완료되면 `DeferredResult`의 `setResult` 연산을 수행한다.
- `@Controller`가 물고 있는 쓰레드가 noti 되고, HTTP 응답이 클라이언트에게 전달된다.

### DeferredResult의 콜백 

`DeferredResult`에는 3가지 종류의 콜백이 존재한다.

1. `onCompletion()` : 비동기 요청이 완료되었을 때 실행될 콜백
2. `onTimeout()` : 요청 중 타임아웃이 발생했을 때 실행될 콜백. 요청 처리 시간을 제한하기 위해서 `DeferredResult` 객체 생성 시에 타임아웃 값을 주입할 수 있다.
3. `onError()`: 요청 중 에러가 발생했을 떄 실행될 콜백. 발생한 에러에 따라서 서로 다른 응답 코드와 메시지 본문을 에러 핸들러를 통해 설정할 수 있다.