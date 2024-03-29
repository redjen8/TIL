---
title: "모던 자바스크립트 Deep Dive 42장 - 비동기 프로그래밍"
date: 2023-11-14T23:33:59+09:00
draft: false
author: redjen
---

### 동기 처리와 비동기 처리

이전에 정리했던 [함수 실행 컨텍스트](https://redjen8.github.io/posts/js-deepdive/execution-context/)내용 참고..

1. 함수 호출
2. 함수 코드 평가 (eval)
3. 함수 실행 컨텍스트 생성
4. 생성된 컨텍스트를 실행 컨텍스트 스택에 push
5. 함수 코드가 실행
6. 함수 코드 실행 종료 시 해당 실행 컨텍스트가 스택에서 pop

> 즉 함수가 실행되기 위해서는 함수 실행 컨텍스트가 실행 컨텍스트 스택에 푸시되어야 한다.

그리고 **자바스크립트 엔진은 단 하나의 실행 컨텍스트 스택을 가진다**.  (동시에 두 개 이상의 함수를 실행할 수 없다)

동기, 비동기, 블로킹, 논블로킹의 차이는 매우 중요하기 때문에 아래에서 알아보고 가자.

#### 동기 / 비동기

동기 / 비동기는 **호출하는 함수와 호출된 함수의 관심사**에 차이가 있다.

- 동기 (synchronous): 호출하는 함수가 호출된 함수의 완료 상태를 신경 쓴다
- 비동기 (asynchronous): 호출하는 함수가 호출된 함수의 완료 상태를 신경쓰지 않는다.
#### 블로킹 / 논블로킹

블로킹 / 논블로킹은 **호출된 함수의 제어권**에 차이가 있다.

- 블로킹 (blocking): 호출된 함수가 작업 종료될 때까지 제어권을 가진다. 호출한 함수는 그 동안 다른 작업을 진행할 수 없다.
	- 호출된 함수는 작업에 대한 결과를 작업이 끝날 때 반환한다.
- 논블로킹 (nonblocking): 호출된 함수가 작업 종료되지 않아도 제어권을 호출한 함수로 넘겨줄 수 있다.
	- 호출된 함수는 작업에 대한 결과를 작업이 끝나지 않아도 반환할 수 있다 (아직 완료 안됐음을 리턴)

#### 4가지 각 조합 별 특징

https://velog.io/@redjen/java-reactive-programming-1-Introduction

##### 커널 레벨에서 네트워크 통신이 실제로 어떻게 이뤄질까?

- [운영체제, File Descriptor를 다루는 IO 모델들](https://black7375.tistory.com/90#i/o-%EB%AA%A8%EB%8D%B8%EB%93%A4)
- [IO Multiplexing을 다루는 함수들](https://nima101.github.io/io_multiplexing)
- [IO Multiplexing을 사용해 저수준 다중 접속 서버 만들기](https://jongmin92.github.io/2019/02/28/Java/java-with-non-blocking-io/)

#### 자바스크립트로 돌아와서

> 자바스크립트는 싱글 쓰레드로 동작한다.  

멀티 쓰레드로 동작하면 더 좋은 거 아닌가? **왜 자바스크립트는 싱글 쓰레드로 동작하게 만들었을까**?
- 멀티 쓰레드로 동작하기 위해서는 많은 것들을 신경써야 한다.
	- 공유 자원으로 인한 동시성 문제와 그를 해결하기 위한 락의 구현이 필수적
	- 이런 것들을 고려하지 않아도 되는 싱글 쓰레드 기반 언어는 구현하기 무척 쉬워진다는 장점
- JS는 웹 브라우저 내에서 스크립팅을 위해 고도화된 언어
	- 웹 브라우저는 동시에 많은 사용자를 고려하지 않아도 되는 클라이언트 프로그램

자바스크립트는 싱글 쓰레드로 동작하기 때문에, 동시에 한 가지의 일 밖에 처리하지 못한다.
- 껌 씹으면서 횡단보도 못 건넘
- 껌 다 씹고 횡단보도 건너야
- 만약 동시에 하고 싶다면?
	- HTTP 요청을 통해 서버로부터 데이터를 가지고 오는 동시에 렌더링을 한다던가
	- HTML 요소가 막 애니메이션으로 움직이는 동시에 사용자 이벤트를 처리한다던가

### 이벤트 루프와 태스크 큐

대부분의 JS 엔진 (V8로 대표되는)은 크게 두 가지 영역으로 구분된다
- 콜 스택
	- 실행 컨텍스트 스택
	- 소스 코드가 평가되면서 생성된 실행 컨텍스트가 스택에 막 추가된다
- 힙
	- 객체가 저장되는 메모리 공간

> JS 엔진은 태스크 요청 시 콜 스택에 있는 작업을 순차적으로 처리만 한다. 

즉 비동기 처리에서 소스코드의 평가, 실행을 제외한 모든 처리는 JS 엔진을 구동하는 환경에서 담당한다.
- `setTimeout` 콜백 함수의 평가와 실행: JS 엔진이 수행
- 호출 스케줄링을 위한 타이머 설정, 콜백 함수 등록: JS엔진이 구동되는 환경인 브라우저 또는 Node에서

브라우저(또는 Node)에서는 JS 엔진이 넘겨준 잡다한 일들을 수행하기 위해 다음의 환경을 제공한다.
- 태스크 큐: 비동기 함수의 콜백 함수 또는 이벤트 핸들러가 잠시 보관되는 영역
	- 프로미스의 콜백 함수가 잠시 보관되는 마이크로 태스크 큐는 여기선 잠시 논외로..
	- https://262.ecma-international.org/6.0/#sec-promise-constructor
	- https://chromium.googlesource.com/v8/v8.git/+/refs/tags/7.4.136/src/microtask-queue.cc
- 이벤트 루프: 다음의 일을 수행하는 아주 중요한 녀석
	- 콜 스택에 현재 실행 중인 실행 컨텍스트가 있는지
	- 태스크 큐에 대기 중인 함수가 있는지
	- 계속 반복적으로 확인하다가
		- 콜 스택이 비어 있고 && 태스크 큐에 대기 중인 함수가 있다면
		- 태스크 큐에 대기 중인 함수를 콜 스택으로 이동시킨다
		- 이렇게 이동된 함수는 곧 실행된다
	- 즉 **태스크 큐에 잠시 보관되는 함수는 비동기 처리 방식으로 동작한다.**

#### 예시 코드의 실행 방식을 보자

```js
function foo() {
	console.log('foo');
}

function bar() {
	console.log('bar');
}

setTimeout(foo, 0);
bar();
```

1. 전역 코드 실행 시점에 `setTimeout` 함수 호출
2. 콜백 함수(`foo`)를 호출 스케줄링하고 종료되어 콜 스택에서 pop
	1. 타이머는 누가 재냐? --> 브라우저
	2. 콜백 함수를 태스크 큐에 넣는 놈은 누구냐? --> 브라우저
3. 다음의 일이 병렬로 처리된다
	1. 브라우저: 타이머 끝날때까지 기다린다. 타이머 땡하면 `foo` 함수가 태스크 큐에 푸시
		1. 푸시된 함수는 태스크 큐에 있기 때문에 콜 스택이 차 있다면 비워지는 걸 계속 바라만 보고 있어야 한다
		2. 때문에 `setTimeout`을 사용해서는 정확히 delay 만큼 기다린 이후에 수행되는 것을 절대 보장할 수 없는 구조
	2. `bar` 함수가 호출되어 실행 컨텍스트 생성 및 콜 스택에 푸시된다. 종료 된다면 다시 pop된다
4. 전역 코드 실행 종료, 전역 실행 컨텍스트가 콜 스택에서 pop 된다
5. 이벤트 루프: 콜 스택 비어 있으니까 이제 태스크 큐 꺼내야지
	1. 대기 중인 콜백함수 `foo`가 이제서야 콜 스택에 들어가서 실행된다

> JS 엔진은 싱글 쓰레드로 동작하지만, JS 엔진이 수행되는 환경인 브라우저, Node는 멀티 쓰레드로 동작한다.

### (참고 자료) 이벤트 루프의 동작 방식에 대해

- [브라우저 이벤트 루프에 대한 더 깊은 이해](https://github.com/atotic/event-loop)
- [Node.js 이벤트 루프에 대한 더 깊은 이해](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
	- [libuv 시스템 디자인](https://docs.libuv.org/en/v1.x/design.html)
	- [libuv loop 코드](https://github.com/libuv/libuv/blob/v1.x/src/unix/loop.c)
- [스프링 웹플럭스에서의 동시성 처리](https://www.baeldung.com/spring-webflux-concurrency)
- [vert.x에서의 이벤트 루프](https://vertx.io/docs/vertx-core/java/)
