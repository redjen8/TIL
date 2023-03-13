---
title: "Webclient Exchange vs Retrieve"
date: 2023-03-13T19:29:51+09:00
draft: false
author: redjen
---

WebClient를 사용해서 HTTP 요청을 리액티브 스트림 방식으로 받는 로직이 이제는 더 이상 이상하지 않다.

그런데 WebClient에는 요청에 대한 응답을 받는 방식이 크게 두 가지가 있다.

exchange와 retrieve는 어떻게 다를까?

https://stackoverflow.com/questions/58410352/spring-boot-webclients-retrieve-vs-exchange

## Exchange()

`exchangeToMono()`나 `exchangeToFlux()`와 같은 `exchange()` 류 메서드들은 다음과 같은 특징이 있다.
- `ClientResponse` 를 리턴한다.
- `ClientResponse` 안에는 응답 상태와 헤더가 포함되어 있다.
- 물론 응답의 body도 포함되어 있다.

## Retrieve()

`retrieve()` 메서드는 반면 `ClientResponse` 객체를 반환하지 않고 response body만 반환한다.

## 언제 어떤 것을 사용할까?

두 메서드 간 아주 큰 차이점은 헤더의 유무라고 생각된다.

만약 `HEAD` 요청을 통해 서버를 단순히 찔러본다거나 하는 등의 동작은 반드시 `exchange()`류 메서드를 사용해야 한다. 헤더가 포함되니까!

반면 HTTP 응답을 보내고 단순히 응답을 받기를 원할 때에는 `retrieve()`를 사용하면 된다.

대신 `retrieve()`를 사용하면 에러 처리가 좀 까다로워지지 않느냐? 할 수 있는데.
- 어차피 `Mono<T>` Publisher 시그널이 전달될 때 발생하는 `onError`의 경우 reactor 자체에서 핸들링이 가능하다.
- 예를 들자면 `onErrorMap` 등을 통해 response body를 `Mono<T>`로 변환하고 다음으로 전달 할 때
	- 발생한 `onError` 를 받아서 `Exception`을 throw 하는 방식으로 에러를 처리할 수 있다.

하지만 위 방식의 경우 HTTP 요청에 대한 세밀한 예외처리가 불가능하다. (status code에 따른 분기 처리라던지)
> 때문에 아주 통제된 환경이나 제한된 상황이 아니라면 `ClientResponse`를 받는 `exchangeToMono()`나 `exchangeToFlux()`를 쓰는 것이 좋아보인다.

