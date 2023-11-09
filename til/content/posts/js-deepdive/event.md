---
title: "모던 자바스크립트 Deep Dive 40장 - 이벤트"
date: 2023-11-09T19:19:15+09:00
draft: false
author: redjen
---

### 이벤트 드리븐 프로그래밍

사용자와의 상호작용을 더 풍부하게 하기 위해 브라우저는 다음의 시퀀스를 지원 
- 처리할 사건이 발생했음을 인지
- 특정 타입의 이벤트 발생

특정 타입의 이벤트에 반응해서 어떤 일을 하고 싶다면 이벤트 핸들러를 등록해야 함

> 이벤트가 발생했을 때 호출할 함수를 이벤트 핸들러라고 한다.

> 이벤트가 발생했을 때 **브라우저에게 이벤트 핸들러의 호출을 위임**하는 것을 이벤트 핸들러 등록이라 한다.

시퀀셜한 프로그래밍 방식에서는 사용자와 언제 상호작용을 할 지 알 수 없기 때문에 **언제 함수를 호출해야 하는지**를 모른다.

하지만 브라우저가 이벤트 핸들러 등록을 통해 이벤트가 발생했을 때 행동할 내용을 사전에 정의함으로써, 상호작용이 발생함을 기다리지 않고 상호작용이 발생한 즉시 프로그램의 흐름을 제어할 수 있다.

즉 프로그램의 흐름을 이벤트 중심으로 제어하는 프로그래밍 방식을 이벤트 드리븐 프로그래밍이라 한다.

### 이벤트 타입

종류가 굉장히 많다

상세 내용은 https://developer.mozilla.org/en-US/docs/Web/API/Event 참고

### 이벤트 핸들러 등록

#### 1. 이벤트 핸들러 어트리뷰트 방식

HTML 엘리먼트의 어트리뷰트 중에는 이벤트에 대응하기 위한 이벤트 핸들러 어트리뷰트가 있다.

요러한 어트리뷰트의 특징은 `on` 접두사를 가지고, 이벤트 타입이 그 후에 붙는다.

즉 `onclick` 어트리뷰트는 클릭했을 때의 이벤트에 대응하는 이벤트 핸들러 어트리뷰트라고 짐작할 수 있다.

```html
<!DOCTYPE html>
<html>
<body>
   <button onclick="sayHi('Lee')">Click me!</button>
   <script>
      function sayHi(name) {
         console.log(`Hi! ${name}.`);
      }
   </script>
</body>
</html>
```

주의할 점은 이벤트 핸들러 어트리뷰트 값으로는 **함수 호출문 등의 statement를 할당한다**는 것이다.

- 이벤트 핸들러 등록 === 함수 호출을 브라우저에게 위임
- 함수 호출문을 이벤트 핸들러로 등록하면 함수 호출문의 eval 결과가 이벤트 핸들러로 등록된다
- 값을 반환하는 함수 호출문을 이벤트 핸들러로 등록하면 브라우저는 호출할 함수가 아닌 값을 참조하게 되어 동작하지 않는다
- 때문에 함수 참조를 등록해야 브라우저가 이벤트 핸들러를 호출할 수 있다.

앗 그런데 위 예시에서는 어트리뷰트 값으로 함수 호출문 `sayHi('Lee')`를 썼는데?
- 사실 이 때 이벤트 핸들러 어트리뷰트 값은 암묵적으로 생성될 이벤트 핸들러의 함수 body를 의미한다.
- 즉 `onclick = "sayHi('Lee')"` 어트리뷰트는 파싱되어서 다음의 함수를 암묵적으로 생성한 후  이벤트 핸들러 어트리뷰트 이름과 동일한 `onclick` 이벤트 핸들러 프로퍼티에 할당한다.
```js
  function onclick(event) {
     sayHi('Lee');
  }
```

왜 암묵적으로 함수를 생성하는 일을 할까? (사실 자바스크립트에서 함수를 암묵적으로 생성하는 일은 많긴 하지만)
- 이벤트 핸들러 등록 === 함수 호출을 브라우저에게 위임
- 브라우저는 즉 호출할 함수의 참조 값을 가지고 있어야 한다
- 그런데 함수 참조 값을 넘기는 건 좋은데, 인수를 어떻게 전달하지?

```html
<button onclick="sayHi">Click me!</button>
```

> 즉 이벤트 핸들러 어트리뷰트 값으로 할당한 문자열은 암묵적으로 생성되는 이벤트 핸들러의 함수 몸체가 된다.

하지만 HTML과 자바스크립트는 [관심사가 다르다](https://en.wikipedia.org/wiki/Unobtrusive_JavaScript). 
- 때문에 **HTML + Vanlia JS를 사용한 웹 개발에서는 이를 분리하는 것이 좋다.**
- 프레임워크 / 라이브러리를 사용한 웹 개발에서는 HTML, CSS, JS를 서로 다른 요소가 아닌 뷰를 구성하기 위한 요소로 본다
   - 관심사가 컴포넌트를 구성하기 위한 것이라는 점에서 똑같다.
   - 때문에 **프레임워크 / 라이브러리를 사용한 웹 개발에서는 이벤트 핸들러 어트리뷰트를 사용해서 이벤트를 처리한다.**

앵귤러에선..
```
<button (click) = "handleClick($event)">Save</button>
```

#### 2. 이벤트 핸들러 프로퍼티 방식

DOM 노드 객체는 이벤트 핸들러 프로퍼티를 가지고 있다. 

이벤트 핸들러 프로퍼티의 키는 어트리뷰트와 마찬가지로 `on` 접두사 + 이벤트 타입으로 구성된다.

```html
<!DOCTYPE html>
<html>
<body>
   <button>Click me!</button>
   <script>
      const $button = document.querySelector('button');
      $button.onclick = function () {
         console.log('click!');
      };
   </script>
</body>
</html>
```

- 이벤트 핸들러는 대부분 이벤트를 발생 시킬 이벤트 타깃에 바인딩한다.
- 하지만 반드시 이벤트 타깃에 이벤트 핸들러를 바인딩해야 하는 것은 아니다.
- 이벤트 핸들러는 이벤트 타깃 또는 전파된 이벤트를 캐치할 DOM 노드 객체에서 바인딩한다.

앞서 살펴본 이벤트 핸들러 어트리뷰트 방식과 거의 동일하지만,
- HTML과 JS가 섞이는 문제를 해결 할 수 있다.
- 하지만 이벤트 핸들러 프로퍼티에 하나의 이벤트 핸들러만 바인딩할 수 있다는 단점이 있다.
   - 동일한 이벤트 핸들러 프로퍼티에 이벤트 핸들러를 재정의할 경우 덮어씌워진다.

```html
<!DOCTYPE html>
<html>
<body>
   <button>Click me!</button>
   <script>
      const $button = document.querySelector('button');
      $button.onclick = function () {
         console.log('click!');
      };
      $button.onclick = function () {
         console.log('click2!');
      };
   </script>
</body>
</html>
```

#### 3. `addEventListener` 메서드 방식

추후에 등장한 방식
`EventTarget.addEventListener('eventType', functionName[, useCapture]);` 으로 사용

1. 첫 번째 매개변수에는 이벤트 타입을 전달한다.
   1. 이 때 `on` 접두사를 붙이지 않음에 유의
2. 두 번째 매개변수에는 이벤트 핸들러 함수를 전달한다.
3. 마지막 매개변수에는 이벤트를 캐치할 이벤트 전파 단계를 지정한다.
   1. 생략 또는 false인 경우 버블링 단게에서 이벤트를 캐치
   2. true인 경우 캡처링 단계에서 이밴트를 캐치

```html
<!DOCTYPE html>
<html>
<body>
   <button>Click me!</button>
   <script>
      const $button = document.querySelector('button');
      $button.addEventListener('click', function () {
         console.log('click!');
      });
   </script>
</body>
</html>
```

##### 두 개를 동시에 쓰면?

```html
<!DOCTYPE html>
<html>
<body>
   <button>Click me!</button>
   <script>
      const $button = document.querySelector('button');
      $button.addEventListener('click', function () {
         console.log('click!');
      });
      $button.onclick = function () {
         console.log('click 2!');
      };
   </script>
</body>
</html>
```

버튼에서 클릭 이벤트가 발생하면 2개의 이벤트 핸들러가 모두 호출된다.
> 즉 `addEventListener()` 메서드는 하나 이상의 이벤트 핸들러를 등록할 수 있다. (핸들러는 등록된 순서대로 호출된다)

##### 이벤트 핸들러 제거

`EventTarget.prototype.removeEventListener` 메서드 사용

전달할 인수는 `addEventListener` 메서드와 동일하지만, 메서드에 전달된 인수가 서로 일치하지 않는다면 이벤트 핸들러가 제거되지 않음에 유의

> 즉 이벤트 핸들러를 제거하기 위해서는 이벤트 핸들러의 참조를 별도로 저장하고 있어야 한다.

이벤트 핸들러 프로퍼티에 등록한 이벤트 핸들러는 프로퍼티에 다시 null을 할당해서만 제거가 가능함에 유의

### 이벤트 객체

이벤트 핸들러 어트리뷰트 방식의 경우 이벤트 객체를 인수로 전달받으려면 첫 매개변수 이름이 반드시 `event` 여야 함에 유의
- 암묵적으로 함수를 생성하는 과정에서 첫 매개변수 이름이 `event`가 아니면 바보가 된다

```html
<!DOCTYPE html>
<html>
<body>
   <p>클릭 한 곳 좌표 표시</p>
   <em class="message"></em>
   <script>
      const $msg = document.querySelector('.message');
      function showCoords(e) {
         $msg.textContent = `x: ${e.clientX}, y: ${e.clientY}`;
      }
      document.onclick = showCoords;
   </script>
</body>
</html>
```

이벤트가 발생 -> 암묵적으로 이벤트 객체 생성 -> 이 때 이벤트 객체는 생성자 함수에 의해 생성
- 때문에 이벤트 객체는 프로토타입으로 구성된 프로토타입 체인의 일원이 된다.
- `Event` 인터페이스는 DOM 내에서 발생한 이벤트에 의해 생성되는 이벤트 객체를 표현한다.
- `MouseEvent`와 같은 하위 인터페이스에는 이벤트 타입에 따라 고유한 프로퍼티가 정의되어 있다

> 즉 이벤트 객체의 프로퍼티는 발생한 이벤트의 타입에 따라 달라진다.

#### 이벤트 객체의 공통 프로퍼티

공통 프로퍼티 | 설명 | 타입
--- | --- | ---
type | 이벤트 타입 | string
target | 이벤트를 발생시킨 DOM 요소 | DOM 요소 노드
currentTarget | 이벤트 핸들러가 바인딩된 DOM 요소 | DOM 요소 노드
eventPhase | 이벤트 전파 단계 | number
bubbles | 이벤트를 버블링으로 전파하는지 여부 | boolean
cancelable | 이벤트의 기본 동작을 취소할 수 있는지 여부 | boolean
defaultPrevented | preventDefault 메서드를 호출해서 이벤트를 취소했는지 여부 | boolean
isTrusted | 사용자의 행위에 의해 발생한 이벤트인지 여부 | boolean
timeStamp | 이벤트가 발생한 시각 | number

target 프로퍼티와 currentTarget 프로퍼티는 동일한 DOM 요소를 가르키지만,
이벤트 위임을 사용한다면 서로 다른 DOM 요소를 가르킬 수 있음에 유의
