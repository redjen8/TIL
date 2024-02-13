---
title: "타입스크립트 핸드북 13장 - 다형성 this 타입"
date: 2024-02-13T22:21:31+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/advanced-types.html#%EB%8B%A4%ED%98%95%EC%84%B1-this-%ED%83%80%EC%9E%85-polymorphic-this-types

```ts
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
    // ... 다른 연산들은 여기에 작성 ...
}

let v = new BasicCalculator(2)
            .multiply(5)
            .add(1)
            .currentValue();
```

```ts
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
    // ... 다른 연산들은 여기에 작성 ...
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

위 `BasicCalculator` 클래스는 `this` 타입을 사용하기 때문에 다음의 이점이 있다.
- 클래스 상속이 가능하다.
- 이를 상속하는 새로운 클래스가 아무런 변경 없이 기존 메서드를 사용할 수 있다.

빌더 패턴에서 많이 사용하는 방법이다.

https://siosio3103.medium.com/typescript-%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-0-builder-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4-90552ae0b763

클래스에 대한 인스턴스를 만들 때, 빌더 패턴을 사용한다면 다음의 이점이 있다.
- 코드 가독성이 올라간다
  - 인자의 순서, 항목과는 무관하게 인스턴스 생성 코드만 봐도 어떤 객체를 만드는지 알 수 있다는 장점
- 객체의 생성을 전담하는 클래스를 생성함으로써의 관심사 분리가 가능하다
- 새로운 멤버가 추가되었을 때에도 용이하다
  - 빌더 클래스에서 리턴되는 클래스의 비즈니스적인 로직의 수정 없이 추가된 멤버에 영향이 있는 순수한 비즈니스적인 코드 변경만 이뤄지면 되기 때문
