---
title: "타입스크립트 핸드북 9장 - (고급 타입) 교차 타입, 유니언 타입"
date: 2024-01-24T19:19:27+09:00
draft: false
author: redjen
---

### 교차 타입

https://typescript-kr.github.io/pages/advanced-types.html#%EA%B5%90%EC%B0%A8-%ED%83%80%EC%9E%85-intersection-types


> 여러 타입을 하나로 결합한 타입.

즉 기존의 타입들을 하나로 합쳐 필요한 모든 기능을 가진 하나의 타입을 얻을 수 있다.

#### 믹스인이란

https://en.wikipedia.org/wiki/Mixin

- 객체 지향 프로그래밍에서는 일반적으로 상속을 통해 다른 클래스의 메서드를 사용할 수 있다
- 하지만 상속은 개념 상 has-a 관계가 아닌 is-a 관계여야 한다..
	- `Person` 클래스는 `Animal` 클래스의 자식 클래스 (`Person` is-a `Animal`)
	- `Person` 클래스는 `Toy` 클래스의 자식 클래스가 될 필요는 없다. (`Person` has a `Toy`)
- 믹스인은 **부모 클래스가 아니어도 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스**
	- 때문에 상속이 아닌 포함으로 설명된다 (included, not inherited)
	- 구현된 메서드가 있는 인터페이스로도 볼 수 있다. 이 경우 DIP가 적용된 예로 본다.

믹스인을 사용하게 된다면 다음의 이점을 가진다.
- 코드 재사용을 장려한다.
- 다중 상속으로 인한 상속 모호성을 피할 수 있다.
- 언어에서 다중 상속을 지원하지 않는 문제를 해결 할 수 있음

#### 믹스인을 어떻게 사용할 수 있나?

https://www.typescriptlang.org/docs/handbook/mixins.html

클래스 상속과 함께 제네릭을 사용해서 기본 클래스를 확장하기 위해서 사용할 수 있다.

```ts
class Sprite {
	name = "";
	x = 0;
	y = 0;
	
	constructor(name: string) {
		this.name = name;
	}

	getName(): string {
		return this.name;
	}
}
```

```ts
// To get started, we need a type which we'll use to extend
// other classes from. The main responsibility is to declare
// that the type being passed in is a class.

type Constructor = new (...args: any[]) => {};

// This mixin adds a scale property, with getters and setters
// for changing it with an encapsulated private property:

function Scale<TBase extends Constructor>(Base: TBase) {

	return class Scaling extends Base {
		// Mixins may not declare private/protected properties
		// however, you can use ES2020 private fields
		_scale = 1;
		
		setScale(scale: number) {
			this._scale = scale;
		}

		get scale(): number {
			return this._scale;
		}
	};
}

// Compose a new class from the Sprite class,
// with the Mixin Scale applier:

const EightBitSprite = Scale(Sprite);
const flappySprite = new EightBitSprite("Bird");
flappySprite.setScale(0.8);
console.log(flappySprite.scale);  // 0.8
console.log(flappySprite.getName()); // "Bird"
```

위 예시에서는 기반이 되는 클래스 (`Sprite`)를 확장하기 위해서 다음을 사용했다.
- 타입 (`Constructor`)
- 클래스 표현식을 리턴하는 팩토리 함수 (`Scale()`)

때문에 기반 클래스인 `Sprite`의 상속 없이 클래스의 인스턴스에 신규 멤버와 신규 메서드를 사용할 수 있었다.

#### 믹스인과 교차타입

```ts
function extend<First, Second>(first: First, second: Second): First & Second {
    const result: Partial<First & Second> = {};
    for (const prop in first) {
        if (first.hasOwnProperty(prop)) {
            (result as First)[prop] = first[prop];
        }
    }
    for (const prop in second) {
        if (second.hasOwnProperty(prop)) {
            (result as Second)[prop] = second[prop];
        }
    }
    return result as First & Second;
}

class Person {
    constructor(public name: string) { }
}

interface Loggable {
    log(name: string): void;
}

class ConsoleLogger implements Loggable {
    log(name) {
        console.log(`Hello, I'm ${name}.`);
    }
}

const jim = extend(new Person('Jim'), ConsoleLogger.prototype);
jim.log(jim.name);
```

위 예제에서,
- `extend()` 메서드는 두 개의 인자를 받아, 두 인자의 타입을 교차한 타입을 리턴한다.
- `extend()` 메서드는 두 개의 인자의 멤버들을 전부 가진다. 
- 때문에 `extend()` 메서드로 생성된 `Person & ConsoleLogger` 타입 `jim` 인스턴스는
	- `Person`  클래스에 존재하는 `name` 멤버에 접근할 수 있다.
	- `Loggable` 인터페이스에 존재하고, `ConsoleLogger` 클래스에 의해 구현된 `log()` 메서드를 사용할 수 있다.

### 유니언 타입

https://typescript-kr.github.io/pages/advanced-types.html#%EA%B5%90%EC%B0%A8-%ED%83%80%EC%9E%85-intersection-types

교차 타입과 문법은 매우 비슷하지만, 아주 다르게 사용된다.

```ts
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "    Hello world"를 반환합니다.
```

두 번째 인자가 `any` 타입으로 선언되는 것보다는, 처리할 수 있도록 타입을 **좁히는** 것이 중요하다.

유니언 타입을 값으로 가지고 있다면, 유니언에 있는 모든 타입에 공통인 멤버에만 접근 가능한 점에 유의.