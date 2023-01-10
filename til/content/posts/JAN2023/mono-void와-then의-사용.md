---
title: "Mono Void와 then()의 사용"
date: 2023-01-10T20:37:34+09:00
draft: false
author: redjen
---

deleteMapping으로 리소스를 삭제하는 API들은 ```.then()```을 사용해서 ```Mono<Void>``` 를 반환한다.

> ```Mono<Void>```는 뭐고, 왜 사용할까?

## Mono\<Void\>

https://stackoverflow.com/questions/59196530/why-reactor-monovoid-is-recognized-as-an-empty-mono

reactive 스택에서는 null을 반환하는 것을 금지하고 있다.
Mono는 완료 신호를 보내기 전에 아무것도 반환하지 않거나, 하나의 값만을 반환하도록 명세되어 있다.

따라서 ```Mono<Foo>```는 두 가지 옵션이 있다.
- 완료 신호를 보내거나
- ```Foo``` 인스턴스를 보낸 후에 완료 신호를 보내거나

인스턴스를 보내지 않는 경우 (값이 publish 되지 않는 경우)에는 크게 두 가지 경우가 존재한다.
1. 값이 존재하지 않는 경우
	1. 이 경우에는 데이터베이스나 컬렉션에서 탐색을 한 후 존재하지 않는 경우이다.
	2. 이 때에도 ```Mono<SomeType>``` 을 사용할 수 있으며, 이경우에는 ```SomeType``` 인스턴스를 반환할 수도 있고 아닐 수 도 있다.
3. 값이 절대로 publish 되지 않는 경우
	1. 작업이 완료되었음을 알려주기 위한 경우에 자주 사용된다.
	2. 이 경우에는 ```Mono<Void>``` 를 관례 상 많이 사용한다.

```Void``` 는 절대로 인스턴스화 될 수 없도록 디자인된 클래스이다.
```Void``` 인스턴스는 존재할 수 없기 때문에 ```Mono<Void>```는 인스턴스를 반환하는 것이 아니라 완료 신호만을 보내는데 사용된다.

## then() operator

then~ 이 붙은 operator들의 공통점으로는
- onNext() 신호를 무시한다.
- 완료 신호에 따른 reaction 들 (onComplete나 onError) 도 무시한다.
- 대신 그 시점에 여러 옵션들을 가지고 시퀀스를 유지시킨다.

1. then() 연산자는 단지 source의 종료 신호를 재생하고 ```Mono<Void>``` 를 반환해서 onNext를 절대로 신호하지 않는다는 것을 나타낸다. (작업이 완료되었으므로)
2. thenEmpty() 연산자는 ```Mono<Void>``` 를 리턴하는 것 뿐만 아니라 ```Mono<Void>```를 파라미터로 입력 받는다. thenEmpty()는 source의 완료 신호와, 빈 Mono 완료 신호를 연결한 (concatenation) 결과를 나타낸다.
	1. 즉 A와 B가 전부 순차적으로 완료되었을 때 완료되며, 아무런 데이터도 반환하지 않는다.
3. thenMany() 연산자는 source의 완료를 기다리고, ```Publisher<R>``` 파라미터로 받은 신호들을 전부 재생하여, ```Flux<R>```을 반환한다. 이 Flux는 source가 완료될 때까지 일시정지하고, 완료된 이후에는 제공된 publisher로부터 다수의 element들을 반환하고 완료 신호를 준다.
