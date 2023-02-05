---
title: "Optional Chaining 이란"
date: 2023-02-05T13:53:44+09:00
draft: false
author: redjen
---

자바스크립트를 사용해서 개발하다보면 `undefined`나 `null`에 대한 검사를 계속해서 하게 되는 경우가 많다.
좀 더 편하게 `nullish` 한지 체크하기 위해서 Optional Chaining이란 개념을 알게 되었다.

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining

## 기본 사용법

Optional chaining은 `?.` 연산자를 사용한다. 

`.` 체이닝 연산자와 유사하게 작동하지만, 이전 참조가 `null` 또는 `undefined` 일 경우에는
- 에러가 발생하는 것 대신 (`.` 연산자를 사용할 경우)
- 표현식의 리턴 값을 `undefined`로 바꾼다.
- 함수 호출에서 주어진 함수가 존재하지 않는 경우 `undefined`를 리턴한다.

선언되지 않은 루트 객체에 `?.`를 사용할 수는 없지만, 정의되지 않은 루트 객체에는 사용할 수 있다.

## 의의와 동작

Optional Chaining 연산자는 참조가 기능이 `nullish`일 수 있을 때 사용하며, 연결된 객체의 값에 접근해서 에러를 발생하는 대신 단순화하는 방법을 제공하는 것에 의의가 있다.

중첩된 구조를 가진 객체 `obj`가 다음과 같이 사용된다고 가정하자.
`let nestedProp = obj.first && obj.first.second;`

`obj.first` 값은 `obj.first.second` 값에 접근하기 전에 `null`이 아니라는 것을 검증히기 위한 코드이다.
만약 `obj.first` 값에 대한 nullish 테스트 없이 사용할 때 발생할 수 있는 에러를 막기 위한 코드인 것이다.

위 예제는 optional chaining을 사용하지 않은 케이스이다. 만약 optional chaining을 사용한다면 아래와 같이 바뀔 수 있다.
`let nestedProp = obj.first?.second;`

`.` 대신에 `?.` 연산자를 사용해서 `obj.first.second`에 접근하기 전에 `obj.first` 상태에 따라 명시적으로 nullish를 테스트하지 않아도 된다는 간편함이 있다.
`obj.first`가 `null` 또는 `undefined`이라면 `obj.first.second` 표현식은 자동으로 `undefined`가 된다.

### 함수의 호출과 Optional chaining

존재하지 않는 함수를 호출할 때 optional chaining을 사용할 수 있다.
함수 호출에서 optional chaining을 사용하면 찾을 수 없는 함수에 대해 에러를 발생시키지 않고 자동으로 `undefined`를 반환하도록 할 수 있다.

```javascript
function doSomething(onContent, onError) {
    try {
        // 특정 작업 수행
    }
    catch (err) {
        if (onError) { // onError가 실제로 존재할 때의 테스트
            onError(err.message);
        }
    }
}
```
위 코드는 Optional chaining을 사용한다면 아래와 같이 바뀔 수 있다.

```javascript
function doSomething(onContent, onError) {
    try {
        // 특정 작업 수행
    }
    catch (err) {
        onError?.(err.message); // onError가 undefined여도 에러가 발생하지 않음
    }
}
```

### 표현식에서의 Optional chaining

아래 예시처럼 optional chaining 연산자를 속성의 표현식으로 접근할 때 대괄호 표기법을 사용할 수 있다.
`let nestedProp = obj?.['prop' + 'Name'];` 
