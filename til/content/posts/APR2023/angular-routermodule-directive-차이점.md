---
title: "Angular Routermodule Directive 차이점"
date: 2023-04-28T18:57:19+09:00
draft: false
author: redjen
---

앵귤러 개발을 하던 중 앵귤러 라우터 관련하여 궁금한 점이 생겼다.

`component`, `loadChildren`, `redirectTo` 는 어떻게 다를까?

## component

`component` 속성은 컴포넌트로 직접 이동할 수 있는 링크 path를 제공한다.
- 단일 route에 대한 URL 처럼 기능하는 문자열이다.

## loadChildren

`loadChildren`은 `asychronous component`를 로딩하기 위해서 사용된다.
- 로딩할 route들의 집합에 대한 URL 처럼 기능하는 문자열이거나
- 로딩할 route들의 집합을 리턴하는 함수이다.

## redirectTo

`redirectTo`는 단순히 다른 route로의 리다이렉트만 제공한다.
- 단일 route에 대한 URL 처럼 기능하는 문자열이거나
- 가능하다면 URL로 리다이텍트를 시도하기 위해 사용되지만, 불가능한 경우 기본 URL로 기능한다.

https://angular.io/guide/router