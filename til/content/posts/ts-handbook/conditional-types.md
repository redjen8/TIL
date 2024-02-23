---
title: "타입스크립트 핸드북 16장 - 조건부 타입"
date: 2024-02-23T21:11:45+09:00
draft: false
author: redjen
---
https://typescript-kr.github.io/pages/advanced-types.html#%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85-conditional-types

`T extends U ? X : Y`

- T가 U에 할당될 수 있다면 X 타입이 된다.
- T가 U에 할당 불가능한 경우 Y 타입이 된다.

조건부 타입은 다음의 특징을 가진다.
- 타입이 `X` 또는 `Y`로 결정되는 것처럼 보이지만 지연될 수도 있다
	- 조건이 하나 이상의 타입 변수에 의존하기 때문 (`T extends U`)
- 즉, 타입이 즉시 결정되지 못하는 경우가 있다

#### 타입이 즉시 결정되는 예시

```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// 타입은 'string | number'
let x = f(Math.random() < 0.5)
```

```ts
type TypeName<T> =
    T extends string ? "string" :
    T extends number ? "number" :
    T extends boolean ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;  // "string"
type T2 = TypeName<true>;  // "boolean"
type T3 = TypeName<() => void>;  // "function"
type T4 = TypeName<string[]>;  // "object"
```

#### 타입이 분기되지 않고 고정되는 예시

```ts
interface Foo {
    propA: boolean;
    propB: boolean;
}

declare function f<T>(x: T): T extends Foo ? string : number;

function foo<U>(x: U) {
    // 'U extends Foo ? string : number' 타입을 가지고 있습니다
    let a = f(x);

    // 이 할당은 허용됩니다!
    let b: string | number = a;
}
```

- `a`는 아직 분기 선택되지 못한 조건부 타입을 가진다. 
	- `foo()` 의 인자로 입력된 타입이 결정되어야 분기가 결정되어 조건부 타입이 결정
- 하지만 조건부 타입이 평가되지 않아도 결국 `string` 타입 또는 `number` 타입 중에서 선택한다는 사실은 변하지 않는다
	- 때문에 `string | number` 유니언 타입으로 할당이 가능하다

### Distributive conditional types

https://stackoverflow.com/questions/51651499/typescript-what-is-a-naked-type-parameter

#### naked type parameter

> naked type parameter는 다른 타입 / 제네릭으로 감싸지지 않은 타입

```javascript
type NakedUsage<T> = T extends boolean ? "YES" : "NO"
type WrappedUsage<T> = [T] extends [boolean] ? "YES" : "NO";
```

```javascript
type Distributed = NakedUsage<number | boolean > // = NakedUsage<number> | NakedUsage<boolean> =  "NO" | "YES" 
type NotDistributed = WrappedUsage<number | boolean > // "NO"    
type NotDistributed2 = WrappedUsage<boolean > // "YES"
```

naked vs non naked (래핑된) 타입 인수의 차이는 다음과 같다.
- naked 타입 인수의 경우 유니언에 대해 **분산된다**.
	- 때문에 조건부 타입이 유니언의 각 멤버에 대해 적용된다.
- 즉 검사 대상 타입이 naked type parameter인 조건부 타입 == 분산 조건부 타입

#### 조건부 타입을 유니언 타입으로 분산시키는 예제

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>;  // BoxedValue<string>;
type T21 = Boxed<number[]>;  // BoxedArray<number>;
type T22 = Boxed<string | number[]>;  // BoxedValue<string> | BoxedArray<number>;
```

#### 유니언 타입을 필터링하기 위한 분산 조건부 타입의 예제

```ts
type Diff<T, U> = T extends U ? never : T;  // U에 할당할 수 있는 타입을 T에서 제거
type Filter<T, U> = T extends U ? T : never;  // U에 할당할 수 없는 타입을 T에서 제거

type T30 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T31 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"
type T32 = Diff<string | number | (() => void), Function>;  // string | number
type T33 = Filter<string | number | (() => void), Function>;  // () => void

type NonNullable<T> = Diff<T, null | undefined>;  // T에서 null과 undefined를 제거

type T34 = NonNullable<string | number | undefined>;  // string | number
type T35 = NonNullable<string | string[] | null | undefined>;  // string | string[]

function f1<T>(x: T, y: NonNullable<T>) {
    x = y;  // 성공
    y = x;  // 오류
}

function f2<T extends string | undefined>(x: T, y: NonNullable<T>) {
    x = y;  // 성공
    y = x;  // 오류
    let s1: string = x;  // 오류
    let s2: string = y;  // 성공
}
```

#### 매핑 타입과 결합된 조건부 타입의 예제

```ts
type FunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? K : never }[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
    id: number;
    name: string;
    subparts: Part[];
    updatePart(newName: string): void;
}

type T40 = FunctionPropertyNames<Part>;  // "updatePart"
type T41 = NonFunctionPropertyNames<Part>;  // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part>;  // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part>;  // { id: number, name: string, subparts: Part[] }
```

### Type inference in conditional types

https://medium.com/@developer.olly/understanding-typescript-infer-ac42bd018f3

> `infer` 키워드는 조건부 타입에서 타입으로부터 다른 타입을 찾기 위해서 사용된다.

- `infer` 선언을 통해 타입의 그룹으로부터 타입을 뽑아서 다른 타입의 정의에 사용할 수 있다.
- 서로 다른 여러 입력 타입이 존재하는 제너럴한 타입을 만들 때 유용하다.

```ts
type UnArray<T> = T extends any[] ? T[number] : T
```

위와 같은 방식으로 unwrap 하는 타입을 만들수도 있지만

```ts
type UnArray<T> = T extends Array<infer U> ? U : T;
```

위와 같이 사용한다면 더 깔끔한 접근 방법이 된다.

```ts
type Unpacked<T> =
    T extends (infer U)[] ? U :
    T extends (...args: any[]) => infer U ? U :
    T extends Promise<infer U> ? U :
    T;

type T0 = Unpacked<string>;  // string
type T1 = Unpacked<string[]>;  // string
type T2 = Unpacked<() => string>;  // string
type T3 = Unpacked<Promise<string>>;  // string
type T4 = Unpacked<Promise<string>[]>;  // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>;  // string
```

즉 하나의 타입 안에서 하나의 타입을 콕 꼽아서 다른 제너럴한 타입을 만드는 좋은 도구가 될 수 있다.

```ts
type ReturnType<T extends (...args: any[]) => infer R> = R;
```

- 하지만 위와 같이 **조건부 타입이 아닌, 일반 타입 인수에 대한 constraint 절에서 `infer`를 사용할 수는 없다.**
- 일반 타입 인수의 제약 조건에서 타입 변수를 지우고 조건부 타입을 지정하면 비슷하긴 하다.

```ts
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R ? R : any;
```

### Predefined conditional types

ts 2.8에서는 자주 사용되는 조건부 타입이 추가되었다.
- `Exclude<T, U>`
	- U에 할당할 수 있는 타입은 T에서 제외
	- `type Exclude<T, U> = T extends U ? never : T;`
- `Extract<T, U>`
	- U에 할당할 수 있는 타입만 T에서 추출
	- `type Filter<T, U> = T extends U ? T : never;`
- `NonNullable<T>`
	- T에서 `null` 과 `undefined`를 제외
	- `type NonNullable<T> = Exclude<T, null | undefined>;`
- `ReturnType<T>`
	- 함수 타입의 반환 타입을 추출
	- `type ReturnType<T> = T extends (...a: any[]) => infer R ? R : any;
- `InstanceType<T>`
	- 생성자 함수 타입의 인스턴스 타입을 추출
	- `type InstanceType<T> = T extends new (...args: any[]) => infer R ? R : any;


```ts
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function>;  // string | number
type T03 = Extract<string | number | (() => void), Function>;  // () => void

type T04 = NonNullable<string | number | undefined>;  // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined>;  // (() => string) | string[]

function f1(s: string) {
    return { a: 1, b: s };
}

class C {
    x = 0;
    y = 0;
}

type T10 = ReturnType<() => string>;  // string
type T11 = ReturnType<(s: string) => void>;  // void
type T12 = ReturnType<(<T>() => T)>;  // {}
type T13 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T14 = ReturnType<typeof f1>;  // { a: number, b: string }
type T15 = ReturnType<any>;  // any
type T16 = ReturnType<never>;  // never
type T17 = ReturnType<string>;  // 오류
type T18 = ReturnType<Function>;  // 오류

type T20 = InstanceType<typeof C>;  // C
type T21 = InstanceType<any>;  // any
type T22 = InstanceType<never>;  // never
type T23 = InstanceType<string>;  // 오류
type T24 = InstanceType<Function>;  // 오류
```
