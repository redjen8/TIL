---
title: "타입스크립트 핸드북 15장 - 매핑 타입"
date: 2024-02-21T20:49:40+09:00
draft: false
author: redjen
---
https://typescript-kr.github.io/pages/advanced-types.html#%EB%A7%A4%ED%95%91-%ED%83%80%EC%9E%85-mapped-types

> 기존 타입을 '매핑' 하여 각 프로퍼티를 변환한 신규 타입을 만드는 방법

```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}

type PersonPartial = Partial<Person>;
type readonlyPerson = Readonly<Person>;
```

- 프로퍼티 목록은 `[P in keyof T]`.로 접근하고, 결과 타입은 `T[P]`로써 접근할 수 있다.
	- 마치 템플릿 처럼 활용 가능
- 동형의 변경이기 때문에 `T`의 프로퍼티에만 매핑을 적용 가능
- 컴파일러 레벨에서 접근 지정자와 같은 프로퍼티 지정자를 알 수 있다는 장점도
	- `Person.name`이 readonly 라면 `Partial<Person>.name` 또한 readonly 일 것이라 추론

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
type Record<K extends keyof any, T> = {
    [P in K]: T;
}
```

- `Record`타입의 경우 `T[P]`를 결과 타입으로써 접근하지 않는다.
	- 동형의 매핑이 아니다.
		- 프로퍼티를 복사하는 입력 타입을 받지 않기 떄문 (`K extends keyof any`)
		- 때문에 프로퍼티 지정자를 복사할 수 없다.
		- 본질적으로  새로운 프로퍼티를 생성한다.

### Inference from mapped types

- 매핑 타입: 타입을 한 단계 감싸서 새로운 타입을 만든다
- 감싼 타입을 언래핑하기 위해서는?

```ts
function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

let originalProps = unproxify(proxyProps);
```


