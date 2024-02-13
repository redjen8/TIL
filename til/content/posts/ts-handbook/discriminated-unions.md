---
title: "타입스크립트 12장 - 판별 유니온"
date: 2024-02-13T21:58:17+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/advanced-types.html#%ED%8C%90%EB%B3%84-%EC%9C%A0%EB%8B%88%EC%96%B8-discriminated-unions


> 싱글톤 패턴 + 유니언 타입 + 타입 가드 + type alias의 짬뽕

아래의 3가지 요소로 구성된다.
- 공통 싱글톤 타입 프로퍼티를 가지는 타입
- 해당 타입들의 유니언으로써의 타입
- 공통 프로퍼티의 타입 가드

### 싱글톤 타입이란

https://medium.com/@tar.viturawong/using-typescripts-singleton-types-in-practice-f8b20b1ec3a6

위 글에서는 '리터럴 타입'을 싱글턴 타입으로써 소개한다.

타입스크립트는 primitive한 멤버를 primitive type의 값이 아닌 정확한 그 값으로 인식하는 경우 == 싱글턴 타입

`"Medium"` 은
- `string` 타입으로 볼 수도 있지만
- `"Medium"` 문자열 타입으로도 볼 수 있다

이렇게 해서 얻을 수 있는 이점이 뭐냐?

### 싱글턴 타입의 이점

#### 1. 조건문을 깔끔하게 구성할 수 있다

이메일 주소를 입력받는 input 필드에 대한 validation 함수를 구성한다고 예를 들어보자.

```ts
function validateRequiredEmail(value: string) {
    if ( !value ) return "empty";
    if ( value.indexOf('@') < 0 ) return "noAtSign";
    return null;
}
```

위 함수의 리턴 타입은 `null | "empty" | "noAtSign"`이 된다.

```ts
const validationResult = validateRequiredEmail("not an email");
if ( validationResult === "empty" ) doThis();
else if ( validationResult === "noAtSign" ) doThat();
else if ( validationResult === "broccoli" ) doSomethingElse();
```

때문에 위와 같은 방식으로 구성할 수 있다. 마지막 `broccoli`에 대한 조건문에서 이는 리턴되지 않는 타입임을 캐치할 수 있다고 개발 과정 중에 알 수 있는 이점이 있다.

#### 2. 판별 유니온에서의 사용

아래에서 설명..

### 판별 유니온의 예시

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

- `kind` 멤버를 전부 문자열 리터럴 타입 (싱글턴 타입)으로써 서술하였고
- `kind` 멤버에 대한 유니언 타입 `Shape`을 선언하였고
- `Shape` 인자에 대한 타입 가드로써 switch - case 조건문을 사용하여 분기 처리하였다.

### 엄격한 검사

위 `Shape` 타입에 한 가지 타입이 더 추가가 된다면 어떨까?

`type Shape = Square | Rectangle | Circle | Triangle ;`

기존에 작성한 `area` 함수는 `case "triangle"`에 대한 처리가 없기 때문에 오류가 발생한다. 이를 해결하기 위해서는 
- `--strictNullChecks`를 사용하기
  - `undefined`를 반환하는 기존 `area()` 함수에서 에러를 체크할 수 있게 해주긴 한다
  - 하지만 backward compability 면에서 조금 아쉽다
- `never` 타입을 사용하기

```ts
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // 빠진 케이스가 있다면 여기서 오류 발생
    }
}
```

위와 같이 `never` 타입을 사용해 구성한다면 `s` 인자가 `never` 타입인지 검사하기 때문에, `s`는 `undefined`가 아닌 `never` 타입을 가지게 되기 때문에 조금 더 명백한 코드가 된다는 이점이 있다.
