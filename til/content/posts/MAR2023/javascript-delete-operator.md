---
title: "Javascript Delete Operator"
date: 2023-03-31T17:22:47+09:00
draft: false
author: redjen
---

Javascript에서 `delete` 연산자는 언제, 왜 쓰일까?

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/delete

## 기능

`delete` 연산자가 객체를 삭제하는 것이라고 생각했다면, 오산이다.

`delete` 연산자는 객체를 삭제하지 않고 단순히 객체의 속성들을 제거하는 연산자이다.

그 후에 제거한 객체의 참조가 어디에서도 사용되지 않는다면 GC가 할당된 리소스를 수거해간다.

## 주의해야 할 점

앞서 말했듯이 `delete`는 메모리 해제에 대해 직접적으로 어떤 작업도 하지 않는다.

자바스크립트는 GC가 존재하는 언어이고, 그리고 모든 메모리 관리는 GC를 통해 간접적으로 일어난다.

이 사실을 포함하여 `delete`를 사용할 때 조금 헷갈리는 엣지 케이스들이 몇 개 있는데,
- 존재하지 않는 속성을 삭제한다면 `delete`는 아무 에러도 발생시키지 않고 `true`를 반환한다.
- 객체의 프로토타입 체인에 같은 이름의 속성이 있는 경우
  - 삭제 후에 객체의 프로토체입 체인을 통해 '삭제된 것처럼 보이는' 속성을 사용할 수 있다.
  - 즉, `delete`는 오직 해당 객체 자신의 속성만 제거한다.
- `var`로 선언된 어떤 속성이라도 글로벌 스코프나 함수 스코프에서 제거될 수 없다.
  - 글로벌 스코프의 어떤 함수라도 `delete`로 제거될 수 없다.
  - 객체의 속성으로 존재하는 함수는 `delete`로 제거 될 수 있다. (마찬가지로 글로벌 스코프는 예외)
- `let` 또는 `const`로 선언한 속성은 어느 스코프던 삭제 불가능하다.
- 빌트인 객체의 속성들이나 `Object.defineProperty()` 같은 메서드로 만든 설정 불가능한 속성들은 삭제 불가능하다.

## 엄격 모드에서의 동작

엄격 모드에서 `delete`로 변수나 함수를 삭제하려 하면 `SyntaxError`가 발생한다.

`var`로 정의된 변수는 non-configurable로 구분되기 때문에, 
- 비엄격모드에서는 `false`를 반환한다.
- 엄격모드에서는 `SyntaxError`를 발생시킨다.

```js
function Employee() {
    delete salary;
    var salary;
}

Employee(); // false
```

```js
"use strict";
function someFunction() {

}

delete someFunction; // SyntaxError
```

## 객체의 프로토타입 체인에서의 동작

```js
function Foo() {
  this.bar = 10;
}

Foo.prototype.bar = 42;

var foo = new Foo();

delete foo.bar; // true. foo 객체의 고유 속성을 제거한다.

console.log(foo.bar); // foo.bar는 아직 프로토타입 체인에 존재하기 때문에 사용가능하다.

delete Foo.prototype.bar;

console.log(foo.bar); // 더 이상 프로토타입으로부터 inherit 받지 않기 때문에 undefined 출력
```
## 객체 요소 제거하기

배열의 원소를 삭제할 때, 배열의 길이는 영향을 받지 않는다. 마지막 원소를 제거해도 그렇다.

```js
var trees = ['redwood', 'bay', 'cedar', 'oak', 'maple'];
delete trees[3];
if (3 in trees) {
    // 실행되지 않는다.
}
```
```js
var trees = ['redwood', 'bay', 'cedar', 'oak', 'maple'];
trees[3] = undefined;
if (3 in trees) {
    // 이건 실행된다.
}
```

배열 객체가 존재하지만 undefined 값으로 놔두고 싶다면 `delete` 연산자 대신 그냥 `undefined` 값을 대입해야 한다.
