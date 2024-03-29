---
title: "React Dom Mount"
date: 2023-02-26T19:18:46+09:00
draft: false
author: redjen
---

저번에는 리액트의 `useState`와 `useEffect` 훅에 대해 알아봤었다.

그런데, 리액트에서 '마운트'한다는 것은 어떤 의미인지 정확히 모르는 것 같아 좀 더 찾아보았다.

https://stackoverflow.com/questions/31556450/what-is-mounting-in-react-js

### 리액트가 하는 일

리액트의 주된 일은 컴포넌트가 화면에 나타나길 바라는 변경 사항을 바탕으로 DOM을 어떻게 수정해야 할 지 알아내는 것이다.

리액트는 이 목표를 '마운트'와 '언마운트', '업데이트'를 통헤 달성한다. 각각의 행위를 간단히 정리하면 아래와 같다.
- 마운트: DOM에 노드를 더하는 행위이다.
- 언마운트: DOM에서 노드를 제거하는 행위이다.
- 업데이트: 이미 DOM에 있는 노드에 변화를 주는 행위이다.

### 마운트는 실제로 어떻게 이뤄질까

마운트는 DOM에 노드를 더하는 행위이다.

- 리액트의 노드가 어떻게 DOM 노드로 표현될 것인지
- DOM 노드의 어디에 들어갈 것인지
- DOM 트리에 언제 나타날 것인지

에 대한 정보는 React의 상위 레벨 API를 통해 이뤄진다.

이를테면 `let foo = React.createElement(FooComponent);` 와 같은 코드를 통해 리액트의 컴포넌트를 만들 수 있다. (이 떄 실제로 인스턴스가 생기는 것이 아님에 유의해야 한다. `foo` 객체는 가상의 DOM 표현체라고 본다)

하지만 이 코드가 선언된 시점에서는 아무 일도 일어나지 않는다. `foo` 객체는 단지 단순한 자바스크립트 객체이다.

`foo` 객체는 현재 어느 페이지에도 존재하지 않고. DOM 엘리먼트도 아니고, DOM 트리 어디에도 존재하지 않고, 리액트 엘리먼트 노드에서 빠져 있으며, 도큐먼트 내부에서 어떠한 의미 있는 표현을 가지지 않는다.

`foo` 객체는 단순히 리액트에게 '만약 이 엘리먼트가 렌더링 된다면' 화면에 뭐가 나와야 하는지 알려주는 그 역할 이상도, 이하도 수행하지 않는다. 

`ReactDom.render(foo, domContainer);` 코드를 실행하면~

- 리액트가 `foo` 객체를 페이지에 보여준다.
- 리액트는 `FooComponent` 클래스의 인스턴스를 생성하고, `render` 메서드를 호출한다.
- 리액트는 생성한 인스턴스의 DOM 노드를 생성하고, DOM 컨테이너에 삽입한다.

> 인스턴스와 리액트 컴포넌트에 해당하는 DOM 노드를 셍상하고, DOM 내부에 삽입하는 것이 리액트에서의 마운트이다.

### JSX가 하는 일은 뭘까

jsx는 자바스크립트에도 존재하지 않고, 브라우저에도 존재하지 않는다.

> JSX는 매우 구체적인 자바스크립트 객체를 만들기 위한 syntactic sugar이다.

https://ko.reactjs.org/docs/introducing-jsx.html

```jsx
const element = <h1>Hello, world!</h1>;
```
위와 같은 자바스크립트를 확장한 문법을 사용함으로써, UI가 어떻게 생겨야 하는지 설명하기 위해 리액트와 함께 사용된다.
- JSX는 리액트 엘리먼트를 생성한다.

`React.createElement('div', { }, 'Hello')`와 같은 코드는 앞서 말했듯이 순수 자바스크립트 객체를 생성한다.
그리고 이 엘리먼트 아래의 자식 엘리먼트를 삽입하게 되면, `props.children`에 `ref` 속성으로 해당 객체의 심볼을 가지게 된다.

JSX가 파싱되고 `React.createElement` 콜이 해석되면 하나의 큰 nested 객체를 가지게 된다.