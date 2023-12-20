---
title: "타입스크립트 핸드북 1장 - 기본 타입"
date: 2023-12-20T22:16:43+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/basic-types.html

### Number

- 모든 숫자는 js 처럼 부동 소수값
- 16진수 리터럴, 10진수 리터럴, 2진수 리터럴, 8진수 리터럴을 지원

### String

- 큰 따옴표 `"` 나 작은 따옴표 `'` 로 문자열 데이터를 감싸서 사용
- 템플릿 문자열 (`${expr}`) 을 사용해 표현식을 포함시킬 수 있음

### Array

- 배열 요소들을 나타내는 타입 뒤에 `[]` 사용해서 선언
   - `let list: number[]`
- 제네릭 배열 타입으로 사용
   - `let list: Array<number>`

### Tuple

> 요소의 타입과 개수가 고정된 배열

```ts
let x: [string, number];

x = ["hello", 10];
```

### Enum

> 값의 집합에 이름을 붙여준다

```ts
enum Color {Red, Green, Blue}
let c: Color = Color.green;
```

- 기본 값은 0부터 시작
- 값을 수동으로 설정 가능

```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2]; //Green
```

### Any

- 알지 못하는 타입을 표현해야 하는 경우에 사용
- 이 경우 타입 검사를 하지 않음
   - 해당 값은 컴파일 시간에 검사를 skip
- js에서 하는 것처럼 `Object`를 할당할 경우 메서드 호출이 불가능

```ts
let anyVar: any = 4;
let obj: Object = 4;
anyVar.toFixed(); //성공
obj.toFixed(); //오류
```

### Void

> 어떤 타입도 존재할 수 없음을 나타냄

- 메서드의 반환 타입이 void인 경우
   - 일반적인 아무 것도 반환하지 않는 함수의 타입으로 사용
- 변수의 타입으로 void를 사용하는 경우
   - 해당 변수에는 `null` 또는 `undefined`만 할당 가능

### Null과 Undefined

- 기본적으로 `null`과 `undefined`는 모든 타입의 서브 타입
   - 즉 `null`과 `undefined`는 `number`와 같은 타입에 할당 가능
- `--strictNullChecks`를 사용하면 `null` 과 `undefined`는 오직 `any`와 각자의 타입에만 할당 가능
   - 예외적으로 `undefined`는 `void`에 할당 가능

### Never

> 절대 발생할 수 없는 타입

- 함수 표현식이나 화살표 함수 표현식에서 항상 오류를 발생시키거나 절대 반환하지 않는 반환 타입으로 사용됨
- 변수에서는 타입 가드에 의해 아무 타입도 얻지 못하게 되는 경우 `never` 타입이 될 수 있음
- 마찬가지로 `never` 타입은 모든 타입에 할당 가능한 하위 타입
   - 하지만 반대로 어떤 타입도 `never`에 할당 불가능하고, 서브 타입이 아니다 (any도) 

### Type Assertions

- 개발자가 ts보다 값에 대해 더 잘 알고 있는 경우가 있을 수 있다
   - 어떤 엔티티의 실제 타입이 현재 타입보다 더 구체적인 경우
- 다른 언어의 타입 변환과 유사하지만 다른 검사를 하거나 데이터를 재구성하지는 않음
- 런타임에 영향을 끼치지 않고, 컴파일 단계에서만 이를 활용한다

```ts
let someValue: any = "this is a string";
let strLength1: number = (<string>someValue).length; // angle-bracket 문법
let strLength2: number = (someValue as string).length; // as 문법
```