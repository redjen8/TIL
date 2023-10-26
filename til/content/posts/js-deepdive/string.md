---
title: "모던 자바스크립트 Deep Dive 32장 - 문자열 객체"
date: 2023-10-26T19:19:05+09:00
draft: false
author: redjen
---

### 생성자 함수를 통한 문자열 객체 생성

```js
const strObj = new String();
console.log(strObj); // String {length: 0, [[primitiveValue]]: ""}
```

`[[primitiveValue]]` : `[[StringData]]` 내부 슬롯에 접근할 수 있는 프로퍼티 (es5)

```js
const strObj = new String('Lee');
console.log(strObj); // String {0: "L", 1: "e", 2: "e", length: 3, [[primitiveValue]]: "Lee"}
```

> String 객체는 각 문자를 프로퍼티 값으로 가지는 유사 배열 객체이면서 이터러블한 객체


```js
const StrObj = new String(null);
console.log(strObj); // String {0: "n", 1: "u", 2: "l", 3: "l", ...}
```

요런 경우는 조심해서 사용해야 한다..

```js
String(1); // "1"
String(true); // "true"
String(NaN); // "NaN"
```

> 생성자 함수의 인수로 문자열이 아닌 값을 할당하면 인수를 문자열로 강제 변환한다.
### mutator 메서드와 accessor 메서드

> 문자열은 primitive한 값이기 때문에 변경할 수 없다. (read only)

- 문자열은 불변 (immutable)한 원시 값이다.
- 때문에 String 객체의 메서드는 **언제나 새로운 문자열을 반환한다**
- 즉 String 객체에는 원본 String 래퍼 객체를 직접 변경하는 메서드가 존재하지 않는다.

### 여러 유틸 메서드

- 대상 문자열 내 특정 **문자열**의 위치 찾을 때: `String.prototype.indexOf`
- 대상 문자열 내 특정 **문자**의 위치 찾을 때: `String.prototype.charAt`
- 대상 문자열 내 **정규식**과 매치하는 문자열 유무를 구할 때: `String.prototype.search`
- 대상 문자열 내 **문자열**의 포함 여부를 구할 때: `String.prototype.includes`
- 대상 문자열이 인수로 전달받은 문자열로 시작하는지 여부를 구할 때: `String.prototype.startsWith`
- 대상 문자열이 인수로 전달받은 문자열로 끝나는지 여부를 구할 때: `String.prototype.endsWith`
- 대상 문자열의 특정 부분을 잘라 새 문자열을 생성할 때
	- `String.prototype.substring`: 음수 인수 전달 불가
	- `String.prototype.slice`: 음수 인수 전달 시 뒤에서부터 문자열 잘라 반환
- 대상 문자열을 모두 대문자로 변환한 문자열을 생성할 때: `String.prototype.toUpperCase`
- 대상 문자열을 모두 소문자로 변환한 문자열을 생성할 때: `String.prototype.toLowerCase`
- 대상 문자열 앞 뒤 공백 문자를 제거한 문자열을 생성할 때: `String.prototype.trim`
- 대상 문자열을 인수로 전달받은 수만큼 반복해 연결한 문자열을 생성할 때: `String.prototype.repeat`
- 대상 문자열에서 특정 문자열 또는 정규식 매치되는 문자열을 치환한 문자열을 생성할 때: `String.prototype.replace`
- 대상 문자열에서 특정 문자열 또는 정규식 매치되는 문자열로 구분한 문자열 배열을 생성할 때: `String.prototype.split`

### 자바스크립트에서 문자열은 어떻게 메모리에 저장될까?

https://levelup.gitconnected.com/bytefish-vs-new-string-bytefish-what-is-the-difference-a795f6a7a08b

여러 차례 언급되었지만, 문자열은 불변의 원시값이다.

그리고 이 원시값 데이터 자체는 문자열 상수 풀(string constant pool)이라는 메모리 공간에 할당되어 레퍼런스된다. 
	
(위 블로그 글에서는 마치 상수 풀이 별도의 메모리 공간인 것처럼 그려놨지만 사실 힙 메모리 안에 상수 풀이 존재한다)

하지만 문자열 래퍼 객체 (`String()`)은 원시값을 레퍼런스 하는 객체이다. 때문에 다음과 같은 결과가 나오는 것.. 🤮
```js
let str1 = 'hello';
let str2 = new String('hello');
let str3 = 'hello';
let str4 = new String('hello');

console.log(str1 === str3); // true
console.log(str2 === str4); // false
console.log(str2.valueOf() === str4.valueOf()); // true
```

#### 난 여태까지 primitive한 string으로도 String 객체의 메서드를 잘 써왔는데..?

https://stackoverflow.com/questions/5751704/javascript-do-primitive-strings-have-methods

그렇다면, 다음의 코드가 동작하는게 의아할 수 있다.
```js
let str = 'hello';
console.log(str.charAt(1)); // 'e';
```

- `str` 변수에는 원시값 문자열이 할당되었다.
- 객체의 프로토타입 메서드는 오직 해당 객체에서만 호출 가능하다.
- `String` 객체의 프로토타입 메서드인 `charAt`을 호출할 수 없어야 하는 거 아닐까?

실제로 원시값 문자열은 메서드들을 가지고 있지 않다. 그냥 힙 메모리 공간의 일부분인 문자열 상수 풀의 한 주소를 가지고 있을 뿐이다.

> 자바스크립트 런타임은 원시값 문자열에 대해 `String` 객체의 메서드들을 호출하는 시점에 원시값을 래핑한 객체로 promote 시킨다. 

그렇기 때문에 여러 문자열 메서드들을 마치 원시값 문자열에서도 사용할 수 있는 것처럼 보였던 것이다.
