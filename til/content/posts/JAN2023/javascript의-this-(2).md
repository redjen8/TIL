---
title: "Javascript의 This (2)"
date: 2023-01-21T16:49:14+09:00
draft: false
author: redjen
---

## 생성자에서의 this

함수가 `new` 키워드를 통해서 생성자로 사용될 때, `this`는 새로 생성되는 객체를 가르킨다.
이 때, 생성자 함수가 접근하는 객체가 어떤 객체인지는 상관 없이 이루어진다.
`this`의 값은 생성자가 다른 non primitive한 값을 반환하지 않는다면 `new` 표현식에 대한 값이 된다.

```javascript
function C() {
    this.a = 37;
}

let o = new C();
console.log(o.a); //37

function C2() {
    this.a = 37;
    return { a : 38 };
}

o = new C2();
console.log(o.a); // 38
```

상기 예제의 `C2` 함수에서는 생성 도중에 객체가 반환되었기 때문에 `this`에 바인딩된 새 객체가 무시되었다. 이 때문에 `this.a = 37` 코드가 쓸모 없게 되었다. 엄밀히 말하면 실행 자체는 이뤄지기 때문에 죽은 코드는 아니지만 아무런 side effect 없이 제거할 수 있다.

### super

함수가 `super.method()` 형식으로 호출될 때, `method` 함수 안에 들어가 있는 `this`의 값은 `super.method()` 호출 근처의 `this` 값과 동일하다.
이는 `super.method`가 지금까지 살펴봤던 종류의 객체의 멤버가 아니고 다른 바인딩 규칙에 따른 특별한 문법이기 때문이다.

## Class 컨텍스트에서의 this

클래스는 static과 instance 두 컨텍스트로 나눠질 수 있고, `this`의 값은 각 컨텍스트에서 달라진다. 
- 생성자, 메서드, 인스턴스 필드 초기화 키워드 (`public`과 `private`) 는 instance 컨텍스트에 속한다.
- static 메서드, static 필드 초기화 키워드, static 초기화 블록들은 static 컨텍스트에 속한다. 

클래스 생성자는 항상 `new` 키워드를 통해 호출된다. 때문에 클래스 생성자는 직전에 함수 생성자에서의 역할과 동일하게 `this` 값을 새로 생성되는 인스턴스로 가르킨다.

클래스 메서드들은 오브젝트 literal에서의 메서드들처럼 동작한다. 때문에 `this` 값이 메소드가 접근하는 객체의 값이 된다. 만약 메서드가 다른 객체로 전이되지 않으면, `this` 값은 클래스의 인스턴스를 가르킨다.

정적 메서드들은 `this`의 속성들이 아니다. 정적 메서드들은 클래스 자체의 속성들이다.
그렇기 때문에 정적 메서드들은 클래스에 의해 보통 접근되고, `this` 값은 클래스의 값을 가르킨다.
static 초기화 블록들은 현재 클래스에서 설정된 `this` 값과 함께 evaluate 된다.

필드 초기화 키워드들 (field initializer)은 클래스 컨텍스트 안에서 evaluate 된다.
인스턴스 필드들은 생성하는 인스턴스의 `this` 값과 함께 evaluate 된다.
static 필드들은 현재 클래스의 `this` 값과 함께 evaluate 된다.
그리고 이것은 필드 초기화 과정에서 화살표 함수들이 클래스에 바운드 되는 이유이다.

```javascript
class C {
    instanceField = this;
    static staticField = this;
}

const c = new C();
console.log(c.instanceField === c); // true
console.log(C.staticField === C); // true
```

### 파생 클래스 생성자들에서의 this

기반이 되는 클래스 생성자와는 달리, 파생 클래스 생성자에서는 처음에 `this` 값에 대한 바인딩이 존재하지 않는다.
`super()`를 호출하게 되면 그제서야 생성자 내에서 `this` 바인딩을 생성되고, 기본적으로는 다름 코드 line을 evaluate한다.

파생 클래스들은 아래 두 가지 경우를 제외하고는 `super()`를 호출하기 전에 리턴되어서는 안된다.
1. 생성자가 객체를 리턴하지 않거나 (`this` 값이 덮어씌워지는 경우)
2. 클래스가 애초에 생성자를 가지지 않는 경우

```javascript
class Base {}
class Good extends Base {}
class AlsoGood extends Base {
    constructor () {
        return { a : 5};
    }
}

class Bad extends Base {
    constructor () {}
}

new Good();
new AlsoGood();
new Bad(); // ReferenceError 발생. 
// Must call super constructor in derived class before accessing 'this'
// or returning from derived constructor
```

## 전역 컨텍스트에서의 this

전역 '실행 컨텍스트'에 해당하는 경우는 몇 가지가 있다.
- 어떤 함수나 클래스 안에 위치하지 않을 때
- 전역 스코프에서 정의된 블럭 안에 있을 때
- 전역 스코프에서 정의된 화살표 함수 안에 있을 때

전역 컨텍스트 안에서는 `this` 값이 스크립트가 실행되는 실행 컨텍스트 안에 따라 달라진다.
콜백 함수처럼, `this` 값은 caller와 같은 런타임 환경에 따라 달라진다.

스크립트의 최상위 레벨에서는 `this`는 `globalThis`를 가르킨다. (엄격 / 비엄격 모드는 상관 X) 이 때는 전역 객체에서의 `this`와 같은 경우로, 예를 들어 소스코드가 HTML의 `<script>` 엘리멘트 안에서 삽입되어 스크립트로 실행된 (`this === window`) 경우이다. 

`globalThis`는 보통 전역 객체와 동일한 컨셉이다. (예를 들어 `globalThis`에 속성을 추가하는 것은 전역 객체에 속성을 추가하는 것과 동일하다) 그런데 이는 브라우저와 node에서만 그렇고, 호스트는 전역 객체에 대해 관련이 없기 때문에 `globalThis`에 대해 다른 값을 제공할 수 있다.

```javascript
console.log(this === window); // 웹 브라우저에서 true

this.b = "MDN";
console.log(window.b); // "MDN"
console.log(b); // "MDN"
```

- 소스코드가 module로 불러와진다면 (`<script>`에서 `type="module"`를 통해 추가되는 경우)`this`는 항상 최상위 레벨에서 `undefined`이다.
- 소스코드가 `eval()`로 실행된다면, indirect eval에 대해 `this` 값은 direct eval을 둘러싸는 컨텍스트와 동일하거나 별도의 전역 스크립트에서 실행되는 것처럼 `globalThis`와 동일하다.

```javascript
function test() {
    console.log(eval("this") === this); // Direct eval
    console.log(eval?.("this") === globalThis); // Indirect eval, 비엄격 모드
    console.log(eval?.("'use strict'; this") === globalThis); // Indirect eval, 엄격 모드
}

test.call( {name: "obj" }); // 3개의 true를 출력
```

전역 스코프처럼 보이는 일부 소스 코드는 실제로 실행될 때 함수에 래핑되어 실행되는 경우도 있기 때문에 헷갈린다. 
- Nodejs의 CommonJS 모듈은 함수에 래핑되어 실행되고 `this`값을 `module.exports`로 설정한다.
- 이벤트 핸들러 attribute 들은 attach되는 요소에 대한 `this` 값으로 실행된다.

객체 리터럴들은 `this` 스코프를 생성하지 않는다.
객체 안에 정의된 함수 (메서드) 들만 `this` 스코프를 생성한다.

객체 리터럴 안에서 `this`를 사용할 때에는 둘러싸고 있는 스코프의 `this` 값을 상속받는다.

```javascript
const obj = {
    a: this,
};

console.log(obj.a === window); // true
```
