---
title: "Javascript의 객체 복사"
date: 2023-02-24T17:58:09+09:00
draft: false
author: redjen
---

얕은 복사 / 깊은 복사 개념은 아니까 패스~

어떻게 복사하는 게 좋을까? 하던 중 잘 정리된 글이 있어서..
https://www.daleseo.com/js-objects-clone/

## 얕은 복사

### Object.assign()

`Object.assign()`을 사용하면 한 객체의 속성을 다른 객체의 속성으로 복사할 수 있다.

언뜻 봤을 때에는 메서드 명이 깊은 복사를 할 수 있다고 착각하기 쉽지만, 객체나 배열의 속성을 변경해보면 그렇지 않다는 것을 알 수 있다.

### spread operator`(...)`

https://redjen8.github.io/posts/jan2023/javascript-spread-operator%EC%97%90-%EB%8C%80%ED%95%B4/
이전에 알아봤던 것처럼 spread operator는 얕은 복사를 하는 대표적인 예시이다.
의도적으로 얕은 복사를 사용할 일이 있다면, spread operator를 사용하는 것이 좀 더 깔끔한 코드를 작성하는데 도움이 될 수 있을 것 같다.

## 깊은 복사

### JSON.parse(JSON.stringify(obj))

- 객체를 Json 형태 문자열로 바꾼다음
- Json 문자열을 다시 객체로 변환하는 무지막지한 방법이다.

직렬화 / 역직렬화에 비용이 많이 들기 때문에, 본능적으로 이건 별로 쓰고 싶지 않다는 생각이 든다.

이 방법은 깊은 복사를 별도의 라이브러리 의존 없이 할 수는 있지만, 알려진 문제점들로는
1. 객체 안에 들어 있는 함수 데이터들이 누락된다. (Json 직렬화 과정에서 누락 - Json은 함수 데이터를 포함하지 않기 때문)
2. 객체 트리 내에 순환 참조가 있는 경우 오류 발생
	1. `JSON.stringify()` 메서드에서 순환 참조를 감지하고 오류를 발생

## lodash 라이브러리의 cloneDeep()

널리 알려져 있는 라이브러리인 lodash의 `cloneDeep()`을 사용하면 깊은 복사를 쉽게 할 수 있다.
하지만 라이브러리 의존은,, 의존성 관리 때문에 고민해봐야 하는 점이 좀 있다. 편하지만 손이 가지는 않는다.

### 직접 구현

상단에 링크를 단 DaleSeo 님의 블로그에서 코드를 퍼오자면, 
```javascript
function clone(source) {
  var target = {};
  for (let i in source) {
    if (source[i] != null && typeof source[i] === "object") {
      target[i] = clone(source[i]); // resursion
    } else {
      target[i] = source[i];
    }
  }
  return target;
}

const clone5 = clone(original);

console.log(original.func); // function func()
console.log(clone5.func); // function func()
```

재귀를 통해 깊은 복사를 위와 같이 구현할 수 있을 것이다.
만약 범용적으로 깊은 복사를 사용하지 않고, 특정 객체에만 사용할 것이라면 굳이 별도의 메서드를 사용하지 않고 속성들을 손수 한땀한땀 옮겨주는 것이 오히려 더 좋을 수도 있을 것 같다.

다만 깊은 복사 도중에 비즈니스 로직을 좀 끼워넣는 경우가 아니라면,, 그냥 맘 편하게 lodash 쓰는게 나을 것 같다!