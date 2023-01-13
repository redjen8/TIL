---
title: "Mono의 여러가지 생성방법"
date: 2023-01-14T00:01:54+09:00
draft: false
author: redjen
---

Mono의 몇 가지 instantiation 방법들 간에는 어떤 차이가 있을까?

https://stackoverflow.com/questions/56115447/mono-defer-vs-mono-create-vs-mono-just

## Mono.just()

```Mono.just```는 Mono를 생성하는 가장 primitive한 방법이다. 객체를 그냥 Mono로 감싸서 subscriber에게 downstream으로 전달한다.

## Mono.defer()

```Mono.defer(supplier)```는 Mono instance를 supply하는 전체 expression을 제공한다. 이 expression은 누군가가 subscribe할 때까지는 defer된다. Mono.defer 안에는 ```Mono.error(throwable)```과 같은 제어를 넣어서 컨트롤할 수 있다.
```Mono.just```는 이와 같은 행동을 지원하지 않는다.

## Mono.create()

```Mono.create()```는 값을 전달하는 과정을 온전히 컨트롤할 수 있는 가장 advanced한 방법이다. Mono 인스턴스를 콜백으로 리턴하는 대신에 ```MonoSink.success()```, ```MonoSink.success(value)```, ```MonoSink.error(throwable)```메서드를 통해서 값을 전달할 수 있는  ```MonoSink<T>```객체를 사용한다. ```Mono.create()```의 문서는 아래 링크를 참고해야 겠다.

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#create-java.util.function.Consumer-

## 그래서 뭘 써야 할까?

간단하게 Mono 객체를 downstream에 내려줘야 할 때에는 ```Mono.just()```로도 충분하다.
그러나, 그 Mono 인스턴스를 생성하는 과정에서 에러를 graceful하게 핸들링하고 싶다면 ```Mono.defer()```를, 값 전달 과정에 있어서 더 강력한 컨트롤이 필요하다면 ```Mono.create()```를 사용하는 것이 바람직하다고 느껴진다. 

또한, ```Mono.just()```와 나머지 두 개에는 큰 차이가 있다.
값이 만들어진 이후에 Mono로 뿌리는 just와는 다르게, create와 defer는 값이 준비되지 않은 상태에서도 subscriber가 있다면 실행이 된다.
실행 흐름에 있어서 미묘한 차이가 있는 것에 유의해야 한다. 