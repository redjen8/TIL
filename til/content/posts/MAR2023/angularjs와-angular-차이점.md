---
title: "AngularJS와 Angular 차이점"
date: 2023-03-24T17:01:12+09:00
draft: false
author: redjen
---

angular.js 와 angular는 어떻게 다를까?

https://www.interviewbit.com/blog/difference-between-angular-and-angularjs/

구분 | angular | angular.js
--- | --- | ---
Maintainer | Google의 Angular 팀 + 기관과 개인이 포함된 커뮤니티 | 구글 + 기관과 개인이 포함된 커뮤니티
지원 언어 | Typescript를 JS보다 잘 지원 | JS Only
모바일 지원 | 모바일 친화적 | 모바일 친화적이지 않음
아키텍쳐 지원 | 컴포넌트 기반 아키텍쳐 (디렉티브 + 템플릿) | MVC, MVVM 아키텍쳐 지원
성능 또는 속도 | 데이터 바인딩 기능을 통한 고성능 어플리케이션, 추가적으로 SSR 지원도 가능 | 동적 웹 페이지를 위한 양방향 바인딩 지원 - 꽤 괜찮은 성능이지만 모던 어플리케이션에 어울리는 성능은 아님
난이도 | 지켜야 할 규칙들이 있기 때문에 난이도가 있는 편 | Angular에 비해서는 배우고 사용하기 쉽다
프로젝트 관리 | 쉽다 | 특정 구조를 따르지 않기 때문에 관리하기 어려움
의존성 주입 | 계층적인 의존성 주입을 통해 어플리케이션 성능 향상 | 의존성 주입 대신 디렉티브를 사용
라우팅 | url을 통한 다양한 view 간 라우팅을 지원 | 마찬가지로 라우팅 정보를 정의할 수 있음
데이터 바인딩 | view - model 간 양방향 바인딩 지원 | 마찬가지로 양방향 바인딩 지원하지만 `ng-bind`은 단방향, `ng-model` 은 양방향임
CLI | AngularCLI 툴을 통해 컴포넌트와 프로젝트 파일 생성 가능 | CLI 툴 미지원
테스팅 | Karma를 통한 유닛 테스트 지원, CLI를 통해 앱 빌딩하기 때문에 쉬운 테스팅 보장 | 써드 파티 툴을 통해서 테스트 가능
SEO 지원 | 검색 엔진의 크롤러를 위한 요소를 지원 | SEO 친화적인 방법을 지원하지 않음
어플리케이션 예시 | Gmail, Upwork, JetBlue, Wikiwand | Netflix, Lego, IStock, AngularJS 공식 웹 사이트
아키텍쳐적인 특징 | 컴포넌트 + 디렉티브를 사용 | 보통 MVC 디자인 사용
Angular Universal 지원 | 지원 | 미지원
