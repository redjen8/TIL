---
title: "Typescript Keyof Operator"
date: 2023-05-01T20:49:52+09:00
draft: false
author: redjen
---

타입스크립트의 `keyof` 연산자는 무엇이고 언제, 어떻게 사용할 수 있을까?

https://www.typescriptlang.org/docs/handbook/2/keyof-types.html

## keyof 연산자

`keyof` 연산자는 객체 타입을 입력 받아 '객체의 키의 문자열이나 숫자 리터럴 유니온'을 반환한다.

```typescript
type Point = { x: number; y: number };
type P = keyof Point;
```

이 예제에서 `P`는 `"x" | "y"` 와 동일한 타입을 가진다.

```typescript
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
```

타입이 `string`이나 `number` 인덱스 시그니쳐를 가지는 경우에 `keyof`는 아래의 타입을 대신 리턴한다.

- 상기 예시에서, `A` 타입은 `number` 타입을 가진다.
- 상기 에시에서, `M` 타입은 `string | number` 타입을 가진다.

`M` 타입이 `string | number` 타입을 가지는 이유는 무엇일까?

자바스크립트 객체의 키들은 항상 string으로 강제 지정되기 때문이다.
때문에 `obj[0]`은 항상 `obj["0"]` 과 동일하다.

## 언제 쓸까?

`keyof` 연산자는 사실 단독으로 쓰일 일은 많지 않지만, 맵드 타입과 같이 사용했을 때 그 진가를 발휘할 수 있다.

맵드 타입은 코드의 중복을 피하기 위해, 다른 타입을 바탕으로, 새로운 타입을 생성하기 위해 사용된다.

그리고 매핑된 타입은 `PropertyKey`의 조합을 사용해서 타입을 반복적으로 생성하는 제너릭 타입이다.
그리고 `PropertyKey`를 생성할 때에는 `keyof`를 자주 사용하게 된다.

