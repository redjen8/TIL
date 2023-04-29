---
title: "Typescript Type Class Interface 비교"
date: 2023-04-29T21:13:34+09:00
draft: false
author: redjen
---

Typescript를 사용하다 보니 `interface`와 `type`의 차이점이 헷갈리게 되었다.
또 이것들은 `class` 와는 어떻게 다를까?

https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces

## Type Aliases

우리는 `Object` 타입과 `union` 타입을 타입 어노테이션을 직접적으로 활용해서 사용해왔다.
이런 방식은 편리하긴 하지만, 대부분의 경우에는 같은 타입을 한 번 이상 사용해야 할 유즈케이스가 많다는 것이 문제이다.

Type alias는 어떤 타입에 대한 이름이다. 

```typescript
type Point = {
    x: number;
    y: number;
};

function printCoord(pt: Point) {
    console.log("x : " + pt.x);
    console.log("y : " + pt.y);
}

printCoord({x: 100, y: 100});
```

Type alias를 사용해서 어떠한 타입에 대해서라도 이름을 부여할 수 있다. (심지어는 객체 타입까지도)

```typescript
type ID = number | string;
```

하지만 alias는 alias일 뿐이고, 실제로 type alias를 사용해서 서로 다른 버전의 동일 타입을 생성할 수 없음에 주의해야 한다.

아래 코드는 유효하지 않은 것처럼 보일 수 있지만, 타입스크립트가 처리할 때에는 두 가지 타입들이 동일 타입에 대한 서로 다른 alias로 처리한다.
```typescript
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// Create a sanitized input
let userInput = sanitizeInput(getInput());
 
// Can still be re-assigned with a string though
userInput = "new input";
```

## Interface

인터페이스 선언은 객체 타입에 이름을 붙이기 위한 또 다른 방법이다.

```typescript
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

아까 살펴봤던 type alias와 사용법이 똑같다!

타입스크립트가 신경 쓰는 것은 `printCoord`로 념겨지게 되는 value의 구조만 해당된다. 즉 예상한 속성들만 넘어오는 지의 여부만 검사한다.

구조와 수행할 수 있는 일들의 범위만 신경쓰게 하는 것 때문에 타입스크립트를 **Structurally Typed** 타입 시스템을 가졌다고 말할 수 있는 것이다.

## Type Alias vs Interface Type

두 가지 방법을 통해 타입에 이름을 붙이는 것을 알아봤는데, 그럼 이 둘은 어떻게 다를까?

많은 경우에는 두 방법을 구분 없이 사용할 수 있다. `interface`의 거의 모든 기능들은 `type`에서 사용할 수 있다.

하지만 `type`은 새로운 속성들에 대해 다시 정의될 수 없는 반면 `interface`는 항상 확장 가능하다는 점이 주된 차이점이라고 할 수 있겠다.

인터페이스의 확장 방법은..
```typescript
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

반면 타입의 확장 방법은..
```typescript
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```

인터페이스에 새 필드를 추가하려면..
```typescript
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

반면 타입은 새 필드의 재정의가 불가능하다.
```typescript
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'.
```

## Class

`type`과 `interface`가 타입스크립트의 기능인 반면, `class`는 ES6에 포함된 자바스크립트의 기능이다. 

https://ultimatecourses.com/blog/classes-vs-interfaces-in-typescript

인터페이스가 인스턴스화 할 수 없는 것과는 반대로, 인터페이스를 클래스로 만들어서 클래스의 인스턴스를 반환하도록 하는 것은 가능하다.

- `class`: 인자, 반환 유형, 제네릭과 같은 타입 검사의 이점을 누리면서 사용자 정의 객체의 인스턴스를 생성해야 하는 경우 사용
- `interface`: 인스턴스를 생성하지 않아도 되는 경우, 소스 코드를 생성하지 않으면서도 가상으로 타입을 검사하고 싶은 경우 사용

클래스와 인터페이스는 모두 객체의 구조를 정의하고, 경우에 따라서 서로 바꿔서 사용할 수 있다.

때문에 여러 클래스 간에 구조 정의를 공유해야 하는 경우
- 인터페이스에서 해당 구조를 정의한 다음
- 각 클래스가 해당 인터페이스를 구현하도록 할 수 있다.
- 이런 경우 각 클래스는 인터페이스의 각 속성을 선언하거나 구현해야 한다.

