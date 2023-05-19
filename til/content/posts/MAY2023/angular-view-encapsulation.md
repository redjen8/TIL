---
title: "Angular View Encapsulation"
date: 2023-05-19T16:05:02+09:00
draft: false
author: redjen
---

앵귤러를 사용한 FE 개발을 하다가 도저히 컴포넌트에 먹인 scss 적용이 잘 되지 않아서 구글링을 열심히 했다.

그러다 나온 것이 바로 뷰 캡슐화 개념이다..! 이에 대해 알아봤다.

https://angular.kr/guide/view-encapsulation

## 개념

컴포넌트에 적용된 CSS 스타일이 컴포넌트 뷰를 대상으로 캡슐화시켜서, 다른 애플리케이션의 컴포넌트에 영향을 주지 않도록 하는 개념이다.

때문에 캡슐화 정책은 컴포넌트마다 다르게 지정할 수 있고, 정책은 `Component` 데코레이터의 `encapsulation` 옵션 지정을 통해 바꿀 수 있다.

## 종류

- `ShadowDom` : 브라우저의 기본 [shadow DOM](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_shadow_DOM)을 활용해 뷰를 캡슐화
- `Emulated` : 컴포넌트 CSS 셀렉터를 조정해 'shadow DOM처럼' 캡슐화한다.
- `None` : 뷰 캡슐화를 하지 않는다.
  - 컴포넌트에 지정한 스타일은 해당 컴포넌트 뿐 아니라 전역 범위에 적용된다.
  - 컴포넌트는 캡슐화되지 않고, HTML 문서에 직접 컴포넌트 스타일을 지정한 것과 같은 효과를 낸다.

## 자동 생성된 CSS

뷰 캡슐화 정책을 `Emulated`로 설정하면 모든 컴포넌트에 대한 스타일을 전처리해서 표준 shadow CSS 규칙처럼 변환한다.

```html
<hero-details _nghost-pmm-5>
  <h2 _ngcontent-pmm-5>Mister Fantastic</h2>
  <hero-team _ngcontent-pmm-5 _nghost-pmm-6>
    <h3 _ngcontent-pmm-6>Team</h3>
  </hero-team>
</hero-details>
```

붙여지게 되는 어트리뷰트는 두 가지 종류가 있다.
1. `_nghost` : 섀도우 DOM의 호스트 엘리먼트에 붙는다. (일반적으로 앵귤러 컴포넌트의 호스트 엘리먼트에 해당)
2. `_ngcontent` : 컴포넌트 뷰 안에 있는 자식 엘리먼트에 추가된다. 섀도우 DOM의 동작을 흉내내기 위해 붙는다.


