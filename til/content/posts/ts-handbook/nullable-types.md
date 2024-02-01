---
title: "타입스크립트 핸드북 11장 - 널러블 타입"
date: 2024-01-31T20:18:09+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/advanced-types.html#%EB%84%90%EB%9F%AC%EB%B8%94-%ED%83%80%EC%9E%85-nullable-types

### Nullable Types

> `null`과 `undefined`는 아무 것에나 할당 가능하다. 

- 이를 방지하기 위한 tsconfig 플래그 값이 `--strictNullChecks`
- TS 3.7 이후부터는 optional chaining을 사용할 수 있다.
- 타입스크립트는 `null`과 `undefined`를 다르게 처리한다.

#### Optional Parameters and Properties

tsconfig의 `--strictNullChecks` 플래그를 적용하면 선택적 매개변수와 프로퍼티에 ` | undefined` 가 자동으로 추가된다.

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다
```

```ts
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // 오류, 'undefined'는 'number'에 할당할 수 없습니다
c.b = 13;
c.b = undefined; // 성공
c.b = null; // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다.
```

즉 `?`로 표시된 선택적 매개변수 또는 프로퍼티에는 명시적으로 `null` 할당은 불가능, `undefined` 할당은 가능하다.
- js deepdive에서도 배웠었지만 실행 컨텍스트 평가 시점에 암묵적으로 `undefined`를 할당하기 때문, `undefined`는 반드시 할당 허용되어야 하는 값
- https://redjen8.github.io/posts/js-deepdive/execution-context/

#### Type Guards And Type Assertions

널러블 타입이 유니언으로 (`| undefined`)로 구현되기 때문에, `null`을 제거해주기 위한 타입 가드가 필요하다.

```ts
function f(sn: string | null): string {
    return sn || "default";
}
```

컴파일 시점에 `null`이 들어올지 몰라 오류를 발생시키지 못하는 경우, 타입 단언 연산자인 `!`를 사용해서 수동으로 `null` 값을 제거할 수 있다.

```ts
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // 오류, 'name'은 아마도 null 입니다
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // 성공
  }
  name = name || "Bob";
  return postfix("great");
}
```

- 타입스크립트 컴파일러는 중첩 함수 안에서 `null` 제거가 불가능 (즉시 실행 함수는 예외)
  - 이유는 잘 생각해봤을 때 외부 함수에서 함수를 nested하게 호출한 경우 모든 내부 함수를 추적하는 것이 사실상 불가능
  - ㄷ함수가 어디에서 호출되었는지 알 수 없다면 컴파일 시점에 `name`의 타입을 알 수 없다.