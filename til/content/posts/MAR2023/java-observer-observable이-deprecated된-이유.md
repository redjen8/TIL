---
title: "Java Observer Observable이 Deprecated된 이유"
date: 2023-03-03T18:45:09+09:00
draft: false
author: redjen
---

학부 설계 패턴 수업에서는 GoF 디자인 패턴들에 대해 배웠었다.

기억에 남는 디자인 패턴에는 여러 가지가 있지만, 지금 가장 많이 사용하는 디자인 패턴이라고 한다면 아마 옵저버 패턴이 아닐까? 일단 ReactiveX를 기반으로 한 웹플럭스를 쓰고 있으니..

그런데 옵저버 패턴을 실습하던 시간에 `Observer` 인터페이스와 `Observable` 클래스가 deprecated 되어 있었다는 점이 떠올랐다.

아니? 뭐야, 우린 왜 deprecated된 클래스를 사용하면서 실습을 하고 있는거지? 하는 찝찝함이 들었지만 일단 수업을 들었던 것이 기억난다.

그리고 졸업을 하고 나서야 왜 deprecated 되었는지에 대한 이유를 찾을 수 있었다.

https://dzone.com/articles/javas-observer-and-observable-are-deprecated-in-jd

## Observable의 역사

Java의 `Observable` 클래스는 Java 1.0 부터 역사를 같이 해 온 할아버지 인터페이스이다.

JDK9에 와서는 Deprecated 되기로 결정되었다. 그 사이에 어떤 일들이 있었던 걸까?

그런데 `Observable`의 deprecation은 조금 다르다.. 언제부터 deprecated 될지에 대한 `since()` 영역은 표시되어 있지만 언제부터 실제로 클래스가 제거될지에 대한 `forRemoval()` 영역이 비어 있다.

즉 deprecated 되었지만 실제로 `Observable` 클래스가 삭제될 일은 없는 것이다. 오랜 시간 자바와 세월을 같이 해 온 용사에 대한 예우일까? 

## Observable을 사용하지 말아야 하는 이유

`Observable` 클래스가 deprecated 될 때 작성되었던 문서에는 다음과 같이 적혀있다.
- `Observer`와 `Observable`에 의해 지원되는 이벤트 모델은 상당히 한정될 수 밖에 없다.
- `Observable`에 의해 전달되는 알림의 순서는 특정될 수 없다.
- 상태 변화 또한 알림에 일대일 매칭된다고 보장할 수 없다.
- `java.beans` 같은 패키지에는 보다 풍부한 이벤트 모델을 사용할 수 있도록 되어 있고
- 쓰레드 간 순서가 보장되는 메시징에는 `java.util.concurrent` 패키지의 자료 구조를 사용하는 것이 합리적이다.
- 리액티브 스트림 스타일 프로그래밍은 Flow API를 사용할 수 있다.

즉, 초창기 자바에는 별다른 이벤트 모델이 많이 없었기 때문에 `Observable` 클래스의 존재 이유가 있었지만 시간이 흐르고 보다 복잡한 이벤트의 처리를 어플리케이션 내에서 처리하기 시작했던 것이 화두였다. 

> `Observable` 클래스는 현대의 복잡한 이벤트 모델을 처리하기엔 부족한 면이 너무 많았다. `Observable` 클래스가 가진 기능을 더하기엔 다른 패키지들에서 관련 기능을 아주 잘 제공하고 있었다.

## Deprecated 되었지만, 왜 없어지지는 않을까?

앞서 농담조로 왜 deprecated 처리한 `Observable`을 없애지 않는지 궁금했었는데,
- `Observable` 과 `Observer`는 현재 사용되고 있는 곳이 많지 않다.
- 때문에 deprecation이 큰 문제가 되지 않는다.
- 때문에 구체화된 계획을 가지고 이 클래스를 굳이 제거하지 않아도 된다. 

라는 결론에 다다른 것으로 보인다.