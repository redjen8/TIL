---
title: "Gradle Dependency 관리"
date: 2023-02-14T19:37:20+09:00
draft: false
author: redjen
---

## gradle은 무엇인가?

https://docs.gradle.org/current/userguide/what_is_gradle.html

gradle은 거의 모든 소프트웨어에 적용할 수 있을 정도로 유연한 오픈소스 빌드 자동화 툴이다.

maven과 비교하여 조금 더 간단한 문법이 취향이라 개인 프로젝트에서 많이 사용하게 되는 것 같다.

현재 LTS 버전은 8버전이다. 

## gradle의 의존성 관리

### 모듈 의존성

가장 흔한 종류의 의존성들이다.
레포지토리 안에 있는 모듈을 참조한다.

`DependencyHandler`에 의해 설정이 된다.

https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html

기본적인 문법 구조는 아래와 같다.
```groovy
dependencies {
	configurationName dependencyNotation 
}
```

### 파일 의존성

프로젝트들은 가끔 이진 레포지토리에 의존하지 않고 파일로부터 의존을 할 때가 있다. 대외로 공개되지 않은 사내 전용 메신저 플러그인 같은 것들이 그 예시이다. 

- 공유 드라이브에 이 의존성들을 호스팅해서 프로젝트 소스코드와 함께 버전 관리의 대상이 되도록 관리하는 방식을 많이 사용하게 된다.

모듈 의존성과는 달리 부가적인 메타 데이터를 포함하고 있지 않고 바로 다이렉트로 파일을 참조한다는 차이점이 있다.

파일 의존성 관리의 치명적인 약점이 있다면, gradle의 버전 컨플릭트 관리 도구의 대상이 되지 않는다는 점이다. 
> 그렇기 때문에, gradle을 사용해 파일에 대한 의존성을 관리할 때에는 파일 이름에 올바른 버전을 명시하여 변경 사항을 추적할 수 있도록 하는 것이 매우 중요하다.

### 프로젝트 의존성

소프트웨어 프로젝트들은 종종 소프트웨어의 컴포넌트들와 모듈들로 분리되곤 한다. 이러한 분리는 관리와 소프트웨어들 사이의 강결합을 막는 역할을 수행한다.

모듈들은 동일한 프로젝트 내에서 코드를 재사용하기 위해 의존성을 정의할 수 있고, gradle은 이 모듈에 대한 정의를 gradle project 단위로 관리한다.

`project(":some:path)"`와 같은 방식의 표기점을 사용할 때 주의해야 하는 점은
- 의존의 대상이 되는 모든 프로젝트의 경로를 기억해야 한다.
- 프로젝트 경로를 변경할 때, 해당 프로젝트 의존성이 사용되는 모든 곳에서 의존성을 전부 수정해줘야 한다. 
	- 하나라도 빼먹게 된다면 문제가 발생하게 될 것이다.

Gradle 7 버전에서부터는 프로젝트 의존성을 설정하기 위한 Type-safe한 API를 제공한다.
이 Type-safe한 API는 IDE Completion을 제공하기 때문에 프로젝트의 실제 이름을 알 필요를 제거한다.

## compileOnly, runtimeOnly, implementation의 차이점

https://tomgregory.com/gradle-implementation-vs-compile-dependencies/

1. compileOnly: `compile` classpath에만 의존성을 추가한다.
2. runtimeOnly: `runtime` classpath에만 의존성을 추가한다.
3. implementation: `compile`과 `runtime` classpath 모두에 의존성을 추가한다.

적절한 classpath에 의존성을 추가하는 것은 다음과 같은 장점이 있다.
1. `compile` classpath가 더 적은 의존성을 가질 경우 **컴파일 속도가 빨라진다.**
2. `runtime` classpath에만 등장하는 의존성들에 대해서만 사용 가능하기 때문에, 실수로 정의되지 않은 클래스를 혼동하여 사용하는 일을 방지할 수 있다.
3. classpath들을 깔끔하게 관리하여 복잡도를 낮춘다.

## test에 대한 의존성 관리는 어떻게?

Gradle Java Plugin은 앞서 말했던 `compileOnly`, `runtimeOnly`, `implementation` 뿐만 아니라
- `testRuntimeOnly`: 
- `testImplementaton`: 
- `testCompileOnly`
를 제공한다. 기본적으로는 네이밍에서 유추할 수 있듯 각각 `compile` classpath, `runtime` classpath, 그리고 둘 다에 의존성을 추가하는 설정들이다. 

## api, implementation의 차이점

api는 의존하는 모듈을 상속하는 모듈에도 의존성이 전파가 되고,
implementation은 의존하는 모듈을 상속하는 모듈에 대해서는 의존성이 한번 래핑되어 관리된다.

때문에, api는 모든 프로젝트에 공통으로 사용되는 의존성이 아니라면 자제하는 것이 좋다.
프로젝트가 너무 많은 모듈에 의존하기 시작하면 관리해야 할 포인트들이 많아지기 때문이다. 
