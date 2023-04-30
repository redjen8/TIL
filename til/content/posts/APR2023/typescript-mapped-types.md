---
title: "Typescript Mapped Type"
date: 2023-04-30T13:28:37+09:00
draft: false
author: redjen
---

타입스크립트에서 제공하는 타입을 사용하다 보면 계속해서 사용되는 타입 자체를 공통화 하고 싶은 경우가 생긴다.

이럴 때 사용하는 것이 Mapped Type이라던데, 맵드 타입에 대해 알아봤다.

https://joshua1988.github.io/ts/usage/mapped-type.html#%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-map-%ED%95%A8%EC%88%98%EB%9E%80

## Mapped Type

Mapped Type이란 기존에 정의되어 있는 타입을 새로운 타입으로 변환해주는 문법이다.

```typescript
// before
{ 
    name: string;
    email: string;
}

// after
{
    name: number;
    email: number;
}
```

마치 자바스크립트의 `map` 함수를 타입에 적용시키는 느낌으로 이해하면 될 듯하다.

## 기본 문법

```typescript
{ [P in K] : T}
{ [P in K] ? : T}
{ readonly [P in K] : T}
{ readonly [P in K] ? : T}
```

기본적인 폼에서, `readonly` 키워드의 유무와 `?` 옵셔널 키워드의 유무로 총 4가지 용례가 존재한다.

## 간단한 예제

```typescript
type Heroes = 'Hulk' | 'Thor' | 'Capt';
```

유니온 타입 `Heroes`에, 나이 정보를 붙인 객체를 저장하기 위한 타입으로 바꾸기 위해서 맵드 타입을 사용할 수 있다.

```typescript
type HeroProfiles = { [K in Heroes] : number };
const heroInfo: HeroProfiles = {
    Hulk: 54,
    Thor: 1000,
    Capt: 33,
} 
```
상기 코드에서 `[K in Heroes]` 부분은 자바스크립트의 `for ~ in` 문법과 동일하다.

사전에 정의했던 `Heroes` 타입을 순회하여서 `number` 타입 값을 가지는 객체의 키로써 정의가 된다. 결과적으로는 아래와 같은 타입을 정의하게 되는 것이다.

```typescript
type HeroProfiles = {
    Hulk: number;
    Thor: number;
    Capt: number;
}
```
## 보다 어려운 예제

```typescript
type Subset<T> = {
    [K in keyof T] ? : T[K];
}
```

상기 코드는 키와 값이 있는 객체를 정의하는 타입을 그 객체의 부분 집합을 만족하는 타입으로 변환해준다.

```ts
interface Person {
    age: number;
    name: string;
}
```

위 `Subset` 타입을 적용하면 아래와 같은 객체를 모두 정의할 수 있다.
```ts
const ageOnly: Subset<Person> = {age: 23};
const nameOnly: Subset<Person> = {name: 'Tony'};
const empty: Subset<Person> = {};
```

## Mapping Modifiers

https://www.typescriptlang.org/docs/handbook/2/mapped-types.html

`readonly`와 `?` 연산자는 그럼 어떤 역할을 할까?
- `readonly`는 mutability에 영향을 준다.
- `?`는 optionality에 영향을 준다.

그리고 각각의 modifier들은 `-`, `+` prefix를 붙여서 더하고 뺄 수 있다.

```ts
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;

//결과
type UnlockedAccount = {
    id: string;
    name: string;
}
```

