---
title: "Java Enum은 언제 써야할까"
date: 2023-04-24T19:22:18+09:00
draft: false
author: redjen
---

많은 사람들이 enum 자체가 안티 패턴이고 사용을 신중하게 해야 한다고 말한다.

대부분의 경우에서는 enum은 종종 무분별하게 사용되기 떄문에 마치 사용을 지양해야 하는 것처럼 여겨지고 있다. 하지만 enum 자체가 나쁘다는 의미는 아니다. 올바르게 사용한다면 강력한 툴이 될 수 있기 때문이다.

Enum을 언제, 어떻게 적재적소에 쓸 수 있을지 알아봤다.

https://betterprogramming.pub/when-to-and-when-not-to-use-enums-in-java-8d6fb17ba978

## Enum

enum은 주로 상수의 집합을 표현하기 위해 사용된다. enum 키워드로 생성되고, 콤마로 구분된다.

```java
enum Houses {

GRYFFINDOR,
HUFFLEPUFF,
RAVENCLAW,
SLYTHERIN

}
```

다른 클래스처럼 단순한 문자열 뿐만 아니라 다른 클래스를 enum 안에 넣는 것도, 메서드를 넣는 것도 물론 가능하다.

```java
public enum Planet {

	MERCURY (3.303e+23, 2.4397e6),
	VENUS (4.869e+24, 6.0518e6),
	EARTH (5.976e+24, 6.37814e6),
	MARS (6.421e+23, 3.3972e6),
	JUPITER (1.9e+27, 7.1492e7),
	SATURN (5.688e+26, 6.0268e7),
	URANUS (8.686e+25, 2.5559e7),
	NEPTUNE (1.024e+26, 2.4746e7);
	
	private final double mass; // in kilograms
	private final double radius; // in meters


	Planet(double mass, double radius) {	
		this.mass = mass;
		this.radius = radius;
	}

	// universal gravitational constant (m3 kg-1 s-2)
	public static final double G = 6.67300E-11;

	double surfaceGravity() {
		return G * mass / (radius * radius);
	}

}
```

## Enum의 단점

### 다수의 switch case 문을 강제한다

enum이 나쁜 코드로 여겨지는 큰 이유 중 하나는 코드 전체에 있어서 swtich 문을 통해 분기를 해야 한다는 것이다.

switch 문을 통해 분기하는 것이 뭐가 안 좋은거냐? 할 수 있는데, switch 문은 다수의 OOP 원칙들을 깬다.
- 새 타입이 추가되었을 때, 함수의 크기가 더 커져야 하고 가독성은 줄어든다.
- SRP를 위반한다. 변경에 있어서 하나 이상의 이유가 생기기 때문이다.
- OCP를 위반한다. 새 타입이 추가될 때마다 변경이 생겨야 하기 때문이다.
- 똑같은 구조를 가지는 다른 함수들이 무한히 존재할 수 있기 때문이다. 
	- enum에 새 타입을 더하는 것은 코드에서 해당 enum의 쓰임을 전부 찾아서 새 분기를 해줘야 하는 것을 의미한다.

마틴 옹은 클린 코드에서 "단 하나의 switch 문만 사용하라"고 말했다.
> 타입을 선택할 때 하나 이상의 switch 문을 사용해서는 안된다. switch 문은 반드시 다른 switch 문을 대체하기 위한 polymorphic한 객체를 만들어야 하기 때문이다.

### 최적의 모델링 설계를 방해한다

이전 이슈와 비슷하게, enum 자체의 문제는 아니지만 enum은 디자인적인 측면에서 나쁜 결정을 내리는데 도움을 준다.

부모 자식 간의 관계로 나타내어질 수 있는 관계인 is - a 관계 또한 enum이 들어간 구조에서는 분리가 쉽지 않을 수 있기 때문이다. 

## 결론

즉 enum은 아래의 특수한 경우가 아니라면 사용을 자제해야 한다.
- 컴파일 타임에 가능한 모든 값에 대해 알고 있거나
- 어떤 것이 간단한 값으로 표현이 가능한 경우

