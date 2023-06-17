---
title: "Rxjs를 소개합니다"
date: 2023-06-12T22:07:17+09:00
draft: true
---

## Reactive Streams와 ReactiveX에 대해

[ReactiveX의 공식문서](https://reactivex.io/)에서는 `Observable`한 시퀀스를 사용해 비동기적인, 이벤트 기반의 프로그램들을 다루기 위한 라이브러리로써 ReactiveX로 소개한다.

ReactiveX는 옵저버 패턴을 적용해서
- 데이터나 이벤트의 시퀀스들을 지원하거나
- 시퀀스들을 선언적으로 다룰 수 있는 연산자들을 지원하면서도
- 로우 레벨에서 이뤄지는 쓰레드 제어, 동기화, thread safety, 동시성을 지원하는 자료구조, 논블락킹 IO에 대해 신경쓰지 않게 해준다. (개인적으로는 이게 가장 큰 장점이라고 생각한다)

ReactiveX의 `Observable`은 여러 아이템의 비동기적인 시퀀스를 다룰 수 있는 이상적인 방법을 제공한다.

구분 | 단일 아이템 | 복수 아이템
--- | --- | ---
동기 | `T getData()` | `Iterable<T> getData()`
비동기 | `Future<T> getData()` | `Observable<T> getData()`

ReactiveX 라이브러리들은 가끔 함수형 반응형 프로그래밍으로써 불리지만, 이건 잘못된 명명이라고 소개한다.

> ReactiveX may be functional, and it may be reactive, but “functional reactive programming” is a different animal. One main point of difference is that functional reactive programming operates on values that change continuously over time, while ReactiveX operates on discrete values that are emitted over time.

> ReactiveX는 함수형일수도 있고, 반응형일수도 있지만, '함수형 반응형 프로그래밍'은 전혀 다른 종류의 것이다. 한 가지 큰 차이점은 함수형 반응형 프로그래밍은 시간에 따라 지속적으로 변화하는 값을 다루지만, ReactiveX는 시간에 따라 방출되는 이산 값에 대해 작동한다는 것이다.

ReactiveX의 `Observable`은 Java의 `Future`처럼 단일 스칼라 값의 emission이 아니라 값의 시퀀스 또는 무한한 스트림을 다룬다. `Observable`은 여러 유즈 케이스를 다룰 수 있도록 설계된 단일 추상화이다. 

### Iterator 패턴이 어떻게 적용되었나

> `Observable`은 비동기 / 푸시 방식을 활용하고, `Iterable`은 동기 / 풀 방식을 활용한다.

이벤트 | Iterable (pull) | Observable (push)
--- | --- | ---
데이터 수신 | `T next()` | `onNext(T)`
에러 처리 | throw `Exception` | `onError(Exception)`
완료 처리 | `!hasNext()` | `onCompleted()`

### 리액티브 프로그래밍의 장점

ReactiveX는 이런 `Observable`들을 필터링하고, 선택하고, 변화시키고, 합칠 수 있는 연산자를 제공한다.

Iterator 패턴에서 컨슈머가 프로듀서로부터 값을 풀하는 것과 반대로 `Observable`은 프로듀서가 값이 준비되자 마자 컨슈머에게 값을 밀어넣는 방식으로 동작한다.

`Observable` 타입은 GoF의 옵저버 패턴에 존재하지 않는 두가지 의미를 부여한다.

1. 프로듀셔가 더 이상 사용할 수 없는 데이터가 없는 상태임을 컨슈머에게 알려줄 수 있다.
2. 프로듀서가 컨슈머에게 값을 전달하던 중 오류가 발생했음를 알릴 수 있다. (Iterable은 iteration 도중 에러가 발생하면 Exception을 던지지만, Observable은 옵저버의 `onError` 메서드를 호출한다)

RxJava, RxJS, 

https://yozm.wishket.com/magazine/detail/1753/

https://angular.io/guide/rx-library

https://rafasiqueira.medium.com/rxjs-real-world-examples-part-1-4b0b7562ac64

https://blog-ko.superb-ai.com/nestjs-interceptor-and-lifecycle/
