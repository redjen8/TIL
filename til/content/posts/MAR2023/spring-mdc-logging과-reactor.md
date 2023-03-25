---
title: "Spring Mdc Logging과 Reactor"
date: 2023-03-25T22:23:54+09:00
draft: false
author: redjen
---

자바 웹 어플리케이션에서는 Mapped Diagnostic Context (MDC)를 사용하여 어플리케이션 로깅을 지원한다.

MDC는 로깅이 실제로 일어나는 스코프에서 접근 불가능한 정보들을 담아 로그 메시지를 더 의미 있고 유용하게 만든다.
로깅의 질적 향상은 프로그램 실행을 추적하기 더 용이하게 해준다.

https://www.baeldung.com/mdc-in-log4j-2-logback

## MDC를 사용하는 이유

금융, 결제와 관련된 소프트웨어를 작성한다고 가정해보자.
내가 작성한 어플리케이션에 대해서는 `@Slf4j`와 같은 어노테이션을 통해 로그 기록에 대한 AOP를 쉽게 적용시킬 수 있다.

그런데, 내부의 어플리케이션이 아닌 외부 모듈과 통신한 결과를 로깅하고 싶을 때에는 어떻게 할까?

실제로 이런 경우에는 로그가 우리가 생각했던 것 만큼 유용하지 않은 경우가 빈번하다.

각각의 전송이 실행되는 것에 대해 트래킹하는 것은 어렵다.
1. 송수신되는 정보의 전체가 아닌 일부분만 로깅하고 싶다면?
2. 동일한 쓰레드에서 동일한 양의 송금되었는데, 이 트랜잭션의 ID를 얻고 싶다면?

MDC를 사용해야 한다.

## Log4j에서의 MDC

Log4j의 MDC는 Map과 같은 자료 구조를 로그 메시지가 실제로 기록되는 appender가 쓸 수 있는 정보의 조각으로 채우도록 한다.

MDC 구조는 `ThreadLocal`이 그런 것처럼 내부적으로 실행 중인 쓰레드와 같이 동작한다. 벌어지는 일의 순서는 아래와 같다.
1. MDC를 appender에게 쓸 수 있도록 허용한 정보로 채운다.
2. 메시지를 로깅한다.
3. MDC를 초기화한다.

## MDC와 쓰레드 풀

MDC 구현은 전형적으로 `ThreadLocal`을 사용해서 컨텍스트의 정보를 저장한다. 쉽고 합리적인 방법으로 쓰레드 안정성을 보장할 수 있기 때문이다. 

하지만 MDC를 쓰레드 풀과 함께 다룰 때에는 조심해서 사용해야 한다.

`ThreadLocal` 기반한 MDC와 쓰레드 풀의 조합이 어떻게 위험할 수 있는지 단계별로 살펴보면:
1. 쓰레드 풀에서 쓰레드를 가져온다.
2. `MDC.put()`이나 `ThreadContext.put()`을 사용하여 MDC에 컨텍스트 정보를 저장한다.
3. 해당 정보를 사용하여 로깅하고, 어쩌다 보니 MDC 컨텍스트를 초기화하는 것을 잊었다고 가정하자.
4. 빌려온 쓰레드를 쓰레드 풀에 반납한다.
5. 잠시 후에 어플리케이션이 쓰레드 풀에서 동일한 쓰레드를 가져온다.
6. 지난 번에 MDC를 잘 초기화하지 않았기 때문에, 해당 쓰레드는 이전 실행에서의 일부 데이터를 가지고 있다.

이러한 동작은 실행 간 예상하지 못한 불일치성을 유발시킬 수 있다.. 어떻게 예방할까?

1. 매 실행의 끝에 MDC 컨텍스트를 초기화하는 것을 항상 잊어버리지 않는 것
   1. 개발자가 매번 신경써야 하기 때문에 결국에는 에러에 취약해진다.
2. `ThreadPoolExecutor` 훅을 사용해서 매 실행 후에 필수적으로 실행되어야 할 cleanup 작업들을 정의한다.

```java
public class MdcAwareThreadPoolExecutor extends ThreadPoolExecutor {

    public MdcAwareThreadPoolExecutor(int corePoolSize, 
      int maximumPoolSize, 
      long keepAliveTime, 
      TimeUnit unit, 
      BlockingQueue<Runnable> workQueue, 
      ThreadFactory threadFactory, 
      RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        System.out.println("Cleaning the MDC context");
        MDC.clear();
        org.apache.log4j.MDC.clear();
        ThreadContext.clearAll();
    }
}
```

## 리액티브 환경에서의 MDC

https://medium.com/@grigorryev/project-reactor-mdc-logging-1047d235ff6e

요청 당 쓰레드가 1대1로 매핑되는 패러다임에서는 MDC 로깅이 꽤 아름답게 수행될 수 있다. 
- 서블릿 인터셉터의 어딘가에서 MDC 컨텍스트를 채운다.
  - 로그를 평소처럼 기록한다.
- 하지만 리액터에서는 요청에 대한 처리가 서로 다른 쓰레드 간에서 재스케쥴링 될 수 있다.
  - 때문에 `ThreadLocal` 스토리지를 사용할 수 없다.

다행히도 리액터 프로젝트에서는 고유한 `Context` 추상화를 제공해서 리액티브한 연산자들의 파이프라인 간에 컨텍스트를 신뢰성 있게 전파할 수 있도록 한다. `Context`는 Subscriber에서 Publisher로 전파된다.

```java
@PostMapping("/someMethod")
fun someMethod(exchange: ServerWebExchange, request: Request): Mono<Response> {
  logger.debug { "Start processing $request.." }
  return anotherServiceStub.getInfo()
    .flatMap {
        logger.debug { "Got response from Another Service: $it. Saving to Redis..." }
        redisRepository.save(it)
    }
    .map {
        logger.debug { "Saved $it to Redis. Generating response..." }
        toResponse(it)
}
    }
```

위 코드에서 각 실행에 `traceId`를 붙인다고 가정해보자. 아래와 같이 될 것이다.

```java

@PostMapping("/someMethod")
fun someMethod(exchange: ServerWebExchange, request: Request): Mono<Response> {
  logger.debug { "Start processing $request.." }
  return anotherServiceStub.getInfo()
    .flatMap {
        logger.debug { "Got response from Another Service: $it. Saving to Redis..." }
        redisRepository.save(it)
    }
    .map {
        logger.debug { "Saved $it to Redis. Generating response..." }
        toResponse(it)
    }
    .subscriberContext { ctx -> ctx.put("traceId", getTraceId(exchange)) } 
    // TODO: Put traceId to reactor's context
}
```

이제 `Context`에서 `traceId`를 빼내어서 MDC 컨텍스트에 넣으면 된다. 이런 경우에 사용할 수 있는 리액터 연산자는 아래와 같다.

1. `subscriberContext()` : `Mono<Context>`를 리턴
2. `handle(element: T, sink:SynchronousSink<T>->...)` : `map`과 `filter` 보다 유연하게 사용될 수 있고, `sink`는 현재 컨텍스트를 포함한다는 장점이 있다.
3. `doOnEach(signal: Signal<T>)` : `doOnNext`와 비슷하게 동작하지만, 시그널을 사용한다는 점이 다르다.
   1. 시그널은 Publsher의 시선에서의 단일 이벤트를 표현한다.
   2. `onNext`, `onError`, `onComplete`, `onSubscribe`가 될 수 있다.
   3. `onNext` 시그널은 원소들을 포함할 수 있고, `onError`은 `Throwable`을 포함하고, 이 시그널들은 모두 `Context`를 포함한다.
4. `materialize()` / `dematerialize()` : `Mono` / `Flux`를 시그널들로 변환한다.

개인적으로는 로깅을 위해 간단한 컨텍스트 정보가 필요한 것이라면 `subscriberContext`를 사용할 것 같다. 앗.. 그런데 `deprecated` 되었단다.

https://github.com/reactor/reactor-core/issues/2572

> `Mono.deferContextual()`을 사용하자.
