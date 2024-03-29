---
title: "Mvvm 패턴이란"
date: 2023-02-19T12:02:43+09:00
draft: false
author: redjen
---

서버 개발의 유구한 역사 속에서는 MVC 패턴을 많이 사용한다.

Model - View - Controller로 나누어진 MVC 구조는 사실 불편한 면이 없는 것은 아니지만 그럭저럭 기능들을 관심사에 따라 분리할 수 있고, 무엇보다도 관행으로 굳어진 면이 없지 않아 있어서 다른 사람들 또한 서버를 이런 방식으로 작성한다는 면에서 - 가독성과 프로젝트 초기 서버 구조를 파악하는데 실제로 큰 도움을 많이 받곤 한다.

그런데, MVC 패턴 말고도 MVVM 패턴이라는 말을 종종 접할 수 있어서 그게 뭐지? 하고 정리해보았다.

https://learn.microsoft.com/en-us/dotnet/architecture/maui/mvvm

## MVVM 패턴

MVVM 패턴은 3가지 중심 컴포넌트로 나누어진다.
1. View
2. View Model
3. Model

### 1. View
뷰는 구조, 레이아웃, 사용자가 화면에 보이는 모든 것을 정의한다.

뷰는 뷰 모델과 밀접한 연관을 가지는데,
- 뷰의 데이터 바인딩과 커맨드 동작들은 뷰 모델을 통해 이루어진다.
- 뷰 모델 내의 상태 변화는 뷰에 notification을 통해 전달된다.

### 2. ViewModel

뷰 모델은 뷰의 데이터들이 어떤 곳에 바인딩될지와, 상태 변화에 따른 notification을 뷰에 보내기 위한 속성과 커맨드들을 정의한다. 뷰 모델에 정의된 속성과 커맨드들은 UI 상에 나타나는 기능을 정의하지만, 뷰는 그 기능들 자체가 '어떻게 보여지는 지'에 대한 책임을 가지는 것에서 차이가 있다.

멀티 플랫폼 앱들은 UI 쓰레드를 논블락킹 상태로 두어 사용자 경험을 향상시켜야만 한다. 
> 따라서 뷰 모델은 I/O 작업에 대한 비동기 메서드들을 사용해서 속성들이 변화함에 따라 뷰에 이벤트를 발생시킨다.

각각의 뷰 모델들은 뷰가 쉽게 사용할 수 있는 형태의 데이터를 제공한다. 때문에 뷰 모델은 데이터 변환도 수행한다. 데이터 변환을 뷰 모델에서 수행하는 것은 뷰가 바인딩할 수 있는 데이터를 직접 제공한다는 측면에서 유리하다. 
- 뷰 모델에서 두 속성의 합쳐진 값을 제공해서 뷰가 데이터만 가져다 사용할 수 있게 하는 것이 예시이다.

> 데이터 변환은 하나의 레이어로 중앙화 시켜야 한다.

### 3. Model

모델 클래스들은 어플리케이션의 데이터를 캡슐화하는, 보이지 않는 클래스들이다.

> 모델은 어플리케이션의 비즈니스 도메인 모델과 검증 로직을 표현한다.

뷰에 대한 결합을 느슨하게 유지함으로써 (대신 뷰 모델에 대한 결합을 가져간다) 뷰가 변경되더라도 비즈니스 로직의 변경을 하지 않아도 된다는 점이 MVVM 패턴의 핵심인 것 같다.

## MVVM 패턴의 장점

- 모델의 구현체는 핵심 비즈니스 로직을 구현한다. 그리고 이 비즈니스 로직을 변경하는 것에는 리스크가 따른다.
  - 이 경우 뷰모델이 모델 클래스들에 대한 어댑터로 작용할 수 있다. 모델 코드를 메이저하게 변경하지 않고도 일종의 변경에 대한 효과를 누릴 수 있는 것이다.
- 개발자들은 뷰를 사용하지 않고 뷰 모델과 모델에 대한 유닛 테스트를 생성할 수 있다.
  - 뷰에 대한 기능 테스트를 수행하는 것은 일반적으로 다른 컴포넌트들에 대한 기능 테스트와는 성격이 상이하다.
  - 뷰와 거의 비슷한 기능을 하는 뷰모델은, 뷰에 대한 테스트를 하지 않고도 비슷한 효과를 누릴 수 있다.
- 어플리케이션의 UI는 뷰 모델과 모델 코드를 수정하지 않고도 수정될 수 있다.
  - 새로운 버전의 뷰는 기존 존재하던 뷰 모델과 호환이 되어야만 한다.
- 디자이너들과 개발자들이 독립적이고 동시적으로 컴포넌트에 대한 작업을 할 수 있다.
  - 디자이너들은 뷰에 더 집중할 수 있고, 개발자들은 뷰 모델과 모델 컴포넌트에 집중할 수 있다.

## 왜 MVVM 패턴인가?

https://stackoverflow.com/questions/2653096/why-use-mvvm

간단한 프로젝트의 경우, 뷰 모델 / 뷰의 분리는 불필요하고, 모델과 뷰를 사용해도 괜찮다.

하지만 프로젝트가 커지고, 화면 단에 대한 요구 사항이 급작스럽게 변화하는 상황이 빈번하게 발생한다면..?

때문에 모델과 뷰를 분리해야 할 필요성을 느끼게 되었다.
- 그런데 모델을 바로 뷰에서 사용하기에는 중간 과정들이 너무 많이 필요했다.
- 데이터 변환, 데이터 변경에 따른 비동기 이벤트, 데이터 바인딩.. 이 들을 모델과 뷰에서 전부 수행하기에는 너무 책임과 부담이 커진다. (화면 단에 대한 테스트도 매우 어렵다!)
- 따라서 뷰 모델을 중간 가교 역할로 두어 모델과 뷰에 대한 완충 역할을 하도록 했다.

