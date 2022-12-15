---
title: "왜 Python은 GIL을 사용할까?"
date: 2022-12-15T18:00:28+09:00
draft: false
author : redjen
---

# 왜 Python은 GIL을 사용할까? 

[원문](https://softwareengineering.stackexchange.com/questions/186889/why-was-python-written-with-the-gil)

## 들어가기 전에 

파이썬은 사실상 언어라기 보단 명세서에 가깝다.
여러 파이썬 구현체 중에서 가장 널리 사용되는 것은 CPython이고 그밖에는 IronPython(.NET 기반), Jython(JVM 위에서 동작하는 버전), PyPy (JIT 컴파일러를 활용한 버전) 등등이 있다.

## GIL이란

가장 많이 사용되는 구현체인 CPython에서 사용하는 Global Interpreter Lock의 줄임말이다. 
파이썬 인터프리터가 한 스레드당 하나의 바이트코드를 실행시킬 수 있도록 해주는 Lock
하나의 쓰레드에 모든 리소스를 허락하고, 다른 쓰레드는 접근하지 못하도록 막는 셈

## GIL을 왜 쓰는지? 

파이썬은 GC와 Reference Counting을 통해 메모리를 관리한다.
파이썬에서는 모든 것이 객체인데, 모든 객체는 자신에 대한 Ref Count를 저장한다
Ref Count가 0이 된 객체들이 GC의 대상이 되는 방식으로 메모리가 관리됨
이 Ref Count에 대한 관리가 쓰레드 세이프하지 않기 때문에 멀티 쓰레드 환경에서 Ref Count에 대한 경쟁 상태 발생하고, GC를 제대로 활용할 수 없게 된다.

안타깝게도 GIL이 보장해주는 쓰레드 세이프한 환경에 의존해서 파이썬 생태계가 이미 구축되었기 때문에,
향후에도 GIL을 없애기 위한 시도는 계속되어 질 것 같지만 어려운 과제들을 해결해야 하기에.. 당분간은 없어지지 않을 것 같다.

## GIL의 장단점

독특한 메모리 관리 구조 때문에 생겨나게 된 GIL은 사용했을 때 다음과 같은 장점들이 있다.
1. 싱글 쓰레드 연산에서 더 빠르다
2. I/O에 의존하는 멀티 쓰레드 환경에서 더 빠르다
3. CPU에 의존하는, C 라이브러리의 연산이 많은 작업에 대해 멀티 쓰레드 환경에서 더 빠르다
4. C 익스텐션을 개발하기 더 쉽게 한다. (익스텐션 개발 시 쓰레드 세이프함을 고려할 필요가 없어짐)

## 마치며

GIL은 C 익스텐션에 의해 풀릴 수 있다.
[asyncio가 동작하는 방식](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work) 을 찾아봤는데 흥미로웠다.. 추가로 작성할 여지가 있으므로 다음에 또 찾아봐야지

