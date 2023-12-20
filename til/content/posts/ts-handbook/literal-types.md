---
title: "타입스크립트 핸드북 4장 - 리터럴 타입"
date: 2023-12-20T22:20:21+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/literal-types.html

> 리터럴 타입은 enum 타입의 하위 타입이다.

문자열과 숫자 두 가지 리터럴 타입이 존재하고, 이를 사용할 경우 문자열이나 숫자에 정확한 값을 지정할 수 있다.

### Literal Narrowing

- `var` 또는 `let`으로 변수를 선언하면 컴파일러는 이 친구들의 값이 변경될 수 있다는 것을 안다
- `const`로 변수를 선언하면 컴파일러는 이 친구는 값이 절대 안 바뀐다는 것을 안다

> 즉 무한한 수의 잠재 케이스를 -> 유한한 수의 잠재 케이스로 줄이는 것을 type narrowing이라 한다.

### String Literal Types

문자열 리터럴 타입을 사용하면 문자열로 enum과 비슷한 형태를 갖출 수 있다.

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";

class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {
      // ...
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // 하지만 누군가가 타입을 무시하게 된다면
      // 이곳에 도달하게 될 수 있습니다.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy");
```

문자열 리터럴 타입은 3장 오버로드 목록과 함께 사용할 수도 있다.

### Numeric Literal Types

숫자형 리터럴 타입은 보통 설정 값을 설명할 때 유용하게 사용될 수 있다.

```ts
declare function setupMap(config: MapConfig): void;

interface MapConfig {
  lng: number;
  lat: number;
  tileSize: 8 | 16 | 32;
}

setupMap({ lng: -73.935242, lat: 40.73061, tileSize: 16 });
```