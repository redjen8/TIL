---
title: "Ng Container에 대해"
date: 2023-05-03T21:14:12+09:00
draft: false
author: redjen
---

FE 개발을 하다 보니 `<ng-container>` 요 녀석을 만나게 되었다.

`ng-container`는 어떤 엘리먼트이고, 언제, 어떻게 사용해야 할까?

https://angular.io/api/core/ng-container#description

## ng-container

`<ng-container>` 는 별도의 element 추가 없이 structural directive들을 사용할 수 있게 해준다.
- 때문에 적용되는 DOM 변경 사항들은 디렉티브 자체에 의해서 생긴 변경 사항만 있음을 확인할 수 있다.
- 이렇게 하면 브라우저의 입장에서 렌더링 요소가 줄어들기 때문에
	- 성능 향상
	- DOM과 코드 스타일을 깔끔하게 유지할 수 있다.
- 예를 들어 스타일을 변경하지 않고 structural 디렉티브를 사용할 수 있게 한다. (flex container, margin, child combinator selector 등)

## 사용 방법

### `*NgIf`와 사용

`<ng-container>`는 `*ngIf` structural directive와 종종 같이 사요되곤 한다.

이 엘리먼트를 사용한다면 이해하기 굉장히 쉽고 다루기도 용이한 깔끔한 템플릿을 만들 수 있다.

예를 들어 동일한 루트 엘리먼트 아래에 전체 엘리먼트들이 아닌 그 중 조건적으로 일부 엘리먼트들만 보여주길 원한다면 어떨까?

```html
<ng-container *ngIf="condition">
</ng-container>
```

`<ng-template>`와 함께 사용한다면 `else` statement를 더 강력하게 활용할 수 있다.

```html
<ng-container *ngIf="condition; else templateA">
… 
</ng-container> 
<ng-template #templateA> … </ng-template>
```

### 여러 structural directive와 함께 사용

다수의 structural directive들은 동일한 엘리먼트에 사용될 수 없다.
때문에 하나 이상의 structural directive를 사용하기 위해서는 단일 structural directive 당 `<ng-container>`를 사용하는 것이 권고된다.

가장 흔한 시나리오는 `*ngIf`와 `*ngFor`를 사용하는 것이다.

예를 들어 아이템들의 목록을 가지고 있고 각각의 아이템들이 특정 조건이 true일 때에만 표시되야 한다면 어떨까?
```html
<ul>
	<li ngFor="let item of items" *ngIf="item.isValid"> {{ item.name }} </li> </ul>
```

상기 예시는 한 엘리먼트에 다수의 structural directive를 사용했기 때문에 동작하지 않는다. 때문에 아래와 같이 변경되어야 한다.

```html
<ul>
  <ng-container *ngFor="let item of items">
    <li *ngIf="item.isValid">
      {{ item.name }}
    </li>
  </ng-container>
</ul>
```

이 예시는 DOM에 어떠한 불필요한 엘리먼트를 추가하지 않고도 의도한 대로 동작한다.

### ngTemplateOutlet와 함께 사용

`NgTemplateOutlet` 디렉티브는 어느 엘리먼트에나 적용될 수 있지만, 대부분의 경우 `<ng-container>`에 적용된다. 두 디렉티브를 결합함으로써 불필요한 요소를 추가하지 않고 요청과 동시에 템플릿 뷰가 인스턴스화 되기 때문에 HTML과 DOM 구조를 이해하기 매우 쉬워진다.

예를 들어 큰 HTML이 있고 서로 다른 부분이 서로 다른 곳에서 필요하다고 가정해보자.
간단한 솔루션은 우리가 가진 HTML을 단순히 반복하는 `<ng-template>`를 선언하고 `NgTemplateOutlet`와 함께 `<ng-container>` 을 사용해서 렌더링하는 것이다. 

```html
<!— … —>

<ng-container *ngTemplateOutlet="tmpl; context: {$implicit: 'Hello'}">
</ng-container>

<!— … —>

<ng-container *ngTemplateOutlet="tmpl; context: {$implicit: 'World'}">
</ng-container>

<!— … —>

<ng-template #tmpl let-text>
  <h1>{{ text }}</h1>
</ng-template>
```