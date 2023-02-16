---
title: "Java Generic 트러블슈팅"
date: 2023-02-16T19:10:28+09:00
draft: false
author: redjen
---

약간 헷갈리던 개념이었는데, 이번 기회에 트러블 슈팅을 하면서 제대로 정리를 하고 간다..!
다음 질문으로부터 파생되었던 개념이었다.

> 왜 자바 컬렉션 내의 특정 객체에 대해서는 instanceof 으로 검사할 수 없을까?

## Generic 정보는 컴파일 시점에 없어진다.

https://stackoverflow.com/questions/19253174/are-generics-removed-by-the-compiler-at-compile-time

- 컴파일러는 제네릭 타입에 대한 정보를 컴파일 과정에서 내부적으로 제거한다.
- 만약 이 과정에서 타입 관련한 오류가 있을 경우, 컴파일 오류를 발생시킨다.
- 검증이 마친 후에는 제네릭에 대한 정보는 바이트 코드 단위에서는 찾아볼 수 없다. (type erasure)

이는 타입을 리플렉션을 통해 들여다 봤을 때 더욱 극명하게 들어난다.
- 모든 인터페이스, 클래스, 함수들은 모두 제네릭하지 않게 변경된다. 
- 리플렉션 API가 런타임에 제네릭에 대한 정보를 접근할 수 있도록 할 수 있지만,
- JVM 레벨에서 리플렉션을 통해 정확한 제네릭의 타입을 체크하는 것은 불가능하다. 

리플렉션을 사용할 경우 타입에 대한 체크를 strict하게 할 수 없다.
`List<String>` 타입의 변수를 `List<Integer>`로 할당하는 것은 컴파일 에러를 발생시키지만 리플렉션 API의 `Method.invoke()`를 사용할 경우 클래스 내부의 메서드를 실행하듯이 하여 '꼼수로' 할당할 수 있다..

물론 그 후에 데이터를 접근할 때에는 타입 캐스팅 에러가 발생하게 된다. 런타임에!!

## instanceof에 대해

`instanceof` 는 런타임 시점에 자바 객체의 타입을 체크하기 위한 operator이다.

자바 언어 스펙에 따르면, `instanceof`는 다음의 특징을 가진다.
- 좌변은 reference 나 null을 가져야 한다. (primitive 타입은 불가능)
- 우변은 제네릭 클래스일지라도 제네릭하지 않은 타입을 가져야 한다.
- 좌변의 컴파일 시점에서의 타입은 우변으로의 타입 캐스팅을 허용해야 한다.
- `instanceof`는 대부분의 경우에서 오직 다운 캐스팅만 가능하다. (아닌 경우도 있나?) 

## 정리하자면

- 제네릭의 타입 정보는 컴파일 시점에 변환된다. 바이트 코드에는 제네릭에 대한 정보를 찾을 수 없다.
- `instanceof`는 런타임에 객체의 타입을 체크하기 때문에, 컴파일 시점에 지워진 제네릭에 대한 정보를 컴파일 전에 체크할 수 없다.
	- 때문에 자바 언어 스펙에 `instanceof`의 우변에는 특정 타입에 대한 제네릭을 사용할 수 없도록 되어 있다.
	- `instanceof`는 오직 Unbound wildcard (`List<?>`등) 에 대한 타입 체킹이 가능하다.

## instanceof을 리플렉션의 일종으로 볼 수 있을까?

https://softwareengineering.stackexchange.com/questions/106362/is-java-instanceof-operator-considered-reflection-and-what-defines-reflection

이론적으로는: `instanceof`는 리플렉션의 형태를 띈다.
> 컴퓨터 과학에서 리플렉션은 컴퓨터 프로그램이 런타임에 자기 자신의 구조를 관찰하고, 수정할 수 있는 일련의 행동 프로세스이다.

하지만 실제로는: 프로그래머가 리플렉션에 대해 얘기하는 것은 보통 변수가 특정 타입인지 체크하는 것보다 훨씬 많은 범위를 커버한다. 리플렉션은 polymorphism 없이는 설명할 수 없다.

> type introspection인 instanceof는 리플렉션의 일종으로 볼 수 없다.
