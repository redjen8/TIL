---
title: "Webflux Scheduler의 종류"
date: 2023-01-19T19:24:44+09:00
draft: false
author: redjen
---

Webflux에서는 `parallel()` 기능을 통해 `Flux` 등을 `ParallelFlux` 로 변환하여 쓰레드 간 병렬적으로 작업을 할 수 있도록 지원한다.
`runOn(someScheduler)`는 어떤 역할을 하는지 찾아봤다.

https://godekdls.github.io/Reactor%20Core/reactorcorefeatures/#45-threading-and-schedulers

[3번 봤는데, 앞으로 297번 더 봐야할 정리 글](https://www.baeldung.com/spring-webflux-concurrency)

## Webflux의 Threading 모델

Webflux 기반 웹 어플리케이션에서 임의의 쓰레드를 생성하여 `Mono`의 처리를 맡기면 어떨까?
연산자의 대부분은 이전 연산자를 실행했던 쓰레드에서 작업을 계속해 나간다.

개발자가 별도로 작업할 쓰레드를 지정하지않았다면 최상단의 연산자는 `subscribe()`를 호출한 쓰레드에서 실행한다.

즉, Controller - Service - Repository - Service - Controller로 이어지는 데이터 흐름에서, Controller를 통해 이벤트 루프가 지정한 워커 쓰레드가 Repository에서 데이터를 조회하는 메서드를 실행하게 되는 흐름이다.
> 새 쓰레드를 생성하더라도 `subscribe()`를 호출하는 쓰레드가 다른 쓰레드라면 모든 콜백함수는 `subscribe()`를 호출한 쓰레드에서 실행된다.

## Webflux의 Scheduler 모델

Webflux에서 코드 실행 모델과 위치는 어떤 Scheduler를 사용했는지에 따라 달라진다.

### Schedulers.immediate()
실행 컨텍스트가 없는 경우이다. 제출한 `Runnable`을 즉시 실행하는 모델이다.

사실상 현재 쓰레드에서 실행하기 때문에 아무 일도 하지 않는 스케쥴러와 같다.

### Schedulers.single()
재사용할 수 있는 단일 쓰레드 모델이다. 스케쥴러를 폐기하기 전까지 여러 번 호출해도 이전에 사용했던, 동일한 쓰레드를 재사용한다.

호출할 때마다 전용 쓰레드를 사용해야 할 필요가 있다면 이 스케쥴러를 사용한다.

### Schedulers.elastic()

Unbounded 쓰레드 풀을 사용하는 모델이다. backpressure에 대한 이슈를 감추고 너무 많은 쓰레드를 사용하기 때문에 잘 사용하지 않는다. (`elastic()`은 풀에 생성할 수 있는 쓰레드 수를 제한하지 않는다) 

이 스케쥴러를 사용할 일이 있다면 대신 `boundedElastic`을 사용한다.

### Schedulers.boundedElastic()

Bounded 쓰레드 풀을 사용하는 모델이다. 이전의 `elastic()`처럼 필요한 경우 새 워커 풀을 만들거나 풀에서 여유 있는 쓰레드가 있다면 재사용한다. 

이 워커 풀들은 관리된다. 너무 오랫동안 (default 60초) 유휴상태에 있는 워커 풀이 있다면 폐기한다.

`elastic()`과는 달리 풀에 생성할 수 있는 쓰레드 개수를 제한한다. (default CPU core # * 10) 쓰레드 개수가 한계점에 다다른 이후에 제출한 작업들은 100,000개까지 큐에 담고 쓰레드에 여유가 생기면 다시 스케쥴링한다. 

보통 블로킹 I/O가 필요한 상황에 사용한다. `boundedElastic()`을 사용하면 블로킹 프로세스에 대한 별도 쓰레드를 할당할 수 있기 때문에 다른 리소스들이 블로킹 로직에 영향을 받지 않도록 할 수 있다.

다만 `boundedElastic()`은 별도의 워커 풀들을 생성해서 작업을 할당시키는 것이기 때문에 너무 많은 쓰레드를 사용할 경우 시스템에 무리를 줄 수 있다. 따라서 사용 시 주의를 요한다.

### Schedulers.parallel()

고정 크기의 워커 풀을 생성하여 작업을 병렬적으로 실행하는 모델이다. (default CPU Core #)

### 보다 다양한 스케쥴러 인스턴스 사용

`Schedulers.newParallel(newSchedulerClass)` 은 `newSchedulerClass`라는 이름의 새 병렬 스케쥴러를 생성한다. 경우에 따라 다양한 스케쥴러 인스턴스를 커스터마이징할 수 있으나 시스템 리소스인 쓰레드 / 워커 풀을 직접 제어하는 것이기 때문에 주의를 요한다. 

