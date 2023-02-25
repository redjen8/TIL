---
title: "React UseState와 UseEffect"
date: 2023-02-25T16:55:12+09:00
draft: false
author: redjen
---

난 리액트를 모른다. 리액트를 사용해서 뭔가를 만들어 본 적도 없지만 리액트로 짜여진 코드는 커뮤니티 등에서 아주 쉽게 찾아볼 수 있었기 때문에 - 관심은 가지고 있는 상태이다.

기회가 된다면 리액트를 사용해 토이 프로젝트 페이지를 만들어보고 싶다. 매력적인 라이브러리이고, 수많은 프론트엔드 개발자들에게 각광을 받는 이유가 반드시 있을 것이라는 생각이 든다.

오늘은 그 중 가장 많이 언급되고 또 기본이라고 생각되는 리액트의 `useState`와 `useEffect`가 무엇인지, 왜 써는지를 알아봤다.

https://initialcommit.com/blog/usestate-useeffect-hooks-react

## 리액트 훅

앵귤러에서의 라이프사이클 훅처럼, 리액트도 리액트 훅이 존재한다. 

> 리액트 훅은 어플리케이션의 상태와 컴포넌트 안에서 라이프사이클 이벤트를 다루기 위한 방법이다.

앞서 알아보고자 했던 `useSstate`와 `useEffect`는 이 리액트 훅의 한 종류이다.

## 리액트의 컴포넌트들은 어떻게 동작하는가?

리액트의 컴포넌트들은 자바스크립트 클래스로 작성된다. 각각의 클래스 컴포넌트들은 `state` 속성에 상태 변수들을 저장하고, `setState` 함수를 통해 상태가 업데이트 된다.

컴포넌트 라이프사이클 이벤트는
- `componentDidMount()`
- `shouldComponentUpdate()`
- `componentDidUpdate()`
- `componentWillUnmount()`
와 같은 메서드들을 사용해서 다루어질 수 있다.

프롭들은 생성자 함수를 통해 컴포넌트들로 전달되고, 컴포넌트들은 `render()` 함수를 톨해 렌더링된다.

## 발생하는 문제점들

이 방식으로 간단한 리액트 컴포넌트들을 만들기 위해서는 너무 많은 메서드들을 사용해야만 했다.

클래스 컴포넌트들은 아직도 리액트에서 지원되는 기능이지만, 대부분의 리액트 개발자들은 함수형 컴포넌트를 사용한다.

### 왜 함수형 컴포넌트인가?

함수형 컴포넌트들은 각각의 컴포넌트를 단일 함수로 생성함으로써 개발 프로세스를 간단히 할 수 있다는 장점을 가진다.

이 함수는 또한 프롭들을 인자로 받아서 분리된 `render()` 함수를 리턴하는 것이 아니라 jsx를 리턴할 수도 있다.

리액트 훅들은 이 함수형 컴포넌트들이 상태를 관리하고 컴포넌트 라이프사이클을 정돈된 채로 유지할 수 있게 도와준다.

## 리액트의 useState 훅

`useState` 훅은 우리가 만든 컴포넌트를 위한 상태 변수를 생성할 수 있도록 해준다.

상태 변수들은 우리가 만든 컴포넌트 안에 사용자가 계속 상호작용해서 변동될 수 있는 동적인 데이터를 저장하는데 사용된다.

상태의 예시로는 - 사용자가 채우는 양식이 있을 수 있다.
사용자가 양식을 채우면, 컴포넌트는 지속적으로 상태를 업데이트하고, 양식의 데이터를 최신으로 유지하기 위해 재렌더링하게 된다.

```javascript
import { useState } from 'react';

function App() {
    const [input, setInput] = useState("");

    return (
        <div className="App">
        <h1>Input value: {input}</h1>
            <input value={input} onChange={(e) => setInput(e.target.value)}/>
        </div>
    );
}
```

`useState`는 초기 값으로 인자를 받아서 상태 변수와 변환 함수를 포함하는 배열을 리턴한다. 일반적으로는 이 배열을 해체하고 내용물들을 const로 받는다. 이는 몇가지 이유 때문인데,
1. 상태 변수를 직접 재할당해서는 안된다.
2. setter를 통해서만 수정해야 한다.
3. setter 함수는 새 값 또는 현재 값을 인수로 사용하고 새 값을 반환하는 함수가 들어갈 수 있다.

## 리액트의 useEffect 훅

`useEffect` 훅은 컴포넌트 라이프사이클 안에서 변화에 응답할 수 있도록 해준다.

컴포넌트 라이프사이클은 컴포넌트가 DOM에 마운트되어 언마운트되기 전까지 발생하는 모든 이벤트의 집합을 말한다.

`useEffect`는 컴포넌트가 처음에 렌더링될 때, 업데이트될 때, 언마운트되는 시점에 코드를 실행하기 위해 흔하게 사용된다.

```javascript
function App() {
  const [input, setInput] = useState("");
  const [words, setWords] = useState([]);

  useEffect(() => {
    document.title = `${words.length} words`;
  }, [words]);

  return (
	// ...
```

`useEffect`는 함수와 의존성 배열을 인자로 허용한다. `useEffect` 함수는 이 의존성 배열이 변화할 때마다 실행된다.

만약 의존성 배열 없이 사용한다면, `useEffect` 함수는 컴포넌트가 재렌더링되는 모든 시점에 실행된다. 

만약 의존성 배열이 비어있게 된다면, 함수는 컴포넌트가 DOM에 처음 마운트될 때에만 실행된다.
    비어 있는 의존성 배열에 `useEffect`를 사용하는 대표적인 예시로는 API로부터 데이터를 가져올 때가 있다.


```javascript
function App() {
  const [data, setData] = useState(null);

  useEffect(async () => {
    const res = await fetch("https://api.com/api/v1");
    const json = await res.json();
    setData(json);
  }, []);
  return <p>data: {JSON.stringify(data)}</p>;
}
```
상기 예시에서는 컴포넌트가 처음 렌더링될 때, API를 fetch해서 그 데이터를 표시한다.

또한, useEffect는 컴포넌트가 언마운트될 때에도 함수를 실행할 수 있게 한다.
- `setInterval`이나 이벤트 기반 라이브러리를 사용할 때 인터벌의 정보나 컴포넌트 라이프사이클 내에서 설정된 이벤트 리스너를 초기화하는데 사용된다.
- 이를 cleanup 함수라고 부르고, `useEffect` 함수에 전달되어 다음과 같이 사용될 수 있다.

```javascript
function KeyLogger() {
  function handleKeyDown(e) {
    document.title = `${e.code} pressed`;
  }
  useEffect(() => {
    document.addEventListener("keydown", handleKeyDown);

    return () => {
      document.removeEventListener("keydown", handleKeyDown);
    };
  });
  return <h2>Press a key</h2>;
}
```

상기 예제에서, 컴포넌트가 마운트 될 때 키 다운 이벤트에 대해 리스너를 추가하고 눌려진 키를 document의 타이틀에 표시한다.

cleanup 함수는 이 등록된 이벤트 리스너를 컴포넌트가 언마운트될 때 제거한다. cleanup 함수가 없다면, 컴포넌트가 언마운트된 이후에도 document의 타이틀이 계속해서 업데이트 될 것이다.

## 정리

> `useState` 훅은 어플리케이션의 상태를 저장하는데 사용되며, 사용자 상호작용이 발생할 때 어플리케이션의 상태를 변경한다.

> `useEffect` 훅은 컴포넌트가 라이프사이클 이벤트에 따라 특정 동작을 수행하기를 원할 때 사용된다. (대표적인 예시로는 DOM에 마운트 되는 시점, 재렌더링되는 시점, 언마운트되는 시점이 있다.)