---
title: "Webflux에서 Blocking 코드 제거하는 법"
date: 2023-01-12T20:46:49+09:00
draft: false
author: redjen
---
## 문제 상황

webflux를 사용하여 reactive한 어플리케이션을 작성하는 것은 어렵다.

어플리케이션의 실행 컨텍스트가 reactive stream인 상태에서 block() 메서드를 사용하게 되면 thread starvation에 빠지면서 계속 stall 되는 현상을 발견했다.

이는 netty의 작동 방식이 tomcat의 쓰레드 풀과는 달리 event loop group 안에서 blocking source에 대한 async한 작업을 비동기 / 논블럭킹 방식으로 수행하기 때문이다.

https://stackoverflow.com/questions/73488468/possibly-blocking-call-in-non-blocking-context-could-lead-to-thread-starvation

```java
// 위는 생략
.flatMap(someObject -> {

someObject.setEntityField(

Flux.fromIterable(someCollecionList)

.flatMap(someCollectionElem -> Flux.just
		 Tuples.of(
		 someCollectionElem, 
		 createSomeMonoModel1(param1, param2, param3(someCollectionElem))
		 )
	 )
 )

.collectMap(Tuple2::getT1, tuple2 -> createSomeMonoModel2(tuple2.getT2().block())).block()

);

return Mono.just(someObject);

})
// 아래는 생략...
```

요지는, MongoDB의 엔티티로 저장되는 객체 데이터의 한 필드는 ```Map<String, String>``` 형식으로 저장되는데, ```Flux<SomeObject>```를 ```.collectMap()```으로 묶으면 ```Mono<Map<String, String>>``` 꼴이 되어 blocking 코드를 사용하지 않으면 reactive한 문맥에서 엔티티에 데이터를 저장할 수 없는 문제였다.

## 변경 및 배웠던 점

상기 코드의 이전 부분에서는 ```Mono<SomeObject>```를 리턴하고 있었는데,
상기 코드를 ```flatMap()```을 사용하지 않고 이전 단계에서 ```Mono<Tuple3<SomeObject, SomeObject2, Map<String, String>>``` 을 리턴하도록 변경하였다.

그 후에 Tuple3 형식을 처리하는 마지막 단계 (blocking 코드가 이미 포함되어 있는)에서 나머지 데이터를 전부 꺼내어 데이터를 주입하는 형식으로 해결하였다.

Reactive 어플리케이션의 Reactive한 문맥에서라면 blocking 코드는 마지막에 한 데 묶어서 처리하고, 그 전까지는 reactive하게 계속 전달되도록 변경하는 것이 중요하다.

또한, Mono 객체 안에 또 Mono가 존재하지 않도록 빼내는 것도 중요한 것 같다.
zipWith()나 zip()을 사용해서 여러 데이터를 묶을 때에는
```Mono<Tuple2 <SomeObject, Tuple2<SomeObject2, Map<String, String>>``` 보다는
```Mono<Tuple3<SomeObject, SomeObject2, Map<String, String>>``` 이 되어야 한다.

Mono.zip()은 2개 이상의 Mono를 묶어서 하나의 Mono로 리턴하는 연산이다.
해당 연산을 잘 사용한다면 나쁜 가독성의 Tuple2 in Tuple2를 풀어 쓸 수 있도록 도움과 동시에,
Tuple2의 Mono 객체를 다시 block() 메서드를 써서 읽어야 하는 불상사를 해결한다.

