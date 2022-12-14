---
title: "Java Stream 이란"
date: 2022-12-31T16:30:45+09:00
draft: false
author: redjen
---

# Java Stream이란?

Java 8 API에서 새로 추가된 기능으로, 스트림을 사용하면 선언형으로 컬렉션 데이터를 처리할 수 있다. 스트림을 사용하면 멀티 쓰레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다는 장점이 있다.

선언형 : 데이터를 처리하는 임시 구현 코드 대신 질의로 표현 가능한 데이터 형

### 장점

- 선언형 : 더 간결하고 가독성 좋은 코드 작성
- 조립 가능 : 유연성 좋은 코드 작성
- 병렬화 : 성능 향상 관점에서의 이득

### 단점

- 디버깅이 힘들다
- 재사용 불가능

### Stream이 재사용 불가능한 이유?

Stream은 다운로드 받은 동영상을 재생하는 것이 아니라 유튜브와 같은 플랫폼에서 동영상을 스트리밍하는 것에 가깝다. 설계 철학 자체가 생산자 중심이 아니라 소비자 중심의 데이터 처리를 목적으로 하다보니 재사용이 불가능하도록 설계되었을 것.

[찾아보니](https://stackoverflow.com/questions/28459498/why-are-java-streams-once-off/28513908#28513908) 이유를 더 자세히 알 수 있었다. 
여러 개의 내부 반복자를 두어서 Stream을 재사용 가능하게 만드는 것은 정말 좋은 방법인지에 대한 의문이 있었고 예측하지 못한 결과를 낳을 수 있다는 결과를 도출했기 때문이다.
예시로 들었던 반복 가능한 것들은 컬렉션으로 처리 가능하지만,
네트워크를 통해서 데이터를 읽어오는 것들은 다시 요청을 하지 않는 한 다시 일어나지 않는다. 파일에서 순차적으로 큰 데이터를 읽는 경우에도 이전 데이터를 다시 읽기 위해서는 메모리에 따로 저장하지 않았다면 다시 파일을 읽어야 한다.

## Stream이란

데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소
- 연속된 요소 : 특정 요소 형식으로 이뤄진 연속된 값 집합의 인터페이스 제공. 컬렉션의 주제는 **데이터**이고 스트림의 주제는 **계산**이다. 
- 소스 : 컬렉션, 배열, IO 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. 정렬된 컬렉션으로 스트림 생성하면 -> 정렬 유지 (순서가 불변)
- 데이터 처리 연산 : 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연산을 지원한다.

### 스트림의 중요 특징

1. 파이프라이닝 : 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환. Laziness와 Short circuit와 같은 최적화가 가능한 이유
2. 내부 반복 : iterator를 사용하여 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원

## Stream의 주요 연산

- filter : 람다를 인수로 받아 스트림에서 특정 요소 제외
- map : 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보 추출
- limit : 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 truncate
- collect : 스트림을 다른 형식으로 변환. 

## 스트림과 컬렉션의 차이

**데이터를 언제 계산하느냐?** 에서 큰 차이를 가진다.

컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료 구조.
- 컬렉션의 모든 요소는 **컬렉션에 추가하기 전에 계산**되어야 함
- 컬렉션은 생산자 중심 (supplier-driven)
- 컬렉션은 특정 시간에 모든 것이 존재하는 공간에 흩어진 값

반면 스트림은 이론적으로 **요청할 때에만 요소를 계산하는** 고정된 자료 구조.
- 사용자가 요청하는 값만 스트림에서 추출하고, 결과적으로 producer - consumer 관계를 형성하게 됨
- 스트림은 소비자 중심 (demand-driven manufacturing)
- 스트림은 시간적으로 흩어진 값의 집합. 
