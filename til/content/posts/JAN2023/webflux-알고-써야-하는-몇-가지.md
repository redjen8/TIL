---
title: "Webflux 알고 써야 하는 몇 가지"
date: 2023-01-28T21:36:19+09:00
draft: false
author: redjen
---

PR 커멘트 받았던 내용 중 Webflux를 정말로 잘 사용하기 위해서는 알아야 할 것들에 대해 정리한 내용입니다. 

[공식 문서](https://projectreactor.io/docs/core/release/reference/#faq)

## `flatMap`은 컨텍스트 스위칭을 야기할 수 있다

https://techblog.woowahan.com/2673/

위 글에서도 언급되었듯 `flatMap`은 단순히 데이터의 depth를 flatten하는 용도로 사용되는 것뿐만 아니라 `map`과는 달리 asynchronous하게 함수를 적용시킨다.

즉 `flatMap`을 사용하게 되면 시그널을 다른 event loop group에 넘기는 과정에서 컨텍스트 스위칭 비용을 요구한다. 

또한 `flatMap` 사용의 depth가 늘어나는 것도 주의해야 한다. inner flatMap 사용은 의도하지 않은 동작을 유발시킬 가능성이 있다.

때문에 단순히 객체의 값을 수정하거나 타입을 변환할 때 항상 `flatMap`을 통해 해결하려 하는 접근은 성능을 떨어뜨리고, webflux를 제대로 사용하지 않는 방식이다.

## `Mono.zip`은 롤백이 필요한 트랜잭션 상황에서 정상적으로 작동하지 않는 경우가 있다

우선, `Mono.zip`은 인자에 들어간 `Mono` 작업들이 완료되면 하나의 `Tuple` 객체로 묶어서 체이닝할 수 있도록 리턴해주는 메서드이다.

`Mono.zip`은 각각의 `Mono` 작업이 병렬적으로 처리되도록 할 수 있기 때문에 병렬처리에서 얻을 수 있는 이득이 크다면 유용하다. 반면 병렬 처리에서 얻을 수 있는 이득이 적다면 굳이 사용할 필요가 없다. `Tuples.of()`를 사용하는 것이 낫다.

그런데 `transactional` 처리 시 첫 실행에 `Mono.zip`의 사용이 context를 잃어버리게 할 수 있다는 리포트가 있는 것 같다. 

https://github.com/spring-projects/spring-framework/issues/28782

`transactional` 사용 시에는
- 첫 실행에서 `Mono.zip`을 사용하지 않고
- transaction number를 최초 데이터 처리에서 받아
- context에서 가질 수 있도록 하는 것이 좋다. 

## 결론

나는 `flatMap`으로 펼친 데이터를 다시 `Mono.zip`으로 묶어서 리턴하였는데, 지금 생각해보면 유의해야 할 점만 모은 방법으로 문제를 해결했던 것이었다.. ㅋㅋ

항상 코드를 작성할 때에는 내 코드가 하는 일에 대해 명확하게 인지한 상태에서 작성하는 것이 중요한 것 같다.

그래서 webflux가 너무 어렵다.