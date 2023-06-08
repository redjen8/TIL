---
title: "React Empty Fragments"
date: 2023-06-07T23:44:02+09:00
draft: false
author: redjen
---

다른 분의 리액트를 코드를 보던 중 `<>`, `</>`와 같은 코드 조각이 있어 저건 뭐지? 하고 찾아보게 되었다.

https://medium.com/fasal-engineering/what-are-react-fragments-or-the-react-empty-tags-190253582905

## React Fragment

리액트 컴포넌트를 작성하는 동안에는 컴포넌트가 항상
- 렌더링할 jsx 표현식의 하나 이상의 엘리먼트이거나
- 부모 엘리먼트에 의해 래핑되어야 한다.

따라서 컴포넌트의 리턴문 (요즘에는 함수형 컴포넌트를 많이 사용하니까)은 항상 단일 엘리먼트를 가진다.
- 여러 자식 엘리먼트가 있는 래퍼 엘리먼트여도 되고,
- 일반적인 단일 HTML 엘리먼트여도 된다.

그렇기 때문에, 컴포넌트를 만들 때에는 항상 여러 개의 엘리먼트를 래핑해서 리턴해야 하는 상황에 봉착한다.

> 리액트 프래그먼트를 사용한다면 추가적인 래퍼 엘리먼트를 사용하지 않고도 단일 컴포넌트로 래핑할 수 있다.

```jsx
return (
    <React.Fragment>
        <h1>{props.title}</h1>
        <p>{props.paragraph}</p>
    </React.Fragment>
);
```

### 장점

리액트 프래그먼트가 도입되면서 아래의 두 가지 문제점들을 해결할 수 있었다.

1. 컴포넌트들을 컨테이너 엘리먼트를 사용하지 않고 리턴하거나 (함수형) 렌더할 수 (클래스) 있게 되었다.
2. jsx 문법을 사용해서 다수의 엘리먼트를 리턴할 수 있게 되었기 때문에, 단일 엘리먼트만 반환하기 위해서 가끔 잘못된 마크업을 작성하던 문제를 해결하였다.

## Empty Tag

자, 그럼 앞서 설명했던 빈 html 엘리먼트(`<>`, `</>`)는 무엇일까? 

이는 empty tag라고 부른다. 리액트 프래그먼트를 더 짧고 간결하게 사용할 수 있는 문법이다.

때문에 이전 예제는 empty tag를 사용해서 아래와 같이도 쓸 수 있다.
```jsx
return (
    <>    
        <h1>{props.title}</h1>
        <p>{props.paragraph}</p>
    </>
);
```

조금 더 보기에 깔끔해보인다. 

그럼 Empty tag와 리액트 프래그먼트는 완전히 기능적으로 동일할까?

### 리액트 프래그먼트와는 이런 점이 달라요

그렇지 않다. 

리액트 프래그먼트는 리액트 프래그먼트의 key 애트리뷰트를 사용할 수 있는 반면 empty tag는 프래그먼트의 애트리뷰트를 사용할 수 없다.

리액트 프래그먼트는 `key` 애트리뷰트를 제공하고, 이는 Keyed Fragment라고 부른다.

```jsx
return (
    <body>
      {props.list.map(({ title, paragraph }, index) => (
        <React.Fragment key={index}>
          <h1>{title} </h1>
          <p>{paragraph}</p>
        </React.Fragment>
      ))}
    </body>
  );
```

다음과 같이 key 애트리뷰트를 사용하는 예제에서는 empty tag를 사용할 수 없다.
