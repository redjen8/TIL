---
title: "Java Reactive Programming - Mono"
date: 2023-03-12T22:40:11+09:00
draft: false
author: redjen
---

## Reactor Publisher

Reactor는 Hibernate처럼 리액티브 스트림 구현체이다.
Reactor의 Publisher에는
1. `Mono<T>` : 0 또는 1개의 아이템을 emit할 수 있는 publisher이다.
	1. 아이템을 emit했다면 `onComplete`
	2. 아이템을 emit하지 못했다면 `onError`
	3. 아이템을 emit하지 않고 `onComplete`도 가능
2. `Flux<T>` : 0 또는 N 개의 아이템을 emit할 수 있는 publisher이다.
	1. 끝나지 않는 데이터의 스트림은 `Flux`를 통해 전달 가능
	2. publisher가 데이터를 전부 전달 후 `onComplete` 메서드를 호출하거나
	3. 데이터를 전달하는 과정에서 에러 발생 시에 `onError` 메서드를 호출할 수 있다.

### 왜 Mono가 필요할까?

단 하나의 아이템이 publisher로부터 생산될 것을 아는 상황에서는 `Mono`가 엄청 편리하기 때문이다.
	테이블에 얼마나 많은 데이터가 있는지 조회하는 쿼리의 경우가 이에 해당한다.

구분 | 현재 데이터 | 이전 데이터들
 --- | --- | ---
 Java | `Address` / `null` | `List<Address>`
 Java Stream | `Optional<Address>` | `Stream<Address>`
 Reactor | `Mono<Address>` | `Flux<Address>`

### Stream Lazy Behavior

Stream을 생성 후 바로 출력하게 된다면 어떤 일이 벌어질까?
 - 아무 일도 벌어지지 않는다.
 - `Stream`에 terminal한 operator를 연결하기 전까지는 lazy하게 동작하기 때문이다.

## Mono를 생성하는 여러 방법들

### Just

Mono를 생성하는 가장 쉬운 방법은 `.just()`를 사용하는 것이다.
```java
Mono<Integer> mono = Mono.just(1);
```

> Stream에서처럼, Publisher를 subscribe하기 전까지는 아무 일도 일어나지 않는다는 것을 명심해야 한다.

Just로 생성한 Mono를 바로 출력하게 되면 똑같이 아무 일도 벌어지지 않는다.
- Mono에는 terminal한 operator 대신 `.subscribe()`를 해주어 lazy하게 동작하도록 트리거할 수 있다.

### Publisher의 Subscribe 성질

Reactor에서는 Publisher - Subscriber 관계가 형성된 이후에는 아래 3개의 메서드 호출로 서로 통신을 하게 된다.
1. `onNext` - `Consumer<T>`
2. `onError` - `Consumer<Throwable>`
3. `onComplete` - `Runnable`

 - `onNext` 메서드는 subscribe했을 때 가장 먼저 호출하게 되는 메서드이자 파라미터이다.
 - `onError` 메서드는 시그널을 전달하는 과정에서 exception이 발생했을 때 호출하는 메서드이다.
 - `onComplete` 메서드는 데이터를 전부 전달하고 요청받은 작업을 전부 수행했을 때 실행하는 `Runnable`을 리턴한다.

```java
Mono<String> mono = Mono.just("ball");
```

```java
mono.subscribe();
```
위 코드는 아무것도 일어나지 않는 것처럼 보이지만 실제로 publisher가 subscriber에게 아이템을 전달한다.

```java
mono.subscribe(
	item -> System.out.println(item),  //onNext 에 전달될 consumer 함수
	err -> System.out.println(err.getMessage()), //onError에 전달된 consumer 함수
	() -> System.out.println("Completed") //onComplete에 전달된 Runnable
);
```
`subscribe()` 메서드들 통해 전달될 onNext, onError, onComplete 인자를 명세하는 코드이다.

```java
Mono<String> mono = Mono.just("ball")
	.map(String::length)
	.map(l -> l / 0);
	
mono.subscribe(
	item -> System.out.println(item),  //onNext 에 전달될 consumer 함수
	err -> System.out.println(err.getMessage()), //onError에 전달된 consumer 함수
	() -> System.out.println("Completed") //onComplete에 전달된 Runnable
);
```
onError를 명세하지 않으면 에러 메시지가 지저분하게 나타나지만, onError를 명세함으로써 아이템 emit 과정에서 발생한 에러 스택을 조금 더 상세하게 trace할 수 있다는 장점이 있다.

### Mono.empty

publisher 입장에서 데이터를 반환하고 싶지 않을때에는 어떻게 해야할까?

null을 사용하는 것은 NPE를 발생시킬 수 있기 때문에 좋은 선택이 아니다.

때문에 publisher는 반환해야 할 데이터가 없다는 것을 subscriber에게 전달하기 위해 더 나은 방법으로 전달해주어야 한다. 그 방식이 `Mono.empty()` 이다.

`Mono.empty()`는 에러가 아니다! 따라서 `onError` 도 발생시키지 않는다.
에러를 발생시키는 메서드는 `Mono.error()` 이다. `Mono.error()`는 `onError`를 발생시킨다.

### Mono.fromSupplier

`Mono.just()` 안에 객체를 리턴하는 함수를 인자로 넣으면 어떻게 될까?

앞에서 알아봤던 것처럼, Publisher는 lazy하게 동작하기 때문에 subscribe 될 때까지는 아무 행동도 하지 않을까? 그렇지 않다.

> Mono.just()는 데이터가 이미 존재하는 경우에만 사용해야 한다.

존재하지 않는 데이터로부터 Mono Publisher를 생성하려면, `Mono.fromSupplier`를 사용해야 한다.

`Mono.fromSupplier()`를 사용한다면 실제로 publisher가 subscribe되기 전까지는 예상하는 것처럼 의도대로 아무것도 실행되지 않는다.

### Mono.fromCallable

`Callable`은 자바5부터 도입된 인터페이스이고, `Supplier`와 같은 함수형 인터페이스는 자바8부터 본격적으로 도입이 되었다.

때문에 Reactor는 자바5의 `Callable`으로부터 생산되는 객체를 Mono Publisher로 만들어 주는 기능을 제공한다. 그것이 `fromCallable()`이다. 나머지 기능들은 전부 동일하다.

### Building vs Executing Pipeline

파이프라인을 생성하는 것과 실행하는 것은 다르다.

- 파이프라인을 생성하는 것은 쉽다.
- 하지만 파이프라인을 실행하는 것은 실제로 비즈니스 로직이 실행되는 경우 시간이 소모되는 작업일 수 있다.
- 실행될 비즈니스 로직은 항상 lazy하게 실행되도록 파이프라인을 구성해야 한다.
- Subscriber가 생겨서 실제로 파이프라인이 실행된다면 그제서야 비로소 시간이 걸리는 비즈니스 로직이 실행되는 구조로~

### Async

기본적으로 Reactor에서 무언가를 실행할 때에는 main 쓰레드에서 실행한다.

Mono Publisher에 대해 `.subscribeOn(Schedulers.boundedElastic())` 을 통해 Subscriber를 추가시키는 방식을 통해 Async + Non blocking하게 로직을 실행할 수 있다.

지금은 정확하게 어떤 일이 일어나는지 자세히 알지 않고 넘어간다.

이후의 강의에서 스케쥴러에 대해 알아보자.

### Block

리액티브 스트림에서는 모든 것이 비동기적으로 실행되지만 가끔은 동기적으로 무언가를 실행하고 싶은 경우가 있다. 그 경우에 `.block()`을 사용한다. 

실제로는 주어진 실행 환경의 쓰레드 자체를 block하는 방식으로 무언가를 구성하려 한다면 그것은 잘못된 구현이라고 보는 것이 맞다. 

> block()를 실제 환경에서는 사용하면 안된다. 

### Mono.fromFuture

자바 8의 비동기 지원 피쳐인 `CompletableFuture` 등으로부터 Mono Publisher를 생성하기 위해서는 `Mono.fromFuture()`를 사용할 수 있다.

`Mono.fromSupplier`와는 달리 별도의 쓰레드 환경에서 시그널이 전달되지 때문에 subscribe가 발생한 후에 실행 중인 main 쓰레드가 종료되었을 경우 결과가 유실될 수 있다.

### Mono.fromRunnable

지금까지 알아봤던 `fromCallable`, `fromSupplier`, `fromFuture` 등의 인터페이스는 어떤 파라미터를 받아서 값을 리턴하는 인터페이스였다.

때문에 Mono Publisher를 생성하는 과정에서는 생성된 값을 바탕으로 publisher가 데이터를 던져주었었는데, Runnable은 어떤 파라미터도 받지 않고 어떤 값도 리턴하지 않는다는 차이점이 있다.

만약 처리하는데 시간이 오래 소요되는 작업이 있고, 그 작업이 끝났을 때 알림을 받기 원하는 상황이라면? `Mono.fromRunnable`을 사용한다.

값을 리턴하지는 않기 때문에 `onNext`가 호출되지는 않는다.

## 정리

- `Mono` Publisher는 0개 또는 1개의 item을 emit하는 Reactor Publisher이다.
- `onComplete`나 `onError` 메서드 호출로 후속 처리를 수행할 수 있다.

타입 | 조건 | 사용할 메서드
--- | --- | ---
Mono 생성 | 데이터가 이미 존재하는 경우 | `Mono.just(data)`
Mono 생성 | 데이터가 생성되어야 하는 경우 | `Mono.fromSupplier(() -> getData())`, `Mono.fromCallable(() -> getData())`
Mono 생성 | 데이터가 async한 `CompletableFuture`로부터 오는 경우 | `Mono.fromFuture(future)`
Mono 생성 | `Runnable`이 완료된 이후에 `empty` emit이 필요한 경우 | `Mono.fromRunnable(runnable)`
Mono를 인자로 전달 | 함수가 `Mono<T>`를 인자로 받는데 리턴할 데이터가 없는 경우 | `Mono.empty()`
Mono를 리턴 | 함수가 Mono를 리턴해야 하는 경우 | `Mono.error(...)`, `Mono.empty()` + 상기 명시된 생성 타입들


