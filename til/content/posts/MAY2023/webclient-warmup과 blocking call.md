---
title: "Webclient Warmup과 blocking call"
date: 2023-05-12T22:18:08+09:00
draft: false
author: redjen
---

스프링을 사용해서 Http 요청을 할 때에는 이제 WebClient를 사용하는 것이 거의 표준이 되었다.

그런데, Spring Webflux 기반의 어플리케이션에서 WebClient를 사용해서 요청을 날리던 도중에 다음과 같은 에러 메시지를 볼 수 있었다.

```bash
java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking, which is not supported in thread webflux-http-nio-10
	at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:83)
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
```

## WebClient의 warmup

https://projectreactor.io/docs/netty/release/api/reactor/netty/http/client/HttpClient.html#warmup--

WebClient에는 warmup 과정이 필요하다.

공식 문서에 따르면, `warmup()`은 아래의 작업을 수행한다.
- 이벤트 루프 그룹의 초기화
- 호스트 네임 리졸버의 초기화
- Transport 작업을 위해 필요한 네이티브 라이브러리들의 로딩
- Security 설정 작업을 위해 필요한 네이티브 라이브러리들의 로딩

기본적으로 `warmup()` 이 사용되지 않는다면, 해당 WebClient를 사용해 날린 첫 번째 요청이 초기화와 warmup을 위해 필요한 추가 리소스들을 로딩하기 위해 부가적인 시간을 필요로 한다.

- 때문에 애플리케이션을 설계할 때에는 이 warmup 과정을 첫 번째 요청에 수행할 수도 있고 (lazy init)
- 또는 애플리케이션의 초기화 / warmup 과정에서 WebClient의 warmup 도 수행할 수도 있다. (eager init)

## warmup()은..

공식 문서에서 보았듯이 `warmup()` 메서드는 `Mono`를 반환한다.

그리고 스프링이 어플리케이션 컨텍스트를 빌딩할 때, 즉 어플리케이션이 기동을 시작할 때에는 Webflux일지라도 blocking call이 허용된다.

즉 리액티브, 논 블락킹 어플리케이션을 설계할 때 `@Bean` 초기화에 HttpClient 혹은 WebClient warmup을 eager하게 initialize하고 싶다면, `warmup().block()`과 같은 방식으로 수행할 수도 있다는 말이다.

### 왜 스프링의 컨텍스트 초기화 단계에서는 블락킹 콜이 허용될까?

어플리케이션에 필요한 설정들은 보통 파일을 통해 구성되어 있기 떄문이다. 파일을 일단 읽어서 컨텍스트를 초기화 시킨 후에야 내부 빈 설정들과 컨텍스트들이 동작하기 때문에, 그 후에는 어플리케이션 전체가 논블락킹으로 동작하게 되는 것이다. 

### 겪었던 문제

내가 겪었던 문제는 WebClient의 warmup을 lazy하게 수행하다가
- 공통화 로직에 포함되어 있는 `warmup().block()`이 어플리케이션 컨텍스트가 초기화 된 후
- 메인 이벤트 루프가 아닌 워커 쓰레드에서 해당 초기화 로직이 실행되어
- non-blocking 작업만 허용된 컨텍스트에서 blocking 콜을 수행했다.

이 경우에 동일한 코드로 똑같이 어플리케이션을 실행했을 때 정상적으로 로직이 수행되는 경우도 있었는데, 이 경우는
- 어플리케이션 컨텍스트 초기화 과정 중에서 lazy하게 초기화 설정해놨던 WebClient가 모종의 이유로 (health check?) 요청이 들어오기 전 초기화 / warmup 작업을 수행했기 때문으로 보인다.
- 때문에 동일한 코드에서 똑같은 로직을 수행해도 어떨 때는 정상적으로 API를 호출할 수 있었던 반면, 대부분의 경우에는 lazy하게 초기화되는 WebClient Bean에 포함된 블락킹 로직 때문에 상기 언급했던 에러를 볼 수 있었다. 
- eager하게 스프링 컨텍스트 초기화 과정에서 문제가 발생했던 WebClient의 warmup을 수행함으로써 문제를 해결할 수 있었다.

웹플럭스는 공부하면 할 수록 어렵다.. 서비스 레이어에서 블락킹 콜이 없는데도 불구하고 블락킹 콜이 호출되어 깜짝 놀랐다.