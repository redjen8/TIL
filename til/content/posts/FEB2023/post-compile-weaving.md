---
title: "Post Compile Weaving"
date: 2023-02-22T23:08:49+09:00
draft: false
author: redjen
---

## AOP란

AOP는 교차 관심사의 분리를 허용함으로써 프로그램의 모듈성을 높이는 것을 목표로 하는 프로그래밍 패러다임이다. 

AOP가 사용되는 대표적인 예시는, Spring의 Slf4j와 같은 로깅 기능이 있다.
로그를 찍는 행위 자체는 많은 곳에 들어가야 하는데, 로그를 찍는 동일한 코드를 여러 군데에 위치시킨다면 깔끔한 코드를 짜기 어렵다.

따라서, AOP의 목적은
- 코드를 수정하지 않고 기존 코드에 추가 동작을 추가한다.
- 대신 어떤 코드를 수정해야 하는지를 따로 정의 / 선언한다.

AspectJ는 자바 프로그래밍 언어의 익스텐션으로 관심사와 교차 관심사의 짜임을 모두 구현한다.

## AspectJ

AspectJ에는 3가지 코어 컨셉이 있다.
1. Join Point :  메서드나 특정 속성에 대한 접근 등 스크립트가 실행되는 지점이다.
2. Pointcut : Join Point와 매칭되는 정규표현식이다. Advice가 Pointcut 표현식과 연관되어 Pointcut과 매칭되는 모든 Join Point를 실행한다.
3. Advice : 특정 Join point에서 Aspect에 의해 실행되는 행위이다.

## Weaving

AspectJ를 사용한 weaving에는 여러 종류가 있다.
1. Compile Time Weaving
2. Post Compile Weaving
3. Load Time Weaving

### 1. Compile Time Weaving

컴파일 타임 위빙은 가장 간단한 위빙의 종류이다. 
- Aspect와 Aspect가 사용되는 모든 코드를 가지고 있을 때 사용할 수 있다.

AspectJ 컴파일러는 소스로부터 컴파일을 하고, 위빙된 클래스를 출력으로 내보낸다.
그 후에 코드가 실행될 때, 위빙 프로세스에 대한 출력 클래스가 다른 자바 클래스들처럼 JVM 안으로 로드되는 방식으로 동작한다.

AspectJ Development Tool을 사용하면 Crosscutting하는 관심사를 시각화하여 볼 수 있기 때문에, Pointcut 세분화 개발 중 디버깅에 유용하게 사용할 수 있다. 또한 코드가 배포되기 전에 Aspect가 결합된 효과도 시각화할 수 있다.

### 2. Post Compile Weaving

만약 내가 작성한 코드가 아닌, 외부 라이브러리의 코드가 실행될 때 뭔가 액션을 취하고 싶다면 어떻게 해야할까?
위빙 대상을 디 컴파일 후에 다시 컴파일하여 사용할 수도 없고,, 여러모로 곤란하다. 그래서 나온 것이 포스트 컴파일 위빙이다.

위빙 대상이 되는 Aspect들은 소스 코드에 존재할 수도, 바이너리한 형태로, 또는 Aspect에 의해 만들어져서 존재할수도 있다. 

일반적으로 개발환경의 IDE들은 컴파일한 클래스 파일들과 import한 라이브러리들을 트래킹하고 있기 때문에 IDE 레벨에서의 Post Compile Weaving을 허용하는 옵션이 존재한다.

https://stackoverflow.com/questions/65119382/unable-to-enable-post-compile-weave-mode-in-intellij-for-a-module

> 외부 라이브러리의 특정 기능이 실행되는 것에 맞춰서 나의 어플리케이션의 특정 기능이 실행되길 원한다면 Post Compile Weaving을 사용하는 것이 하나의 방법이 될 수 있다.

### 3. Load Time Weaving

로드 타임 위빙은 클래스 로더가 클래스 파일을 로드하고, 클래스를 JVM에 정의할 때까지 미뤄지는 간단한 바이너리 위빙이다.

이 위빙을 사용하기 위해서는 한 개 이상의 위빙 클래스 로더가 필요하다. 이 클래스 로더들은 런타임 환경에 명시적으로 제공되거나 Weaving agent를 사용해서 활성화될 수 있다.