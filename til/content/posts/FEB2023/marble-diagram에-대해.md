---
title: "Marble Diagram에 대해"
date: 2023-02-17T20:18:22+09:00
draft: false
author: redjen
---

Reactive Stream 공식 문서에 자주 나오는 구슬 같이 생긴 친구들의 정체는 뭘까?

https://www.oreilly.com/library/view/reactive-programming-for/9781785882883/ch03s02.html

## Marble Diagram

ReactiveX 프로젝트는 Observable한 스트림을 통한 비동기적 프로그래밍 API이다.
그리고 Observable과 비동기적 프로그래밍의 흐름을 보다 이해하기 쉽게 구성해 커뮤니케이션과 전달을 효율적으로 할 수 있게 해주는 그림이 바로 마블 다이어그램이다.

> Observable 아이템들을 시간 순으로, 그래픽으로 나타내는 것이 마블 다이어그램이다.

마블 다이어그램은 또한 `merge`, `delay`, `scan`과 같은 Observable한 시퀀스들에 대한 연산자들을 표현하기 위한 '사실상' 표준이다.

## 구성 요소들

https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5

마블 다이어그램에서 주의 깊게 봐야할 점으로는
- 시간 순으로 이루어지는 흐름 (사실상 Stream의 특징이기도 하다)
- 완료 신호에 따른 분기 (`onComplete()` 콜)
- 그리고 각각의 타입이 어떻게 변하는지

각각의 언어에 적용되는 디테일은 다르지만, ReactiveX 프로젝트 진영에서 추구하는 철학 자체는 동일하다 보니 마블 다이어그램이라는 동일한 도표로 플로우를 나타낼 수 있는 장점이 있다고 생각한다.

나중에 RxJS를 좀 더 본격적으로 사용하게 된다면 `Observable`에 대해 좀 더 정리해봐야겠다.