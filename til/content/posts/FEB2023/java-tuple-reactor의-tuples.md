---
title: "Java Tuple, Reactor의 Tuples"
date: 2023-02-27T21:51:31+09:00
draft: false
author: redjen
---

2개의 데이터를 가지는 원소를 묶어서 하나의 변수 인스턴스로 나타내고 싶을 경우가 있다.
대표적인 예시로는 2차원 상의 좌표를 나타내는 예시가 있을 것이다.

좌표를 `x`, `y` 라는 두 개의 서로 다른 변수 데이터로 나타내는 것보다
`Pair<Double, Double> coordinate` 로 나타내는 것이 훨씬 깔끔하고, '좌표' 라는 데이터를 표현하기 직관적이다.

다른 언어와 마찬가지로, 자바에서도 이러한 형태의 데이터 묶음을 제공한다.

https://www.geeksforgeeks.org/pair-class-in-java-tuples/

## Pair

JavaTuples 라이브러리에는 2개의 원소를 묶을 수 있는 `Pair`가 존재한다.

이 `Pair`는 몇 가지 특징을 가진다.
- Typesafe 하다.
- Immutable 하다.
- Iterable하다.
- Serializable 하다.
- Comparable 하다. (`Comparable<Tuple>`의 구현체이다)
- `equal()`과 `hashCode()`를 구현한다.
- `toString()`을 구현한다.

즉, `Pair` 자체만으로도 객체의 거의 모든 것을 사용할 수 있도록 준비를 다 해놓은 상태인 것이다.

### 인스턴스 생성

1. `new Pair<T, T>(v1, v2);`로 생성
2. `Pair.with()` 로 생성
3. `Pair.fromCollection()`으로 생성 (컬렉션에서 생성 시)

어떤 것을 사용해도 상관 없지만. static한 메서드를 사용하는 것이 더 깔끔해 보이긴 한다.

### 값 꺼내기

- `x.getValue0()`으로 첫 번째 원소를
- `x.getValue1()`으로 두 번째 원소를 꺼낼 수 있다.

### 값 설정

- `x.setAt0()`으로 첫 번째 원소를
- `x.setAt1()`으로 두 번째 원소를 설정할 수 있다.

> 앗.. 여기서 잠깐 헷갈렸다. 분명 Tuple은 Immutable 하다고 하지 않았나? 왜 값 변경이 가능할까?

Mutable한 대표적인 컬렉션에는 `List`가 있다. `List`는 가변 길이의 원소를 저장할 수 있다.
- 하지만 `Tuples` 는 그렇지 않다. 미리 정해진 원소 개수만큼만 저장할 수 있다는 점이 다르다.
- 때문에 `Tuples`가 Immutable하다고 한다.

말 나온 김에 `List`와의 차이점을 정리하자면,

https://www.upgrad.com/blog/list-vs-tuple/

- `Tuple`은 `List`보다 iteration이 빠르고, 메모리 소모도 적다.
	- 하지만 `Tuple`은 `List`보다 내장 메서드들이 적다.
- `List`에서는 삽입과 삭제가 보다 잘 수행된다.
	- `Tuple`은 대신 읽기 연산에 더 유리하다.

### 값 검색

`x.contains()` 를 통해서 값이 존재하는지 찾을 수 있다. 

### 값 삽입

원하는 위치에 원소를 삽입하는 것은 `x.addAt0()`, `x.addAt1()`... 을 통해 할 수 있다.

## 다른 Tuple 구현체

`Pair`는 원소의 개수가 두 개인 `Tuple` 구현체이다.
`Pair`에 값을 더하면 원소의 개수가 3개가 되고, 이는 `Triplet`이라고 부른다.

https://www.wpr.org/what-s-difference-between-quintet-septet-and-octet

- 2개면 `Pair`
- 3개면 `Triplet`
- 4개면 `Quartet`
- 5개면 `Quintet`
- 6개면 `sextet`
- 7개면 `septet`
- 8개면 `octet`
- 9개면 `nonet`
- 10개면 `dectet`

## Reactor의 Tuples

Webflux를 사용하다 보면 이전 파이프라인의 결과 값을 다음 파이프라인에 넘겨주고 싶은 경우가 있다.

데이터 aggregation의 결과 값을 이후 파이프라인에서 사용해야 하는데, 당장은 필요 없는 경우도 사용할 수 있는 것이 `Tuples` 이다.

https://projectreactor.io/docs/core/release/api/index.html?reactor/util/function/Tuples.html

Reactor의 `Tuples`는 다음 특징들을 가진다.
- immutable 하다.
- 임의의 타입을 가지는 객체의 Collection이다.

### Java Tuple과의 차이점

원소의 개수에 따라 Java Tuple은 부르는 명칭이 다 달라졌다. `septet`, `quartet` 등..

Reactor의 `Tuples`는 원소의 개수를 단순히 `Tuple` 뒤에 명시한다.

> Java `Tuple`과 Reactor의 `Tuples`, 어떤 걸 언제 써야 할까?

`Pair`를 쓰는 경우로 이전에 '2차원 공간에서의 좌표를 나타날 때' 를 말했었는데, 데이터가 확실하게 쌍을 이루는 경우에는 `Pair`를 쓰는 것이 좋다고 느껴진다. 

반면 reactor의 `Tuples`는 단순히 데이터의 컬렉션을 나타낸다. 다음 파이프라인에서 소비되길 바라는 데이터의 집합체이다. 

때문에 Webflux를 사용한다면 (애초에 Webflux를 사용하지 않으면 선택지가 Java Tuple 밖에 없다)
- 2차원 공간의 좌표계처럼 특정 데이터의 쌍을 나타날 때에는 `Pair`와 같은 Java Tuple을 쓴다.
- 이외의 모든 경우에서는 reactor의 `Tuples`를 사용하는 것이 좋다고 생각한다.