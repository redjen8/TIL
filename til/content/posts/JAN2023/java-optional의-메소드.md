---
title: "Java Optional의 메소드"
date: 2023-01-23T20:47:33+09:00
draft: false
author: redjen
---

Java 8부터 Java를 null safe하게 사용할 수 있게 해주는 Optional의 메소드에 대해 정리해봤다.

https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html

## 객체를 Optional로 래핑

`of`와 `ofNullable`을 사용한다.

- `of`는 지정된 null이 아닌 객체를 optional로 래핑한다.
- `ofNullable`은 지정된 null일 수도 있는 객체를 optional로 래핑한다.

때문에, 객체가 담고 있는 값이 non-null이라고 확신할 수 있는 상황이 아니라면 `ofNullable`을 사용하는 것이 옳다. 

## Optional Wrapper Check

Optional 객체가 담고 있는 값이 null인지 체크하고, 그에 따른 분기를 히기 위해서는 `isPresent`, `ifPresent`를 사용한다.

- `isPresent`는 Optional 래퍼 객체가 값을 가지고 있다면 true, null이라면 false를 반환한다.
- `ifPresent`는 Optional 래퍼 객체가 값을 가지고 있다면 파라미터로 입력된 `Consumer` 계산식을 평가한다. (함수형 프로그래밍) null이라면 아무것도 하지 않는다.

## Optional 안에 값이 없을 때의 처리

`orElse`, `orElseGet`, `orElseThrow` 를 사용한다.

- `orElse(someObject)`는 값이 존재한다면 리턴하고, 값이 존재하지 않는다면 `someObject`를 리턴한다.
- `orElseGet(someExpression)`은 값이 존재한다면 리턴하고, 값이 존재하지 않는다면 `someExpression`을 계산한 결과를 리턴한다.
- `orElseThrow(someThrowable)`는 값이 존재한다면 리턴하고, 값이 존재하지 않는다면 `someThrowable` 예외 객체를 throw한다.

## 번외 (메서드 네이밍에서 get과 find의 차이)

- get은 null 값이 없을 것으로 예상되는, 보다 빠른 데이터 read에서 사용한다.
- find는 null 값이 있을 수도 있는, 보다 느린 데이터 read에서 사용한다.