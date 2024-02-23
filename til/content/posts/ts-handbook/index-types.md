---
title: "타입스크립트 핸드북 14장 - 인덱스 타입"
date: 2024-02-20T23:53:02+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/advanced-types.html#%EC%9D%B8%EB%8D%B1%EC%8A%A4-%ED%83%80%EC%9E%85-index-types

동적인 프로퍼티 이름을 사용하는 코드를 컴파일러가 알게 하고 싶은 경우

```ts
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
  return propertyNames.map(n => o[n]);
}

interface Car {
    manufacturer: string;
    model: string;
    year: number;
}
let taxi: Car = {
    manufacturer: 'Toyota',
    model: 'Camry',
    year: 2014
};

// Manufacturer과 model은 둘 다 문자열 타입입니다,
// 그래서 둘 다 타이핑된 문자열 배열로 끌어낼 수 있습니다.
let makeAndModel: string[] = pluck(taxi, ['manufacturer', 'model']);

// 만약 model과 year를 끌어내려고 하면,
// 유니언 타입의 배열: (string | number)[] 을 얻게됩니다.
let modelYear = pluck(taxi, ['model', 'year'])
```

- `pluck` 함수는 인자로 입력 받은 프로퍼티의 이름 기반으로 (`propertyName`) 객체의 프로퍼티를 추출해낸다.
- `keyof T`는 인덱스 타입 쿼리 연산자로 사용되었다
	-  `T`는 `any` 타입이다.
	- `keyof T`는 `T`의 공개된 프로퍼티 이름들의 유니언이다.
	- `keyof`를 사용하였을 때, 컴파일러가 인자로 전달되는 올바른 프로퍼티 집합에 대해 검사한다는 장점
- `T[K]`는 인덱스 접근 연산자로 사용되었다.
	- 제네릭 컨텍스트에서 사용했기 때문에 조금 난해할지도
	- `person['name']`은 `Person['name']` 타입을 가진다. 

### 인덱스 타입과 인덱스 시그니쳐

`keyof`와 `T[K]`가 인덱스 시그니쳐와 상호작용하여 사용될 수 있다.

#### 인덱스 시그니쳐란

https://radlohead.gitbook.io/typescript-deep-dive/type-system/index-signatures

객체에 들어 있는 어떠한 객체 참조도 문자열을 통해 엑세스 가능하다. 
- 자바스크립트의 경우 객체의 인덱스 시그니쳐에 사용된 객체에 대해 암묵적으로 `toString()`을 호출한다.

```ts
let obj = {
  toString(){
    return 'Hello'
  }
}

let foo: any = {};

// 오류: 인덱스 서명은 string, number 여야 함...
foo[obj] = 'World';

// FIX: TypeScript는 명시적으로 호출하게 강제함
foo[obj.toString()] = 'World';
```

자바스크립트와 달리 타입스크립트에서 사용자가 명시적으로 `toString()`을 처리하게 하는 이유는 다음과 같다.
- 기본적으로 객체의 `toString()` 구현이 제대로 되어있지 않기 때문이다.
- v8 에서는 항상 `[object Object]`를 반환하는데, 이는 `foo["object Object"]`와 같은 접근을 시도
- 때문에 객체의 인덱스 시그니쳐로 `number`타입을 지원한다.

```ts
interface Dictionary<T> {
    [key: string]: T;
}
let keys: keyof Dictionary<number>; // string | number
let value: Dictionary<number>['foo']; // number
```

```ts
interface Dictionary<T> {
    [key: number]: T;
}
let keys: keyof Dictionary<number>; // 숫자
let value: Dictionary<number>['foo']; // 오류, 프로퍼티 'foo'는 타입 'Dictionary<number>'에 존재하지 않습니다.
let value: Dictionary<number>[42]; // 숫자
```