---
title: "타입스크립트 핸드북 2장 - 인터페이스"
date: 2023-12-20T22:19:12+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/interfaces.html

타입스크립트의 핵심 원칙 중 하나는 타입 검사가 **값의 형태**에 초점을 맞춘다는 것 (duck typing)

- 타입 검사는 프로퍼티들의 순서를 요구하지 않고, 아래 2가지만 검사한다.
   - 인터페이스가 요구하는 프로퍼티가 존재하는지?
   - 프로퍼티들이 요구하는 타입을 가지고 있는지?

### Optional Properties

인터페이스의 모든 프로퍼티가 필요한 경우는 사실 잘 없다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

> Optional한 프로퍼티는 이름 끝에 `?`를 붙여 명시한다.

### Readonly Properties

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
```

객체 리터럴을 생성하여 `Point`를 생성하고 할당된 이후에는 `x`, `y`를 수정할 수 없다.

#### readonly vs const

-  변수는 `const`를 사용한다.
- 프로퍼티는 `readonly`를 사용한다.

### Excess Property Check

타입스크립트는 `{label: string}`을 기대해도 `{size: number; label: string; }`를 허용해준다.

그런데 이 때 optional property를 같이 사용하면 에러가 발생할 수 있다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

- `width` 프로퍼티는 있다
- `color` 프로퍼티는 없다
- `colour` 프로퍼티는 잉여 데이터가 아닌가?
   - 객체 리터럴은 다른 변수에 할당할 때와 인수로 전달할 때 초과 프로퍼티 검사를 받는다.
   - 만약 이 때 객체 리터럴이 대상 타입에는 없는 프로퍼티를 가지면 에러 발생한다.
   - type assertion을 사용하여 피할 수 있다.
   - `let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);`

추가 프로퍼티가 있다고 확신한다면 문자열 인덱스 서명을 추가하는 방법도 있다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

### 함수 타입

인터페이스는 함수 타입을 설명할 수 있다.

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

타입스크립트는 인터페이스에 call signature를 전달하여 함수 타입을 기술한다.
- argument list와 return type만 주어진 함수 선언과 유사

```ts
mySearch = function(src, sub) {
  let result = src.search(sub);
  return "string"; //에러 발생
};
```

### 인덱서블 타입

> 타입을 인덱스로도 기술할 수 있다.

```ts
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```
