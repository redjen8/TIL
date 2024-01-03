---
title: "타입스크립트 핸드북 6장 - 클래스"
date: 2023-12-29T18:32:39+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/classes.html

- 컴포넌트의 재사용: 생산성을 높이기 위한 매우 중요하고 유용한 방법
- ES6 이전 JS에서는 이를 위해 함수 프로토타입 기반 상속을 사용
- ES6 이후 JS에서는 객체 지향적 클래스 기반의 접근 방식을 사용 가능
- TS는 JS 버전과 무관히 클래스 기반의 접근 기법을 사용할 수 있게 함

### 클래스

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

- 클래스 내부에서 클래스 멤버 참조 시 `this` 를 통해 참조 가능
- `new`를 사용해 클래스를 인스턴스화 한다.
- 사전 정의한 생성자를 호출해
  - 클래스 형태의 새로운 객체를 생성하고
  - 생성자를 실행해 생성된 객체를 초기화한다.

### 클래스의 상속

> 상속은 클래스 기반 프로그래밍의 가장 기본적인 패턴이다.

```ts
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

- `Animal`은 기반이 되는, 슈퍼 클래스이다.
- `Animal`로부터 파생된 `Dog` 클래스는 서브 클래스이다.
- 서브 클래스의 생성자 함수는 슈퍼 클래스의 생성자를 `super()`를 통해 호출해야 한다.
  - 이 때 생성자의 `this` 를 통해 프로퍼티에 접근하기 전에 `super()`를 먼저 호출해야 함에 유의해야 한다.
  - https://stackoverflow.com/questions/60689986/why-super-should-come-before-accessing-this
  - 서브 클래스에서 슈퍼 클래스의 초기화되지 않은 멤버를 참조하는 것을 막기 위함
  - 슈퍼 클래스는 서브 클래스의 존재를 몰라야 한다. 
    - 서브 클래스에서 프로퍼티 설정 후 `super()` 호출 시 JS의 작동 방식 때문에 슈퍼 클래스 생성자가 서브 클래스의 프로퍼티에 접근 가능하다.

### 접근 지정자

> TS의 클래스는 기본적으로 public으로 지정된다.

#### ECMAScript 비공개 필드

```ts
class Animal {
    #name: string;
    constructor(theName: string) { this.#name = theName; }
}

new Animal("Cat").#name; // 프로퍼티 '#name'은 비공개 식별자이기 때문에 'Animal' 클래스 외부에선 접근할 수 없습니다.
```

- JS 런타임에 내장되어 있는 문법
- 각각의 비공개 필드의 isolation을 잘 보장할 수 있는 방법

#### `private` 지정자

> `private` 지정자를 통해 클래스 외부에서 멤버에 접근 불가능하도록 설정할 수 있다.

- TS는 구조적인 타입 시스템
  - 두 타입을 비교할 때 모든 멤버의 타입이 호환된다면 **서로 호환가능한 타입이다**.
- 하지만 이는 `private` 또는 `protected` 접근 지정자가 붙은 타입 비교시 다르게 처리된다.
  - 호환된다고 판단되는 두 타입 중 하나가 `private` 멤버를 가지고 있다고 가정
  - 다른 한 쪽도 무조건 동일하게 `private` 멤버를 가지고 있어야 한다.
  - `protected`도 마찬가지

#### `protected` 지정자

> `protected` 지정자는 `protected`로 선언된 멤버를 파생된 서브 클래스 내에서 접근 가능하다는 점을 빼곤 `private` 지정자와 거의 똑같다.

생성자가 `protected`로 지정될 수 있음이 특이했다.

```ts
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee는 Person을 확장할 수 있습니다.
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // 오류: 'Person'의 생성자는 protected 입니다
```

- 클래스를 포함하는 클래스 외부에서 인스턴스화는 불가능 (`new`를 사용한`)
- 클래스 외부에서 확장은 가능하다. (`extends`를 통한)

#### `readonly` 지정자

> `readonly` 지정자를 사용해서 프로퍼티를 읽기 전용으로 만들 수 있다. (해당 프로퍼티들은 선언 또는 생성자에서 초기화 필요)

### 매개변수 프로퍼티

```ts
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 오류! name은 읽기전용 입니다.
```

위 예시는 다음과 같이 프로퍼티의 선언과 초기화를 동시에 할 수 있다.

```ts
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

- 매개변수 프로퍼티는 접근 지정자 또는 `readonly` 또는 둘 모두를 생성자 매개변수에 prefix로 붙여서 선언 가능
- 매개변수 프로퍼티에 `private`를 붙이면 `private` 멤버를 선언과 동시에 초기화
- 마찬가지로 `public`, `protected`, `readonly`도 동일하게 동작함

### 접근자

> TS는 각 객체의 멤버에 대한 접근자를 지원한다.

```java
const fullNameMaxLength = 10;

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (newName && newName.length > fullNameMaxLength) {
            throw new Error("fullName has a max length of " + fullNameMaxLength);
        }

        this._fullName = newName;
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```

- TS가 기본적으로 지원하는 getter / setter를 통해서 입력에 대한 validation을 수행할 수 있다.
- 접근자는 ES5 이상을 출력하도록 컴파일 옵션을 설정해야 함에 유의
- `get` 또는 `set`이 없는 접근자는 자동으로 `readonly`로 유추된다.
  - 프로퍼티 내 사용자들이 변경할 수 없음을 알게 해주기 때문에 `.d.ts` 파일 생성시 유용

### 전역 프로퍼티

> TS는 `static` 지정자를 통해 인스턴스가 아닌 클래스 자체에서 보이는 전역 멤버 생성 가능

```java
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### 추상 클래스

- 추상 클래스는 다른 클래스들이 파생될 수 있는 기초 클래스
- 직접 인스턴스화 불가능
- 인터페이스와는 달리 멤버에 대한 세부 구현 정보 포함 가능
- `abstract` 키워드를 통해 추상 클래스를 선언하고, 추상 메서드를 정의하는데 사용가능

```ts
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log("Department name: " + this.name);
    }

    abstract printMeeting(): void; // 반드시 파생된 클래스에서 구현되어야 합니다.
}

class AccountingDepartment extends Department {

    constructor() {
        super("Accounting and Auditing"); // 파생된 클래스의 생성자는 반드시 super()를 호출해야 합니다.
    }

    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }

    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}

let department: Department; // 추상 타입의 레퍼런스를 생성합니다
department = new Department(); // 오류: 추상 클래스는 인스턴스화 할 수 없습니다
department = new AccountingDepartment(); // 추상이 아닌 하위 클래스를 생성하고 할당합니다
department.printName();
department.printMeeting();
department.generateReports(); // 오류: 선언된 추상 타입에 메서드가 존재하지 않습니다
```

[인터페이스 vs 추상 클래스의 차이점](https://stackoverflow.com/questions/50110844/what-is-the-difference-between-interface-and-abstract-class-in-typescript)
- 인터페이스는 프로퍼티들과 인터페이스를 구현하는 객체가 할 수 있는 행동을 사전에 정의하는 것
  - 인터페이스를 구현하는 것은 자유이다. 때문에 런타임에 존재하지 않는다. (`instanceof`로도 당연히 검사 불가능하다)
- 추상 클래스는 온전하지 못한 구현체이다. 
  - 추상 클래스는 (추상 메서드만 존재하더라도) 런타임에 존재한다.

### 생성자 함수

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

위 타입스크립트 코드는 아래와 같은 자바스크립트 코드로 트랜스파일된다.

```js
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

위 타입스크립트 코드를 조금 수정해보면 재밌는 것을 알 수 있는데,

```ts
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet()); // "Hello, there"

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet()); // "Hey there!"
```

`greeterMaker` 변수는 `Greeter` 라는 심볼의 타입을 제공한다.
- 해당 타입은 `Greeter` 클래스 인스턴스를 만드는 생성자를 포함한다.
- 아울러 `Greeter`의 모든 static 멤버를 포함한다. (때문에 static 멤버는 생성자 없이 호출 가능하다)
- `new`를 사용한다면 새로운 인스턴스를 호출할 수 있다. 