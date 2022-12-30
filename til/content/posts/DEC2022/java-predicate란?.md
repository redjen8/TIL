---
title: "Java Predicate란?"
date: 2022-12-30T16:49:27+09:00
draft: false
author: redjen
---

# Java Predicate란?

## 함수형 인터페이스

추상 메서드가 단 하나인 인터페이스.
추상 메서드가 두 개 이상이 되면 어떤 메서드를 실행해야 할 지 모르기 때문에 오류 발생
```@FunctionalInterface``` 어노테이션으로 함수형 인터페이스 임을 나타낼 수 있다. 없어도 상관 없지만, ```@Override``` 를 붙이는 이유랑 비슷한듯

## Predicate

test(Object) 메서드를 가지는 함수형 인터페이스인 Predicate는 한 인수의 참/거짓을 반환하도록 제공해준다.

자바에서는 모든 것이 객체.
비교가 되는 기준 자체를 객체로 만든다고 생각하면 될 것 같다.

Java의 stream API에서는 filter() 메소드를 제공하는데,
Predicate 인터페이스는 이 filter() 메소드에 인자로 들어갈 수 있다. (또는 람다식을 넣어도 된다)
