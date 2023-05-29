---
title: "Java Optional orElseGet"
date: 2023-05-29T20:06:15+09:00
draft: false
author: redjen
---

`Optional`을 사용할 때, `Optional` 내부의 값이 null이 될 때에 대한 처리를 하고 싶을 때가 빈번하게 일어나는 것 같다.

그럴 때마다 나는 `orElse`를 쓰면 되지~ 했었는데, 다음부터는 조심해서 사용해야 하겠다는 생각을 하게 되어 정리해봤다.

https://stackoverflow.com/questions/33170109/difference-between-optional-orelse-and-optional-orelseget

## orElseGet이란

`Optional.orElseGet()` 메서드는 인자로 전달된 함수가 `Optional.isPresent()`가 false일 때에만 eval된다.

반면 `Optional.orElse()` 메서드는 전달된 인자가 `Optional.isPresent()`의 값에 상관 없이 eval된다.

## 언제 무엇을 써야 할까

실제 코드를 작성할 때에는 필요한 리소스의 비용이 비쌀 때 `orElseGet()`을 사용해서 인스턴스화하는 비용을 정말 필요할 때에만 사용하도록 할 수 있다.

`orElse`의 인자는 옵셔널의 객체 값에 관계 없이 항상 실행되기 때문에 아래와 같은 경우에서는 특히 조심해야 한다.
- `orElse()`의 인자로 로깅문이 포함되는 경우
  - 옵셔널의 값에 관계 없이 로그가 찍히게 되어 혼란을 줄 수 있다.
- `orElse()`의 인자로 시간 민감한 내용이 들어가는 경우
  - DB, API 콜, 파일 입출력 등의 IO 연산을 수행하는 경우 쓸모 없는 코드를 수행하게 된다.
- `orElse()`의 인자로 객체의 상태를 변경하는 경우
  - 상태가 변경되는 객체를 다른 곳에서도 사용한다면 치명적인 버그를 낳을 수 있다.

그럼 `orElse`는 언제 사용해야 할까?
- 인자로 상수를 넣는 경우
- 인자로 enum을 넣는 경우

이외의 모든 경우에는 `orElseGet`을 사용하는 것이 좋아보인다. 