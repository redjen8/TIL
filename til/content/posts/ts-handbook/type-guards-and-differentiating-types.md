---
title: "타입스크립트 핸드북 10장 - 타입 가드와 차별 타입"
date: 2024-01-31T19:45:26+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/advanced-types.html#%ED%83%80%EC%9E%85-%EA%B0%80%EB%93%9C%EC%99%80-%EC%B0%A8%EB%B3%84-%ED%83%80%EC%9E%85-type-guards-and-differentiating-types

유니언 타입: 타입들을 겹치는 상황에서 사용, 하지만 한 타입이 명시적으로 존재하는지 구체적으로 알고 싶다면? 

```js
let pet = getSmallPet();

if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

위 코드는 아래와 같이 타입 단언 (type assertion)을 사용해서 바꿀 수 있다.

```ts
let pet = getSmallPet();

if ((pet as Fish).swim) {
    (pet as Fish).swim();
} else if ((pet as Bird).fly) {
    (pet as Bird).fly();
}
```

- 하지만 위 방법은 타입 단언 (`A as B`)를 여러 번 사용해야 한다는 불편함이 존재한다. 
- 때문에 타입스크립트에서는 여러 종류의 타입 가드를 지원한다.
- 타입 가드를 사용한다면 스코프 안에서의 타입을 보장하는 런타임 검사를 컴파일러가 수행해준다.

### User Defined Type Guards

#### Using type predicates

> 반환 타입이 타입 서술어(`A is B`)인 함수를 작성한다.

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```

#### Using the `in` operator

`in` 연산자는 타입을 좁히는 표현으로 쓰인다.

```ts
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```

`n in x` 표현식에 대해서
- `n`은 문자열 리터럴 또는 문자열 리터럴 타입
- `x`는 유니언 타입
- `true` 분기에서는 필수 프로퍼티 `n`을 가지는 타입으로 좁히고
- `false` 분기에서는 누락된 프로퍼티 `n`을 가지는 타입으로 좁힌다.

#### `typeof` type guards

```ts
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

위 코드에서는 타입이 원시 값인지 확인하는 것이 번거롭다. (`isNumber()`, `isString()`)
- 하지만 타입스크립트에서는 `typeof`를 타입 가드로 사용할 수 있다.
- 때문에 `typeof x === "number"`를 함수로 추싱화할 필요 없다.

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

#### `instanceof` type guards

```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 타입은 'SpaceRepeatingPadder | StringPadder' 입니다
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 타입은 'SpaceRepeatingPadder'으로 좁혀집니다
}
if (padder instanceof StringPadder) {
    padder; // 타입은 'StringPadder'으로 좁혀집니다
}
```
`getRandomPadder()` 함수는 반반 확률로 `SpaceRepeatingPadder` 와 `StringPadder` 인스턴스를 생성한다.

- `instanceof`의 우변은 생성자 함수
- 타입스크립트는 타입을 아래의 순서로 좁힌다.
  - 함수의 `prototype` 프로터티 타입이 `any`가 아닌 타입
  - 타입의 생성자 시그니쳐에서 반환된 유니언 타입

### Angular `HttpInterceptor` 에서의 예시

```ts
return next.handle(clonedRequest).pipe(
      map((event: HttpEvent<any>) => {
        if (event instanceof HttpResponse) {
          // 정상적으로 서버로부터 응답을 받았을 경우 클라이언트 측 액션
        }
        return event;
      }),
      catchError((err: any) => {
        if (
          err instanceof HttpErrorResponse &&
        ) {
          if (true) {
            // 콘솔 상으로 에러를 남길 수 있는 형태의 에러 포맷인 경우
            console.error(err.message);
          }
          // 서버 통신 에러 시 예외 지정하지 않는다면 alert 모달 표출
          this.alertModalService.alert();
        }

        return throwError(err);
      }),
    );
```