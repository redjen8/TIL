---
title: "Scss란"
date: 2023-03-27T19:30:37+09:00
draft: false
author: redjen
---

순수 CSS는 어플리케이션이 커지면서 한계를 많이 맞이하게 되었고, 어떻게 하면 더 효율적으로 css 파일을 작성할 수 있을지 고민한 결과로 SCSS가 탄생했다.

SCSS란 무엇이고, 어떻게 사용되고 있는지 간단히 알아봤다.

https://medium.com/swlh/learn-the-scss-sass-basics-in-5-minutes-73002653b443

## CSS의 불편함과 해결사 SCSS

CSS는 HTML 코드를 보다 예쁘게 만들어주는 스타일 시트이다.

하지만 CSS는 쉽게 읽고 쓸 수 있도록 하는 기능을 많이 지원하지 않았다.

SASS는 변수, Nesting, Partial, Mixin, 확장 / 상속, 연산자를 지원하는 CSS이다.

SASS는 결국 CSS로 컴파일되어서 사용자에게 보여지는 최종 파일이 된다.
그 사이에 개발자는 편하게 개발할 수 있으므로, SASS를 'CSS를 개발 편의성을 위해 한 단계 추상화 한 파일'이라고 봐도 무방할 듯 싶다.

## 무엇이 다를까?

### 1. 변수

모든 프로그래밍 언어에는 변수 개념이 존재한다. SASS는 변수를 선언하도록 허용해서 색상을 변경해야 할 때 동일한 색상에 대해 노가다 작업을 하지 않도록 도와준다. 변수를 각 속성에 적용시키고, 적용된 변수의 값을 바꾸게 되는 것으로 족하다.

```scss
$primary-color: #333; 
body {
  background-color: $primary-color;
}
.text {
  color: $primary-color;
}
```

### 2. Nesting

CSS에서는 Nested한 구조를 허용하지 않는다. properties와 yml 파일의 차이처럼, nested한 구조를 도입하는 것은 중복되는 코드를 상당히 줄이는 데에 도움이 된다.

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }
}
```

### 3. Partial

Partial들은 파일 이름에 `_` (언더스코어)가 붙은 SASS 또는 SCSS 파일이다. (`_test.scss`처럼)

이 partial들은 css로 변환되지 않는 특징을 가진다. 이 파일들은 다른 SCSS 파일들에 의해 import 될 수 있는 CSS 코드 스니펫을 가진다.

이런 방식으로 개발한다면, CSS 자체를 모듈화하여 유지 보수하는데 큰 도움이 된다. 다양한 파일들에서 사용될 변수들을 저장하고 싶을 때에는 partial를 사용하는 것처럼..

### 4. Mixin

Mixin은 프로그래밍 언어와 비슷한 기능이다. 

```scss
@mixin transform($property) {
  -webkit-transform: $property;
  -ms-transform: $property;
  transform: $property;
}
.box { @include transform(rotate(30deg)); }
```

`rotate(30deg)`를 3번 입력하는 것 대신에, 이러한 Mixin들을 사전에 제작해서 함수처럼 활용할 수 있다.

이 Mixin 코드를 사용하려면, 반드시 `@include` 키워드를 사용해야 한다.

