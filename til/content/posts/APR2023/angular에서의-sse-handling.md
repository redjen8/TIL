---
title: "Angular에서의 Sse Handling"
date: 2023-04-03T19:46:20+09:00
draft: false
author: redjen
---

어제는 Spring Webflux와 Spring MVC에서 SSE를 사용한 단방향 이벤트 통신이 어떻게 이뤄질 수 있는지 알아보았다.

그렇다면 이 이벤트들을 화면단에 받아서 리액티브한 페이지를 만드려면 어떻게 해야할까?

https://medium.com/swlh/how-do-server-sent-events-sse-or-eventsource-work-in-angular-e9e27b6a3295

## EventSource

ReactiveX 재단의 대표 리액티브 라이브러리인 RxJS에서는 Observable을 사용하여 SSE를 이벤트 소스로써 구독하고, 수신한 이벤트를 데이터로써 리액티브하게 다루는 방법을 제공한다.

https://stackoverflow.com/questions/36827270/creating-an-rxjs-observable-from-a-server-sent-eventsource

리액티브 스트림 표준에 알맞게 EventSource는 `next`, `error`를 통해서 다음 시그널에 대한 데이터를 처리하고 예외를 처리할 수 있도록 지원한다.

어려운 점은 데이터를 리액티브하게 받는 동시에 View 또한 리액티브하게 변경해야 한다는 점이다.
이건 다음 기회에 알아보도록 하는 것으로..

## 왜 기본 HttpClient에서 지원하지 않을까

지난번에도 알아봤지만, WebSocket과는 달리 Server Sent Event는 HTTP 표준 스펙이다.

그런데 Angular의 기본 HttpClient에서는 SSE를 구독할 수 있는 방법을 제공하지 않는다.
조금 더 찾아보니 React도 마찬가지인 듯 하다.

왜 기본 Http Client에서 표준 http 스펙인 SSE를 지원하지 않을까?
1. MS IE와 Edge 브라우저에서는 SSE를 지원하지 않는다.
	1. 따라서 MS의 브라우저에서 SSE를 지원하려면 polyfill을 사용해야 한다.
2. EventSource API는 커스텀 헤더를 지원하지 않는다.
3. 제일 중요한 것 같은데, 이미 EventSource가 SSE를 지원하는 표준 API가 된 듯하다. ([MDN 문서](https://developer.mozilla.org/en-US/docs/Web/API/EventSource))
	1. 그리고 나는 여기에 ReactiveX 재단의 영향력이 상당 부분 미쳤다고 생각한다. 

특히 기본 SSE 스펙 자체에 비즈니스적인 로직을 끼얹어 커스텀해서 사용해야 하는 경우에는 EventSource에서 받지 못하는 경우가 많은 것 같다. (특히 토큰 인증 등)

이러한 제약 때문에 RxJs의 EventSource를 사용하지 않고 [sse.js](https://github.com/mpetazzoni/sse.js?files=1)와 같은 외부 라이브러리를 사용하는 경우도 있는 것 같다.

