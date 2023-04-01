---
title: "Java Reactive Programming Flux"
date: 2023-04-01T16:48:29+09:00
draft: false
author: redjen
---

## Flux

0 또는 N개의 아이템을 emit할 수 있는 publisher이다.
Subscriber가 50개의 아이템 요청 - Publisher가 3개 밖에 없다면?
- 3개 emit하고 `onComplete`
- 처리 중에 에러 발생 시 `onError`

## Flux.just

최대 하나의 아이템을 emit할 수 있는 Mono와는 달리 Flux는 `.just()` 를 통해 생성할 때에도 다수의 인자를 입력할 수 있다. (`Flux.just(T...)`)

아무 데이터도 없을 때에는 `Flux.empty()`를 사용한다.
- `Mono.empty()`와 동일하게 바로 `onComplete` 시그널을 전달한다.

## Multiple Subscribers

```java
Flux<Integer> integerFlux = Flux.just(1, 2, 3, 4);

integerFlux.subscribe(i -> System.out.println("Subscriber 1 : " + i));
integerFlux.subscribe(i -> System.out.println("Subscriber 2 : " + i));
```

위와 같이 subscriber를 추가해서 순차적으로 Flux의 데이터를 구독시킬 수 있다.
만약 짝수 정수만 입력 받고 싶어서 다음과 같은 코드를 중간에 삽입하면 어떻게 될까?

```java
Flux<Integer> integerFlux = Flux.just(1, 2, 3, 4);

Flux<Integer> evenFlux = integerFlux.filter(i -> i % 2 == 0);

integerFlux.subscribe(i -> System.out.println("Subscriber 1 : " + i));
integerFlux.subscribe(i -> System.out.println("Subscriber 2 : " + i));
```

Subscriber 1은 1, 2, 3, 4를 전달 받고, Subscriber 2는 2, 4만 전달 받는다.
Hot / Cold Publisher에 대한 개념은 나중에 배워보도록..

## From Array / List

Flux는 0또는 N개의, 아이템을 emit할 수 있기 때문에, 사전 구성된 List나 Array로부터 데이터를 불러오는 것이 `Flux.just`보다 더 편할 때가 많다.

`Flux.fromIterable()` 을 통해 Iterable한 자료구조로부터 Flux를 간편하게 생성할 수 있다.

```java
List<String> strings = Arrays.asList("a", "b", "c");

Flux.fromIterable(strings)
	.subscribe(...)
```

## From Stream

```java
List<Integer> list = List.of(1, 2, 3, 4, 5);
Stream<Integer> stream = list.stream();

stream.forEach(System.out::println);
stream.forEach(System.out::println); //error
```

Java 8의 Stream은 one time use 이기 때문에, 동일한 Stream을 다시 사용했을 때에는 'stream has already been operated upon or closed' 에러를 볼 수 있다.

Flux도 동일할까? 그렇다.

```java
Flux<Integer> integerFlux = Flux.fromStream(stream);
integerFlux.subscribe(...);
integerFlux.subscribe(...); //error
```

위와 같은 방법으로 Stream을 통해 생성된 Flux는 Subscriber가 재구독할 시에 에러를 발생시킨다.
- `.subscribe()`가 한번 발생할 때마다 Flux는 원본 Stream에 접근해서 데이터를 가져오려는 시도를 한다.
- 두 번째 시도부터는 Stream이 이미 닫힌 상태이기 때문에 에러가 발생한다.

```java
Flux<Integer> integerFlux = Flux.fromStream(() -> list.stream());
integerFlux.subscribe(...);
integerFlux.subscribe(...); //good
```

위와 같은 방법으로, Flux에 대한 Subscribe 행위가 일어날 때마다 기존 List의 Stream을 생성한다면 Stream이 재사용될 일이 없으므로, 에러도 발생하지 않는다.

## Flux.range

`Flux.range(start, count)`는 간단하게 increment하는 숫자의 Flux를 생성하기 간편한 방법이다.

```java
Flux.range(1, 10)
	.subscribe(System.out::println);
```

위 코드는 1 ~ 10까지의 정수를 출력하게 된다.
`Flux.range()` 는 정수를 emit하는 Flux이지만 경우에 따라 For loop처럼 사용할 수 있다.
특정 데이터를 제네레이팅하는 함수를 반복하고 싶을 때 유용할 수 있을 것 같다. 아래처럼..

```java
Flux.range(1, 10)
	.map(SomeGenerator::getRandomString)
	.subscribe(System.out::println);
```

## log

기나긴 파이프라인을 디버깅하긴 힘들다.
Flux 파이프라인에 `.log()`을 끼워 넣어서 어떤 시그널들이 전달되고 있는지를 염탐할 수 있다.

`map` operator 전후로 log를 끼워 넣었을 때 로깅되는 데이터가 달라질 때 유용하게 사용할 수 있을 것 같다.

## Custom Subscriber Implementation

지금까지는 Reactor에서 제공하는 Subscriber 객체를 사용해서 Publisher가 전달한 시그널을 받아서 사용하였는데, Subscriber 객체를 커스텀하게 사용할 수는 없을까?

```java
AtomicReference<Subscription> atomicReference = new AtomicReference<>();
Flux.range(1, 20)
	.log()
	.subscribeWith(new Subscriber<Integer>() {
		@Override
		public void onSubscribe(Subscription subscription) {
			System.out.println("Received Subscription: " + subscription);
			atomicReference.set(subscription);
		}

		@Override
		public void onNext(Integer integer) {
			System.out.println("onNext: " + integer);
		}

		@Override
		public void onError(Throwable throwable) {
			System.out.println("onError: " + throwable.getMessage());
		}

		@Override
		public void onComplete() {
			System.out.println("onComplete");
		}
	});
```

위 코드를 실행하면 `onSubscribe`만 실행된다.
데이터가 요청되지 않으니, Flux Publisher는 데이터를 내려주지 않는다.

기존 Reactor의 Subscriber들은 내부적으로 요청하는 일들을 수행하고 있었던 것 처럼 우리의 커스텀 Subscriber가 데이터를 요청하도록 변경해야 한다.

```java
sleep(3);
atomicReference.get().request(3);
sleep(3);
atomicReference.get().request(3);
sleep(2);
atomicReference.get().cancel();
sleep(3);
atomicReference.get().request(4);
```

위 Publisher 명세 이후 상기 코드를 사용한다면, 3초 sleep 이후 3개의 아이템을 요청하고, 그 후에 다시 3초 sleep 이후 3개의 아이템을 요청한다.
`cancel()` 을 요청한 다음에 이뤄지는 `request`는 발생하지 않는다는 것을 기억해야한다.

## Flux vs List

여러 개의 데이터 원소를 받기 위해서는 어떤 방법을 쓸 수 있을까?

List에 데이터를 채운 후 사용하려면.. 데이터가 전부 채워질 때까지 기다려야 한다. (블락킹)
- 데이터의 개수가 엄청 많거나
- 엄청 큰 데이터를 List에 담는다고 한다면 상당히 많은 시간이 걸린다.

이는 Flux를 사용해서 개선할 수 있다.

하나의 원소를 제네레이팅 한 다음 처리하고, 다음 원소가 제네레이팅 된 직후 처리하고..

조금 더 컴퓨팅 자원을 효율적으로 사용할 수 있게 된 것이다.

다만 Flux는 List처럼 Collection 도 아니고, 자료구조도 아니다.
데이터를 어떻게 전달해줘야 하는지에 대한 개념 혹은 명세에 가깝다는 생각이다.

## Interval

Flux를 사용하면 여러 개의 아이템을 전달할 수 있다.

전달할 때 짧은 주기를 가지고 데이터를 전달하려면 `Flux.interval`을 사용해서 Flux를 사용하면 된다.

```java
Flux.interval(Duration.ofSeconds(5))
	.subscribe(System.out::println);

Sleep(5);
```

## Another Publisher to Flux

Mono에서 Flux로 변경하려면 어떻게 해야 할까?

`Flux.from()` 은 특정 형태의 Publisher를 Flux로 변경해주는 메서드이다.

```java
Mono<String> mono = Mono.just("a");
Flux<String> flux = Flux.from(mono);
```

## Flux to Mono

```java
Flux.range(1, 10)
	.next()
	.subscribe(System.out::println);
```

Flux는 0에서 N개의 아이템을 전달한다. 그 중 하나만을 선택해서 Mono로 전달하려면 `.next()` 메서드를 사용해서 전달한다.

`next()`는 다음 `onNext` 시그널에 전달되는 아이템을 단일 Mono로 전달한다.

## 정리

### Flux 생성
조건 | 사용법
--- | ---
이미 존재하는 데이터로부터 | `Flux.just()`, `Flux.fromIterable()`, `Flux.fromArray()`, `Flux.fromStream()`
범위나 횟수 | `Flux.range(start, count)`
일정 주기를 가지고 | `Flux.interval(duration)`
`Mono`로부터 | `Flux.from(mono)`
 
### Mono vs Flux : 비어 있는 데이터와 Exception 처리

구분 | `Mono<T>` | `Flux<T>`
--- | --- | ---
Void / Null | `Mono.empty()` | `Flux.empty()`
Exception | `Mono.error(throwable)` | `Flux.error(throwable)`