---
title: "Javascript Optional Argument의 위험성"
date: 2023-03-23T18:48:41+09:00
draft: false
author: redjen
---

MDN에서 소개했던 `Array.map()`의 optional argument로 인한 부작용 사례를 소개한 블로그 글을 보면서 왜 이런 일이 발생했는지 알아보는 시간을 가졌다.

http://wirfs-brock.com/allen/posts/166

## 개요

`["1", "2", "3"].map(parseInt)`의 결과는 무엇일까?

`[1, 2, 3]`일까?

그렇지 않다. 답은 `[1, NaN, NaN]`이다.

## 왜 이런 참사가 발생하는가

### parseInt에 대해

`parseInt()`는 string을 numeric literal로 변환해주는 빌트인 함수이다.

`let n = parseInt("123")` 을 실행한다면, `n`에는 123이 들어간다.

`let x = parseInt("xyz")`를 실행한다면, x에는 NaN이 들어간다.

### map에 대해

`map`은 ECMAScript5 부터 Array에 적용될 수 있는 빌트인 함수이다.

`map`은 배열 안의 각 원소들에 대해 연산을 수행한 결과를 새로운 배열로써 반환한다.

### 범인은 optional argument

`parseInt`에는, 인자를 하나 이상 넣을 수 있다.

`parseInt("ffff", 16)`의 결과는 65536이다.

즉, `parseInt`의 두번째 인자는 파싱되길 원하는 숫자의 진법을 지정한다.

그런데 여기서 두 번째 인자가 비어있거나 (== 인자를 하나만 입력한 경우) 0을 입력했다면 자동으로 `parseInt`는 `parseInt("someNumber", 10)`으로 변환하여 실행한다. 즉 기본 10진법을 사용하도록 설정되어 있기 떄문에 이를 눈치채지 못한 것이다.

그런데, `map`에도 헷갈리게 할 만한 optional argument가 있다.

`map`의 인자로 들어가는 `callbackfn`에는 세 개의 인자가 들어갈 수 있다.
1. 원소의 값
2. 원소의 인덱스
3. 순회 대상이 되는 객체

때문에, 우리가 하려던 `.map(parseInt)`는 사실 아래와 같았지만..
```js
parseInt("1")
parseInt("2")
parseInt("3")
```

실제로 행한 동작은 아래와 같다는 것이다.
```js
parseInt("1", 0, arr)
parseInt("2", 1, arr)
parseInt("3", 2, arr)
```

- 첫 번째 `parseInt`에서는 0이 두 번째 인자로 왔기 때문에 10진법으로 1을 파싱하려 시도하지만,
- 두 번째 `parseInt` 부터는 1 진법으로 "2"를 파싱할 수 없다. 따라서 NaN을 리턴한다.

`parseInt`가 두 번째 인자까지만 받기 때문에 추가 인자인 세 번째 인자는 무시된다. 

## 이런..

우리는 위 코드에서 두 가지를 잊어버렸기 때문에 코드가 의도대로 동작하지 않았다.
1. `parseInt`가 optional argument로 두 번째 인자를 받을 수 있다는 점
2. `map`의 `callbackfn`이 3개의 인자를 사용한다는 점

그럼 어떻게 바꿔야 우리의 의도대로 코드가 동작할 수 있을까?

`["1", "2", "3"].map(parseInt)` 대신

`["1", "2", "3"].map(function(value) { return parseInt(value)}` 를 사용하면 된다.

또는 `only` function을 아래와 같이 정의해서 사용할 수도 있다.
```js
Function.prototype.only = function(numberOfArgs) {
	var self = this;
	return function() {
		return self.apply(this, [].slice.call(arguments, 0, numberOfArgs));
	}
}

["1", "2", "3"].map(parseInt.only(1));
```

위와 같이 `only`를 사용한다면 함수에서 사용할 인자 개수를 제한시킬 수 있다. 

하지만 그보다 더 중요한 것은 `map`이나 `parseInt`의 optional argument에 대해 잘 모르는 채로 함수를 사용했다는 것이고, API를 호출할 때 위와 같은 일이 발생하지 않도록 주의를 기울여야겠다.


