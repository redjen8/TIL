---
title: "Javascript Spread Operator에 대해"
date: 2023-01-25T20:32:27+09:00
draft: false
author: redjen
---

## 개요
Typescript로 `Map<string, someObject>` 형태의 데이터를 가공하며 고민을 해봤던 기록이다. 
아래 코드는 `someObject` 안의 `boolean` 타입 필드로 필터링한 데이터의 키 값을 배열로 가져오기 위한 코드이다.

```typescript
class testObject {
  constructor(n1: string, n2: boolean) {
    this.nestedKey1 = n1;
    this.nestedKey2 = n2;
  }
  nestedKey1: string;
  nestedKey2: boolean;
}

let testMap = new Map<string, testObject>;
testMap.set("test1", new testObject("data1", true));
testMap.set("test2", new testObject("data2", false));
testMap.set("test3", new testObject("data3", true));

let res: string[] = new Array<string>;
testMap.forEach((v, k) => {
    if (v.nestedKey2 === true) {
        res.push(k)
    }
})

console.log(res);
```

하단의 `forEach`문은 다음과 같이 바뀔 수 있다.
```typescript
console.log(
    [...testMap.entries()]
    .filter((entry) => entry[1].nestedKey2)
    .map((entry) => entry[0])
);
```
```typescript
console.log(
    Array.from(testMap.entries())
    .filter((entry) => entry[1].nestedKey2)
    .map((entry) => entry[0])
);
```

결과적으로는 세 개의 코드 모두 동일한 결과인 `["test1", "test3"]`을 출력한다.
`forEach`를 사용하지 않고 깔끔하게 함수형으로 프로그래밍을 짤 수 있게 해줬던 spread operator에 대해 조사해봤다.

## Spread operator란?

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax

spread 문법은 배열이나 `string` 같은 iterable들에 대해,
- 함수 호출할 경우 0개 이상의 인수,
- 배열 리터럴의 경우 배열의 원소로 확장 변환하여
- 0개 이상의 key-value 쌍 객체로 바꿀 수 있다.

이렇게만 쓰면 무슨 말인지 잘 와닿지가 않는다. 좀 더 자세히 알아보자.

## 언제 사용하는가?

Spread operator는 다음과 같은 경우에 사용한다.
- 객체나 배열에 있는 모든 요소들이 새로운 객체나 배열에 포함되어야 할 때
- 객체나 배열에 있는 모든 요소들이 함수 호출의 인수 목록에 1 by 1으로 적용되어야 할 때

즉, Spread syntax에는 서로 다른 3가지의 용법이 존재한다.
1. 함수의 인자 목록으로 사용 (`myFunction(a, ...iterableObj, b)`)
2. 배열 리터럴 (`[1, ...iterableObj, '4', 'five', 6]`)
3. 객체 리터럴 (`{ ...obj, key: 'value'}`)

배열과 같은 iterable한 객체는 array나 함수 인자에서 spread 될 수 있다.
하지만 대부분의 객체는 iterable하지 않다. (`Symbol.iterator`를 포함하지 않는 단순한 객체가 그렇다)

한편으로는 객체 리터럴에서 spread 될 때 spread 연산자는 객체의 속성을 enumerate한다.
전형적인 배열에서 모든 인덱스들은 enumerable한 고유 속성이기 때문에, 배열은 객체 안으로 spread 될 수 있다. (array to object)
```javascript
const array = [1, 2, 3];
const obj = { ...array }; // { 0: 1, 1: 2, 2: 3 }
```

함수 호출에서 spread 문법을 사용할 때에는 자바스크립트 엔진의 인수 길이 제한을 넘지 않도록 조심해야 한다.

## 사용 예시

#### 함수 호출에서의 spread 연산자
```javascript
function myFunction(x, y, z) {}
const args = [0, 1, 2];
myFunction.apply(null, args);
myFunction(...args); // 동일한 효과를 가진다
```
spread 연산자는 `new` 연산자와 함께 사용될 수도 있다. 

`apply()`가 대상 함수를 생성하는 대신에 호출하기 때문에 `apply()`를 사용하면 `new.target`은 항상 `undefined`가 된다. 그렇기 때문에 일반적으로 `new`와 함께 생성자를 호출할 때에는 배열과 `apply()`를 직접적으로 사용할 수 없다.

하지만 spread 연산자와 함께 사용한다면 `new`로도 배열을 통해 객체를 생성할 수 있다.
```javascript
const dateFields = [1970, 0, 1]; // 1 Jan 1970
const d = new Date(...dateFields);
```

### 배열 리터럴에서의 spread 연산자

```javascript
const parts = ['shoulders', 'knees'];
const lyrics = ['head', ...parts, 'and', 'toes'];
// ["head", "shoulders", "knees", "and", "toes"]
```

spread 연산자를 사용하면 배열을 복사할 수도 있다.
```javascript
const arr = [1, 2, 3];
const arr2 = [...arr];

arr2.push(4);
console.log(arr); // [1, 2, 3]
console.log(arr2);  // [1, 2, 3, 4]
```

> spread 연산자를 사용하여 배열을 복사할 때 1 depth의 데이터에 대한 깊은 복사가 이루어진다는 것에 주의해야 한다.
> 
> 때문에 다차원 배열을 복사할 때에는 적합하지 않다.
> - `Object.assign()` 또한 1 depth의 데이터만 복사한다.
> - Javascript에서 네이티브하게 deep copy를 할수 있는 연산자는 제공되지 않는다.
> - Web API에는 `structuredClone()`이 제공되긴 한다.

spread 연산자는 `concat`을 사용하지 않고 배열을 붙일 수 있게 해준다.
```javascript
let arr1 = [0, 1, 2];
const arr2 = [3, 4, 5];

arr1 = arr1.concat(arr2);
arr1 = [...arr1, ...arr2]; // 동일한 효과를 가진다
```

`concat()`을 사용하는 방법과 `spread` 연산자를 사용하는 방법의 차이가 궁금해서 찾아봤더니 [참고한 링크](https://stackoverflow.com/questions/48865710/spread-operator-vs-array-concat)

- 함수의 인자가 배열이 아닐때 동작이 완전히 달라짐
  - 문자열은 iterable, iterable한 성질을 가지는 문자열에 대해 spread 연산자를 적용하면 글자 하나 하나 분리됨
  - `concat()`은 iterate하려는 시도를 하지 않는다. 
- 성능적인 측면에서 `concat()`은 배열에 특정적인 최적화를 시도할 수 있기 때문에 조금 더 빠르다.
  - spread 연산자는 iteration을 통해 접근하기 때문에 최적화의 부재가 발생한다.

### 객체 리터럴에서의 spread 연산자

spread 연산자를 사용하면 `Object.assign()`을 사용하지 않고도 객체의 얕은 복사나 병합이 가능하다.
```javascript
const obj1 = { foo: 'bar', x: 42 };
const obj2 = { foo: 'baz', y: 13 };

const cloneObj = { ...obj1 }; // { foo: "bar", x: 42 }
const mergedObj = { ...obj1, ...obj2 } // { foo: "baz", x: 42, y: 13 }
```

하지만 다음과 같은 경우에서는 `Object.assign()`을 사용해야 한다.
- 객체를 mutate해야 할 때
- 대상 객체의 setter를 사용해야 할 때

때문에 spread 연산자를 마구 사용하는 것은 예상하지 못한 결과를 초래할 수 있다.
잘 알아보고 사용하자.
