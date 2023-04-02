---
title: "Sse in Spring"
date: 2023-04-02T21:16:53+09:00
draft: false
author: redjen
---

ServerSideEvent는 웹 어플리케이션이 단방향의 이벤트 스트림을 다루고 서버가 emit하는 데이터를 업데이트 하기 위한 기술이며, HTTP 표준 스펙이 되었다.

이제 Spring을 사용할 때 서버에서 데이터를 쏴주는 기능을 원한다면 Websocket과 SSE 두 개의 선택지가 있게 된 셈이다.

스프링 5버전 부터는 이를 훨씬 더 쉽게 다룰 수 있게 되었다.

어떻게 다루는지 알아봤다.

https://www.baeldung.com/spring-server-sent-events

## Spring 5 Webflux에서의 SSE

Reactor 라이브러이의 `Flux` 클래스를 사용한다면 가장 쉽게 SSE를 구현할 수 있다.

### Flux를 사용한 단순한 스트림 이벤트

SSE 스트리밍 엔드포인트를 생성하기 위해서는 W3C 스펙을 따라야 한다.

따라서 해당 Controller의 Produce MIME type을 `text/event-stream`으로 지정해줘야 한다.

```java
@GetMapping(path = "/stream-flux", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamFlux() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(sequence -> "Flux - " + LocalTime.now().toString());
}
```

사전 정의된 엔드포인트 (`/stream-flux`)로 GET 요청을 날리게 되면, 서버는 1초마다 생성된 Flux 데이터를 클라이언트에 전달한다.

그 동안 커넥션은 계속 유지된다.

### ServerSentEvent 객체를 사용하는 방법

위 방법은 단순히 Flux 객체를 사용해서 보내고 싶은 데이터를 래핑하여 보내준다.

스프링에서는 사전 정의된 `ServerSentEvent` 객체를 사용해서 SSE를 다룰 수 있도록 지원하는데, 이는 몇 가지 이점이 있다.
1. 각 이벤트들의 메타 데이터를 다룰 수 있게 해준다. 실제 서비스 시나리오에서는 메타 데이터 없이 단순히 데이터만 보낼 일이 없기 때문에 이 편이 더 편리하게 개발할 수 있다.
2. `text/event-stream` MIME 타입 정의 없이도 SSE를 사용할 수 있게 해준다.

`ServerSentEvent` 객체는 다음 정보를 포함한다.
- id
- 이벤트 이름
- 이벤트의 데이터
- 이벤트에 대한 comment
- 이벤트 전송 시도에 사용될 재연결을 정의하는 retry 값 

### WebClient를 사용해서 SSE로 전달된 값 다루기

SSE로 전달된 값은 브라우저 및 FE에서 소비될 수도 있지만, 서버에서 소비할 수도 있다.

WebClient는 이벤트 스트림을 소비하는 방법을 편리하게 제시해준다.

```java
public void consumeServerSentEvent() {
    WebClient client = WebClient.create("http://localhost:8080/sse-server");
    ParameterizedTypeReference<ServerSentEvent<String>> type
     = new ParameterizedTypeReference<ServerSentEvent<String>>() {};

    Flux<ServerSentEvent<String>> eventStream = client.get()
      .uri("/stream-sse")
      .retrieve()
      .bodyToFlux(type);

    eventStream.subscribe(
      content -> logger.info("Time: {} - event: name[{}], id [{}], content[{}] ",
        LocalTime.now(), content.event(), content.id(), content.data()),
      error -> logger.error("Error receiving SSE: {}", error),
      () -> logger.info("Completed!!!"));
}
```

subscribe 메서드는 이벤트를 성공적으로 받았을 때, 에러가 발생했을 때, 스트리밍이 완료되었을 때 행할 행동을 각각 명시할 수 있게 한다. (Reactor 라이브러리의 사용과 동일하다)

## Spring MVC에서의 SSE 스트리밍

웹플럭스가 아닌 Spring MVC에서 SSE 스트리밍을 하려면 어떻게 해야 할까?

`SseEmitter`가 데이터를 푸싱하고 emitter 인스턴스를 리턴하며, 커넥션을 유지하기 위한 쓰레드인 `ExecutorService`를 정의해서 SSE 스트리밍을 할 수 있다.

```java
@GetMapping("/stream-sse-mvc")
public SseEmitter streamSseMvc() {
    SseEmitter emitter = new SseEmitter();
    ExecutorService sseMvcExecutor = Executors.newSingleThreadExecutor();
    sseMvcExecutor.execute(() -> {
        try {
            for (int i = 0; true; i++) {
                SseEventBuilder event = SseEmitter.event()
                  .data("SSE MVC - " + LocalTime.now().toString())
                  .id(String.valueOf(i))
                  .name("sse event - mvc");
                emitter.send(event);
                Thread.sleep(1000);
            }
        } catch (Exception ex) {
            emitter.completeWithError(ex);
        }
    });
    return emitter;
}
```

스프링 MVC는 쓰레드를 요청에 1대1 매핑시키는 것이 기본 전략이기 때문에, 서버의 가장 비싼 자원 중 하나인 쓰레드를 이렇게 별도의 작업을 위해서 사용되는 것은 주의를 요하는 작업이다.

따라서 유즈케이스 시나리오에 알맞는 `ExecutorService`를 골라야 한다.

## SSE에 대해

SSE는 대부분의 브라우저에서 어느 때라도 단방향 이벤트 스트리밍을 허용하도록 구현된 스펙이다.
- 이 이벤트들은 단순한 UTF-8 인코딩된, 표준에 정의된 포맷을 따르는 텍스트 데이터이다.
- SSE 이벤트의 포맷은 키 - 밸류 데이터들로 이루어져 있다.
  - 앞서 언급했던 id, retry, 데이터, 이벤트 등이 이에 속한다.
  - 각 요소들은 line break으로 구분된다.
- 하지만 SSE 스펙 자체는 데이터 페이로드 포맷을 제한하지 않는다.
  - 이 말은 즉 SSE 스펙으로 단순한 String이나 보다 복잡한 JSON, XML 구조를 가질 수 있다는 것을 뜻한다.


SSE 스트리밍과 웹 소켓은 어떻게 다를까?
- 웹 소켓은 서버 - 클라이언트 간 full duplex 통신을 지원한다.
- SSE는 단방향 통신만 지원한다.
- 또한 웹 소켓은 HTTP 프로토콜이 아니고, 에러 핸들링 표준이 존재하지 않는다. SSE는 HTTP 표준 프로토콜에 포함되어 있다.