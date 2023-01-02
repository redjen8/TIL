---
title: "Webflux에서 ResponseEntity 리턴 시 유의해야 할 점"
date: 2023-01-02T19:31:54+09:00
draft: false
author: redjen
---

# Webflux에서 ResponseEntity 리턴 시 유의해야 할 점

Webflux를 사용할 때, Controller에서 ResponseEntity 형태로 dto를 리턴하고 싶을 때가 있다.
이 때, 
```Mono<ResponseEntity>``` 와 ```ResponseEntity<Mono>```
는 어떻게 다를까?

https://stackoverflow.com/questions/57769190/what-is-the-difference-between-responseentitymono-and-monoresponseentity-as?rq=1

ResponseEntity를 어떻게 리턴하는지에 대한 다양한 옵션들이 있다.

## 1.  ```ResponseEntity<Mono<T>>``` 또는 ```ResponseEntity<Flux<T>>```

Response status와 header를 바로 리턴한다.
하지만 **response body는 비동기적으로 나중에 리턴**된다. body가 Mono인지, Flux인지는 response 개수에 따라 달라질 수 있다.

## 2. ```Mono<ResponseEntity<T>>```

response status, header, body를 모두 비동기적으로 나중에 리턴한다.
response status와 header가 비동기적인 요청 처리에 의해 변화할 수 있을 때 사용한다.

## 3. ```Mono<ResponseEntity<Mono<T>>>``` 또는 ```Mono<ResponseEntity<Flux<T>>>```

역시 선택 가능한 옵션이지만 잘 쓰이지는 않는다.
response status와 header를 비동기적으로 먼저 리턴한 다음,
response body를 나중에 비동기적으로 리턴한다. 

## 무엇을 선택하면 좋은지?

물론 클라이언트에게 어떤 방식으로 데이터를 전달해주어야 한다는 스펙에 따라 달라지겠지만, 일반적인 webflux 어플리케이션에서 응답을 전달할 때에는 ```Mono<ResponseEntity<T>>``` 나  ```Flux<ResponseEntity<T>>``` 를 사용할 일이 많아 보인다. 

비동기적으로 요청을 처리할 때 response status나 header 값을 바꿀 수 있기 때문에 exception 처리 등에서 용이한 면이 있을 것 같다.

팀에서는 API 결과를 굳이 리턴하지 않아도 되는 DELETE, 사용자에게 요청 처리 결과를 알려주지 않아도 되는 POST 요청의 경우 ```Mono<Void>```로 리턴하고 있다.

API 결과를 리턴해야 하는 POST 요청이나 GET요청에 대해서는 Dto를 그냥 ```Mono<dto>```나 ```Flux<dto>``` 로 래핑하여 리턴한다. 

Spring에서는 객체를 Jackson 라이브러리를 통해 Json으로 직렬화한다. Webflux 또한 Dto를 Mono나 Flux 등의 Publisher로 래핑하여 리턴할 때 Jackson 라이브러리를 사용한다. 

[공식 documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-ann-responseentity) 을 자주 보게 될 것 같다.. WebMVC와의 차이점이 잘 나타나 있는 것 같다. ModelAttribute를 사용할 일이 있다면 사용법이 다른 것에 유의해야 한다. 
