---
title: "Js Window Screen Document의 차이점"
date: 2023-03-28T21:54:00+09:00
draft: false
author: redjen
---

FE 개발을 하다 보면 헷갈리는 개념들이 많다.

Javascript의 `window`, `screen`, `document`는 전부 비슷해보이는데, 어떻게 세부적으로 다를까?

https://stackoverflow.com/questions/9895202/what-is-the-difference-between-window-screen-and-document-in-javascript

## window

window는 자바스크립트의 루트 객체이다. 

브라우저 환경에서는 window가 곧 global object이다. (node.js 에서는 아니다)

### node.js 환경에서의 global object

https://stackoverflow.com/questions/43627622/what-is-the-global-object-in-nodejs

node.js에서의 global object는 현재 module.exports의 객체라고 한다.

### 브라우저 환경에서의 window 객체 

또한 브라우저 환경에서의 window 객체는 DOM의 루트 객체이기 때문에 window를 통해 브라우저 상에 존재하는 모든 DOM에 접근할 수 있다.

브라우저 환경에서의 global object는 항상 window이기 때문에, screen 이나 document는 `window.` prefix를 붙이지 않고도 접근 가능하다. (사실은 `window.screen`이랑 `window.document` 이다)

iframe을 사용할 경우에는 좀 독특한데, iframe 자체가 window가 된다. 
- 그렇기 때문에 iframe을 사용할 경우에는 frame 바깥의 원소를 접근하고 조작하기 까다로워지는 것이다.
- 이는 iframe이 별도의 루트 DOM 객체를 가지기 때문이다.

## screen

screen은 물리적으로 화면 dimension이 어떻게 구성되는지에 대한 정보를 담고 있는 작은 객체이다.

document 객체가 접근 가능하지만 아직 렌더링되지 않은 객체들에 대한 정보도 담고 있다면, screen 객체는 좀 더 나이브하게 실제로 눈에 보이는 요소들에 대한 정보만 가지고 있다.

- `width`와 `height` 속성을 지정하는 screen 객체는 full screen 이다.
- `availWidth`와 `availHeight` 속성을 지정하는 screen 객체는 toolbar를 그린다.

document를 렌더링하고 보여주는 screen의 일부분을 자바스크립트에서는 viewport라고 부른다.
- 이 개념이 헷갈릴 수 있는데, 우리는 운영체제와 상호작용을 이야기 할 떄 어플리케이션에서 화면을 보여주는 부분을 마찬가지로 viewport라고 부르기 때문이다.
- 모든 `document` 객체의 `getBoundingClientRect()` 메서드는 viewport에서의 객체의 위치를 기술하는 `top`, `left`, `buttom`, `right` 속성을 리턴한다.

## document

document는 현재 DOM으로부터 접근 가능한 (potentially visible) 메인 객체이다.

모든 window 객체는 렌더링하기 위해서 document 객체를 가진다. 이 객체들은 HTML 요소들이 unique한 id를 부여 받았을 때 global object로 더해지기 때문에 헷갈릴 수 있다.

예시로 아래의 HTML 스니펫의 `p` 태그 엘리먼트에 접근하기 위해서는 다양한 방법이 존재한다.
```javascript
<body>
  <p id="holyCow"> This is the first paragraph.</p>
</body>
```

-  `window.holyCow` 
	- 또는 `window["holyCow"]`
-   `document.getElementById("holyCow")`
-   `document.querySelector("#holyCow")`
-   `document.body.firstChild`
-   `document.body.children[0]`

DOM을 조작할 때에는 document 객체를 통해 조작된다는 것을 기억하면 좋을 듯하다.

