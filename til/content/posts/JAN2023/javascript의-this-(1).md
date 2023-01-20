---
title: "Javascript의 This (1)"
date: 2023-01-20T16:56:13+09:00
draft: false
author: redjen
---

Javascript에서 `this` 키워드는 다른 언어와 달리 호출한 위치에 따라서 가르키는 대상이 달라진다.
스크립트가 실행 중일 때에는 할당해서 설정할 수 없고, 호출할 때마다 대상이 바뀐다. 
(`bind`를 통해 `this`가 가르키는 대상을 고정할 수 있긴 하다)

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this

그럼 호출하는 맥락에서 `this`가 가르키는 대상이 어떻게 바뀔까?

## 전역 문맥에서의 this

전역 실행 맥락에서 `this`는 엄격 모드에 상관 없이 전역 객체를 참조한다.
브라우저 내라면 `window`, nodejs 내라면 `global`을 가르킨다.

## 함수 문맥에서의 this

### 전형적인 경우

`this`의 값은 함수가 접근하는 객체이다.
`obj.f()`에서 함수 호출이 이뤄졌다면 `this`는 `obj`를 가르킨다.

> `this`의 값은 함수를 자신의 property로 가지는 객체가 아니라 함수를 호출하기 위해 사용되는 객체이다.

아래 예제에서 잘 확인할 수 있었다.

```javascript
function getThis() {
    return this;
}

const obj1 = { name : "obj1" };
const obj2 = { name : "obj2" };

const obj3 = {
    __proto__: obj1,
    name: "obj3",
};

const obj4 = {
    name: "obj4",
    getThis() {
        return this;
    },
};

const obj5 = { name: "obj5" };

obj1.getThis = getThis;
obj2.getThis = getThis;

console.log(obj1.getThis()); //{ name: 'obj1', getThis: [Function: getThis] }
console.log(obj2.getThis()); //{ name: 'obj2', getThis: [Function: getThis] }
console.log(obj3.getThis()); //{ name: 'obj3' }

obj5.getThis = obj4.getThis;
console.log(obj5.getThis()); //{ name: 'obj5', getThis: [Function: getThis] }
```

엄격 모드에서
- `this`는 primitive한 값에 대해 호출되면 `this`의 값도 primitive한 값이 된다.
- 어떤 객체에도 접근되지 않은 상태에서 `this`를 호출하면 `undefined` 값을 가진다.

비엄격 모드에서
- `this`를 호출한 함수가 `undefined`나 `null`이 된다면 `this`는 `globalThis`로 교체된다.
- primitive한 값에 대한 `this` 호출은 해당 primitive 값에 대한 래퍼 객체에 대한 `this` 호출로 교체된다.

보통 함수 호출을 할 때, `this`는 암묵적으로 함수의 prefix에 붙어서 전달되는 파라미터처럼 전달된다.
명시적으로 `this`에 대한 값을 설정하기 위해서는 `Function.prototype.call()`, `Function.prototype.apply()`, `Reflect.apply()`를 사용할 수 있다.

`Function.prototype.bind()`를 사용하면 함수가 어떻게 호출되던 특정 값을 가지는 `this`를 가지는 함수를 생성할 수 있다.

상기 메서드들을 사용할지라도 비엄격 모드에서의 `this` 교체 규칙들은 여전히 지켜진다. 

### 콜백 함수에서의 this

함수가 콜백으로 전달될 때 `this`의 값은 콜백이 어떻게 호출되는지에 따라 달라진다.
다른 객체에 붙이지 않고 사용되는 전형적인 콜백은 `this` 값으로 `undefined`를 가진다.
이 때 콜백 함수는 비엄격모드이고, `this` 값이 전역 객체인 `globalThis`이다.

```javascript
function logThis() {
    "use strict";
    return this;
}

[1, 2, 3].forEach(logThis); //undefined, undefined, undefined
setTimeout(logThis, 1000); //undefined
```

일부 API들은 콜백 함수들에 대해 `this` 값을 설정할 수 있게 해준다. (`Set.prototype.forEach()`와 같은 API)
```javascript
[1, 2, 3].forEach(logThis, { name: "obj" }) // { name: "obj" }, { name: "obj" }, { name: "obj" }
```

## 화살표 함수에서의 this

화살표 함수에서 `this`는 둘러싸고 있는 lexical context에서의 `this` 값을 가진다. 
다른 말로는, 화살표 함수의 body를 evaluate 할 때 자바스크립트는 새 `this`에 대한 바인딩을 생성하지 않는다.

```javascript
const globalObject = this;
const foo = () => this;
console.log(foo() === globalObject); // true
```

- 화살표 함수들은 `this` 값을 둘러싸고 있는 스코프에 대한 클로저를 만든다.
- 화살표 함수들은 "자동 바인딩" 인 것처럼 동작한다.
- 화살표 함수가 호출되는 방법과는 무관하게 `this` 값은 함수가 생성되었을 때의 상태로 설정된다. (상기 예제에서는 `globalObject`)

`call()`, `bind()`, `apply()`를 사용하는 화살표 함수의 경우 `thisArg` 파라미터는 무시되지만 해달 메서드들을 통해 다른 파라미터는 여전히 전달된다.

```javascript
const obj = { name: "obj" };

console.log(foo.call(obj) === globalObject); //true

const boundFoo = foo.bind(obj);
console.log(boundFoo === globalObject); //true
```
