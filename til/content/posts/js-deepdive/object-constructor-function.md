---
title: "모던 자바스크립트 Deep Dive 17장 - 생성자 함수에 의한 객체 생성"
date: 2023-08-22T22:24:08+09:00
draft: false
author: redjen
---

## Object 생성자 함수

`new` 연산자 + `Object` 생성자 함수 호출 -> 빈 객체 생성해서 반환

JS는 `String`, `Number`, `Boolean`, `Function`, `Array`, `Date`, `RegExp`, `Promise` 등의 빌트인 생성자 함수 제공

객체 리터럴을 사용하는 것이 더 간편해서 별 이점 X

### 생성자 함수

#### 객체 리터럴에 의한 객체 생성 방식의 문제점

> 단 하나의 객체만 생성할 수 있다.

때문에 똑같은 프로퍼티를 가지는 객체 여러 개 생성할 때 비효율적

- 객체의 상태 = 프로퍼티
- 객체의 상태 데이터를 참조 및 조작 = 메서드

따라서 객체마다 프로퍼티는 다를 수 있지만 메서드는 동일한 경우가 일반적

#### 생성자 함수에 의한 객체 생성 방식의 장점

> 프로퍼티 구조가 동일한 객체 여러 개를 간편하게 생성할 수 있다.

객체 인스턴스를 생성하기 위한 템플릿처럼 생성자 함수를 사용할 수 있다.
- 클래스 기반 객체 지향 언어의 생성자와는 다르게 형식이 정해져 있지 않다
- 일반 함수와 동일한 방법으로 생성자 함수를 정의 가능
- `new` 연산자와 함께 호출하면 해당 함수는 생성자 함수로 동작

#### 생성자 함수의 인스턴스 생성 과정

> 생성자 함수는 1) 인스턴스 생성과 2) 생성된 인스턴스를 초기화 시키는 역할을 한다.

```js
function Circle (radius) {
   this.radius = radius;
   this.getDiameter = function () {
      return 2 ** this.radius;
   };
}

const circleInstance = new Circle(5);
```

js 엔진은 암묵적인 처리를 통해 인스턴스를 생성하고 반환하는 점에 유의.
(인스턴스를 명시적으로 생성하고 반환하지 않는다)

인스턴스 생성 -> 초기화 -> 반환은 다음의 과정을 거친다.
##### 1. 인스턴스 생성과 this 바인딩

- 암묵적으로 빈 객체가 생성된다.
- 이 빈 객체는 this에 바인딩된다.
   - 이 처리는 함수 내부 코드가 실제로 한 줄씩 실행되는 런타임 이전에 실행된다.

##### 2. 인스턴스 초기화

생성자 함수에 기술된 코드가 실제로 한 줄씩 실행되어 this에 바인딩되어 있는 인스턴스를 초기화한다.
- 개발자가 작성하는 부분이다.
- 개발자가 프로퍼티 및 메서드를 추가한다.

##### 3. 인스턴스 반환

생성자 함수의 모든 처리가 끝나면 완성된 인스턴스가 바인딩된 this를 암묵적으로 반환한다.
즉 this가 아닌 
- 다른 객체를 명시적으로 반환하면 return 문에 명시된 객체가 반환된다.
- primitive를 명시적으로 반환하면 무시되고, this가 암묵적으로 반환된다.
- 뭐가 됐던 생성자 함수 내에서 다른 값을 반환하는 것은 지양해야

```js
function Circle (radius) {
   this.radius = radius;
   this.getDiameter = function () {
      return 2 * this.radius;
   };
   return {};
}

const circleInstance = new Circle(5);
console.log(circleInstance); // {}
```

#### 함수 호출의 내부 동작

함수는 특별한 객체
- 일반 객체는 호출할 수 없지만 함수는 호출이 가능하다 (Callable)
- 따라서 함수는 일반 객체가 가지고 있는 내부 슬롯 + 메서드에 추가적으로 함수 객체만 가지는
   - `[[Environment]]`, `[[FormalParameters]]` 등의 내부 슬롯과
   - `[[Call]]`, `[[Construct]]`와 같은 내부 메서드를 추가로 가진다.

함수는 내부적으로
1. 일반 함수로써 호출되면 내부 메서드 `[[Call]]`이 호출된다
   1. 이런 함수 객체를 callable이라 부른다
2. `new` 연산자와 함께 호출되면 내부 메서드 `[[Construct]]`가 호출된다
   1. 이런 함수 객체를 constructor라 부른다
   2. 내부 메서드 `[[Construct]]`를 가지지 않는 함수 객체, 즉 생성자 함수로써 호출할 수 없는 함수를 non-constructor 이라 부른다

> 모든 함수는 호출 가능하다 (Callable). 
> 
> 따라서 모든 함수 객체는 내부 메서드 `[[Call]]`을 가진다. 
> 
> 하지만 모든 함수 객체가 `[[Construct]]`를 가지지는 않는다.

#### constructor와 non-constructor의 구분

ECMAScript 사양에서는
- (1) 함수 선언문 (2) 함수 표현식으로 정의된 함수만 constructor이다.
   - 따라서 주의할 점은 이런 함수에 `new` 연산자를 붙여 호출하면 생성자 함수처럼 동작할 수 있다
- (1) ES6의 화살표 함수와 (2) 메서드 축약 표현으로 정의된 함수는 non-constructor이다.
   - 따라서 이런 함수를 생성자 함수로써 호출하면 에러 발생

```js
function add(x, y) {
   return x + y;
}
// 생성자 함수가 아닌 (값을 반환하는) 일반 함수

let inst = new add(); // 일반 함수를 생성자 함수처럼 동작 시킬 수 있음에 유의

console.log(inst); // {}

function createUser(name, role) {
   return [name, role];
}
// 객체를 반환하는 일반 함수

inst = new createUser('Lee', 'admin'); // 객체를 반환하는 함수이기 때문에 생성자로 기능

console.log(inst); // {name: "Lee", role: "admin"}

function Circle(radius) {
   this.radius = radius;
   this.getDiameter = function () {
      return 2 * this.radius;
   }
}

const circle = Circle(5); // [[Construct]]가 호출되는 것이 아니라 [[Call]]이 호출된다

console.log(circle); // undefined
console.log(radius); // 5
console.log(getDiameter()); // 10
circle.getDiameter(); // TypeError
```

#### new.target

IE에서는 지원 X

> 함수 내부에서 new.target을 사용하면 함수가 new 연산자와 같이 생성자 함수로써 호출되었는지 확인할 수 있다.

- `new` 연산자와 함께 생성자 함수로써 호출되면 함수 내부의 `new.target`은 함수 자신을 가르킨다.
- `new` 연산자 없이 일반 함수로써 호출된 함수 내부의 `new.target`은 undefined

```js
function Circle(radius) {
   if (!(this instanceof Circle)) {
      return new Circle(radius);
   }
   // new와 함께 호출되지 않았을 때의 방어로직

   this.radius = radius;
   this.getDiameter = function () {
      return 2 * this.radius;
   }
}
```

대부분의 빌트인 생성자 함수는 `new` 연산자와 함께 호출하지 않아도 동일하게 동작한다 (착한 js)

그런데 아래 빌트인 생성자 함수는 `new` 연산자와 함께 안 쓰면 다른 값을 반환한다 (나쁜 js)

```js
const str = String(123);
console.log(str, typeof str); // 123 string

const num = Number('123');
console.log(num, typeof num); // 123 number

const bool = Boolean('true');
console.log(bool, typeof bool); // true boolean
```