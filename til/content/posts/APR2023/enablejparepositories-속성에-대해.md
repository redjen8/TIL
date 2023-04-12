---
title: "@EnableJpaRepositories 속성에 대해"
date: 2023-04-12T21:00:34+09:00
draft: false
author: redjen
---

스프링 프로젝트 내부에서 다양한 레포지토리로 다양한 데이터 소스를 접근하기 위해서는 그에 알맞는 설정이 꼭 필요하다.

오늘은 `@EnableJpaRepository`의 속성인 `basePackages`와 `repositoryBaseClass`에 대해 알아봤다.

https://stackoverflow.com/questions/45663025/spring-data-jpa-multiple-enablejparepositories

## basePackage

`@EnableJpaRepositories` 어노테이션은 스프링에게 어떤 `DataSource`가 어떤 `Repository`를 통해 다뤄져야 하는지를 알게 한다.

이 때 `basePackage` 속성은 스프링에게 스캔할 패키지 범위를 지정하게 해준다.

멀티 레포지토리 모듈을 가지는 프로젝트의 경우, 서로 다른 패키지에 속해 있는 레포지토리를 의존해야 하는 상황이 생긴다.

이럴 때 서로 다른 레포지토리 간 설정에 있어 혼동을 막기 위해 `basePackage` 속성을 사용한다.

## repositoryBaseClass

그럼 `repositoryBaseClass` 속성은 왜 사용할까?

`repositoryBaseClass` 속성 설명을 읽어보면

> Configure the repository base class to be used to create repository proxies for this particular configuration.

라고 되어 있다.

즉 `repositoryBaseClass` 속성은 스프링에게 마찬가지로 레포지토리 구현체를 생성할 때 기본 구현체를 생성하지 말고 우리가 만든 이 기본 클래스를 사용해라~ 라고 알려주는 것이다.

## 정리

- `@EnableJpaRepositories` 어노테이션은 스프링에게 Datasource - Repository 매핑을 알려준다.
- `basePackage` 속성은 스프링이 스캔할 패키지 범위를 지정해준다.
- `repositoryBaseClass` 속성은 스프링이 기본 구현체 대신 사용할 레포지토리 구현체를 지정한다.

