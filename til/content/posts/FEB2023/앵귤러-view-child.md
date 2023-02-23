---
title: "앵귤러 View Child"
date: 2023-02-23T17:41:49+09:00
draft: true
---

프론트엔드 작업을 하면서 가장 고민이 많이 되고 어렵지만 또 재미있는 개발은 컴포넌트 공통화인 것 같다.
- 동일한 코드를 재사용하지 않고 나이스한 방법으로
- 컴포넌트가 사용되는 여러 곳에 알맞게 딱딱 끼워넣고
- 해당 컴포넌트가 의도한 대로 동작하는 것을 보는 것은 즐겁다.

ViewChild라는 데코레이터는 이러한 종류의 작업에서 유용하게 사용할 수 있을 것 같다.

## 부모가 자식을 보고 싶을 때

부모 컴포넌트에서 자식 컴포넌트에의 접근이 필요한 경우 크게 두 가지 방법을 사용할 수 있다.
1. 템플릿에서 참조하는 방법 (템플릿 참조 변수)
2. 컴포넌트에서 참조하는 방법

### 템플릿 참조 변수

https://angular.kr/guide/template-reference-variables

> 템플릿 변수를 활용하면 템플릿 안에서 다른 영역에 있는 데이터를 참조할 수 있습니다.

템플릿 안에서 `#` 해시 기호를 사용하면 (`#phone`) `phone` 변수를 템플릿 변수로 선언할 수 있다.
그리고 이렇게 선언된 템플릿 변수를 템플릿 내부에서 자유롭게 참조할 수 있다.

### 템플릿 변수에 할당되는 값은 어떻게 관리하나?

- 컴포넌트에 템플릿 변수 선언 -> 해당 템플릿 변수는 **컴포넌트 인스턴스**를 가르킴
- 표준 HTML 태그에 템플릿 변수 선언 -> 해당 템플릿 변수는 **엘리먼트**를 가르킴
- `<ng-template>` 엘리먼트에 템플릿 변수 선언 -> 해당 템플릿 변수는 **`TemplateRef` 인스턴스**를 가르킴

나중에 템플릿 변수 참조에 대해 따로 정리해야겠다..

## ViewChild

컴포넌트 내부에서 부모 컴포넌트가 자식 컴포넌트를 참조해야 할 때 사용하는 데코레이터 중 하나이다.
https://angular.kr/api/core/ViewChild

- ViewChild는 View Query를 설정하는 데코레이터이다.
- 변경 추적기가 view DOM 내부에서 selector와 매칭하는 첫번째 엘리먼트나 디렉티브를 찾는다. (단일 요소에 대한 검색만 수행하는 것에 유의)
	- 여러 요소를 검색하려면 `@ViewChildren` 을 사용해야 한다.
- DOM이 변경된다면, 새로운 child가 selector와 매칭되고, 속성이 업데이트된다.

### View Query의 구성 요소들

View Query들은 `ngAfterViewInit` 콜백이 호출되기 전에 설정된다.
- selector: 쿼리 질의에 사용되는 디렉티브 타입이나 이름
- read: 쿼리된 엘리먼트들로부터 서로 다른 토큰을 읽기 위해 사용된다.
- static: 
	- true로 설정하면 변경 추적기가 실행되기 전에 쿼리 결과를 resolve한다.
	- false로 설정하면 (기본) 변경 추적 이후에 쿼리 결과를 resolve한다.

### 어떤 Selector와 함께?

아래의 selector들이 사용될 수 있다.
- `@Component` 나 `@Directive` 데코레이터가 붙은 모든 클래스
- 문자열로 전달되는 템플릿 참조 변수
	- `<my-component #cmp></my-component>` 와 `@ViewChild('cmp')`
- 현재 컴포넌트의 자식 컴포넌트 트리에 정의된 모든 프로바이더들 
	- `@ViewChild(SomeService)`와 `someService: SomeService`
- 문자열 토큰으로 전달되는 모든 프로바이더들
	- `@ViewChild('someToken')`와 `someTokenVal: any`
- `TemplateRef`과 `@ViewChild(TemplateRef): template;`
	- `<ng-template></ng-template>` 등

### 어떤 read와 함께?

아래의 값들은 `read`에 사용될 수 있다.
- `@Component` 나 `@Directive` 데코레이터가 붙은 모든 클래스
- 이 쿼리에 `selector`와 매칭되는 컴포넌트의 Injector에 정의된 프로바이더
- 문자열 토큰으로 전달되는 모든 프로바이더들
	- `{provide: 'token', useValue: 'val'}`
- `TemplateRef`, `ElementRef`, `ViewContainerRef`

### ngAfterViewInit

[앵귤러 라이프사이클 훅](https://redjen8.github.io/posts/feb2023/%EC%95%B5%EA%B7%A4%EB%9F%AC-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%ED%9B%85/)에서 정리했던 것처럼, `ngAfterViewInit`은 `ngOnInit` 다음에 실행된다는 점에 유의해야 한다.

### Content 탐색 기능 vs View 탐색 기능

차이점을 나중에 정리하려고 적어 놓는다.

https://angular.kr/api/core/ContentChildren

짧게만 정리하면..
> ViewChildren으로 시작하는 요소는 자신의 태그 내부 하위 엘리먼트에만 적용된다.
> ContentChildren으로 시작하는 요소는 자신의 태그에 들어온 새로운 객체에도 적용된다. 
