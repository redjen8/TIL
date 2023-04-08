---
title: "Java Reactive Programming Flux Generate"
date: 2023-04-08T22:32:27+09:00
draft: false
author: redjen
---

이전 강의에서는 Flux가 무엇인지에 대해 주로 알아봤었다.

Flux를 언제, 어떻게 생성해야 할까?

## create를 사용한 Flux 생성

```java
Flux.create(fluxSink -> {
	fluxSink.next(1);
	fluxSink.next(2);
	fluxSink.complete();
}).subscribe(...)
```

처럼 `create`를 사용하면 `Consumer` 내부에서 
- 좀 더 커스텀하게 다음 시그널을 `next`로 내려줄지
- `complete` 완료 시그널을 줄 지
- `error` 에러 시그널을 줄 지 프로그래밍 적으로 결정할 수 있다.

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#create-java.util.function.Consumer-

Flux.create 안에서 반복문을 사용한다면 내부적으로 여러 개의 객체 시그널을 생성하여 전달할 수 있지만.. '`create` 안에서 반복문을 사용하는 것' 이 최선일까?

`create` 내부에서 반복문을 사용하는 대신 `FluxSink`를 generate하는 producer를 사용하여 이를 대체할 수 있다. 

이 방법은 명시적으로 반복문을 사용하지 않기 때문에 몇 가지 이점을 가져올 수 있지만, 그보다 더 주요한 장점이 있다.

## FluxSink의 멀티 쓰레딩

```java
Flux.create(nameProducer)
	.subscribe(...);

Runnable runnable = nameProducer::produce;

for (int i = 0; i < 10; i++) {
	new Thread(runnable).start();
}

sleep(2);
```

위와 같은 코드는 한번에 10개의 runnable 객체로부터 flux를 생성하고 동시에 이를 구독한다.

실행 시에는 runnable 으로부터 생성된 객체 10개가 서로 다른 쓰레드 10개에서 소비된다.

## take 연산자

`map`, `filter`와 같은 연산자는 파이프라인을 통해 전달되는 값을 다룰 수 있는 유용한 여러 기능들을 제공한다.

`take` 연산자는 인자로 들어온 개수만큼의 시그널만 전달하고, 이후 전달되는 시그널은 무시한다.

```java
Flux.range(1, 10)
	.log()
	.take(3)
	.log()
	.subscribe(...);
```

위 코드는 `onNext(1)`, `onNext(2)`, `onNext(3)` 을 호출하고, 이후에는 `onCancel()`을 호출하여 Subscriber가 Publisher에게 구독 관계를 중지할 것을 요청한다.

중요한 것은 upstream subscription을 cancel 함과 동시에 publisher는 더 이상 원소를 생성하지 않고 바로 `onComplete` 시그널을 보낸다는 것이다. 

## Stream 취소 이벤트

앞서 말한 `FluxSink` 내 반복문을 사용해서 데이터를 emit 하는 예제에서, `take` 연산자를 사용해 중간에 구독관계를 취소하면 어떻게 될까?

결론을 말하자면, Subscriber는 구독 관계가 취소되었기 때문에 더 이상 publisher가 생성하는 데이터를 받고 있지 않지만 publisher는 취소된 사실 자체를 모르기 때문에 원래 생성하려던 만큼 계속해서 데이터를 생성하려 한다.

그리고 이렇게 생성된 데이터는 아무 subscriber에게도 전달되지 않기 때문에 낭비다.

> `fluxSink` 내의 `isCancelled()` 메서드를 통해 구독관계가 취소되었는지 여부를 체크할 수 있다.

## Flux.generate

지금까지의 예제에서 다뤘던 `Flux.create`는 Consumer 객체 내부에서 생성되는 시그널을 자유자재로 다룰 수 있었지만
- 단 하나의 `FluxSink` 인스턴스만 생성하고
- 제대로 다루기 위해서는 주의를 기울여야 하며
- 하나의 인스턴스에 대해 emit이 종료되면 그걸로 끝이라는 단점을 가지고 있다.

실제로 사용되기엔 생산성 자체가 높지 않다고 할 수 있다.

`Flux.generate`는 `Flux.create`의 사용법과 거의 비슷한 용법을 가지고 있다. 다른 점으로는..
- `FluxSink` 대신 `SynchronousSink`를 사용한다.
- 이 `SynchronousSink`는 `next`를 사용해서 최대 하나의 아이템을 emit할 수 있다.
	- 하나 이상의 아이템을 emit하려는 시도가 있을 때에는 에러가 발생한다.

하나의 아이템 밖에 emit할 수 없다니, 이거 완전 `Mono` 아니야? 라고 할 수 있지만 실제로는 `synchronousSink.next()`를 통해서 생성되는 아이템은 무한한 Stream을 생성한다.

마치 loop 안에서 `FluxSink.next`를 끊임없이 생성하는 것처럼 동작하는 셈이다.

만약 `Flux.generate`를 통해 생성된 Flux에 `take` 연산자를 사용한다면 어떻게 될까?
- `Flux.create`를 통해 `FluxSink`를 다룰 때에는 구독관계의 취소를 publisher가 전달받지 못했지만
- `Flux.generate`를 통해 `SynchronousSink`를 다룰 때에는
	- `take()`를 통해 발생되는 구독 관계의 취소도,
	- 완료 시그널을 발생하는 `complete()` 메서드도,
	- `error()` 메서드를 통해 발생되는 Exception도 전부 제어가 가능하다.

### with State

하지만 `synchronousSink`라는 이름에서도 유추할 수 있듯이, `Flux.generate` 내부에서는 동시성 이슈가 발생할 수 있다.

`Flux.generate`를 통해서 간단한 카운터 예제를 생성한다고 했을 때 동시성 이슈를 해결하기 위해 `AtomicInteger` 등을 사용하면 해결될까? 그렇지 않다.

`AtomicInteger`는 동일 블럭 내에서 조작했을 때 Flux 외부에서도 내부의 데이터에 영향을 미칠 수 있기 때문에 좋지 않다.

`Flux.generate`에서 상태를 다루려면 인자에 `BiFunction`을 넣어준다. 자바스크립트의 `reduce`처럼.

```java
Flux.generate(
	() -> 1,
	(counter, sink) -> {
		sink.next(someItem);
		if (someCompleteCondition)
			sink.complete();
		return counter + 1;
	}
)
.subscribe(...);
```

## Flux Push

`Flux.create`와 `Flux.generate` 가 약간 섞인 방법으로 `Flux.push`가 존재한다.

하지만 `Flux.push`는 쓰레드 세이프하지 않기 때문에 싱글 쓰레드 producer에서만 사용할 수 있다.

실제로는 거의 사용하지 않을 것 같다.

## 정리

create | generate
--- | ---
`Consumer<FluxSink<T>>`를 Accept | `Consumer<SynchronousSink<T>>`를 accept
Consumer는 한 번 호출됨 | Consumer는 downstream 요구에 따라 재요청될 수 있음
Consumer는 0~N 원소를 즉각적으로 emit 가능 | Consumer는 하나의 원소만 emit 가능
Publisher는 downstream 처리 속도를 모르기 떄문에 overflow 전략 사용해야 함 | Publisher가 downstream 수요에 맞춰 원소를 produce
쓰레드 세이프 | N/A
`fluxSink.requestedFromDownStream()`, `fluxSink.isCancelled()` 유틸 제공 | N/A

