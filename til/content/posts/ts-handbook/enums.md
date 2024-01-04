---
title: "타입스크립트 핸드북 7장 - 열거형"
date: 2024-01-03T22:11:09+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/enums.html

### Numeric Enums

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right,
}
```

- `Up`이 1부터 초기화되기 시작했기 때문에 그 뒤의 값들은 auto increment 된 값을 가진다 (각각 1, 2, 3, 4)
- 멤버를 전부 초기화하지 않는다면 0부터 시작한다.
- 숫자 열거형은 아래에서 언급할 computed and constant member를 섞어서 사용할 수 있다.
  -  computed member가 먼저 온다면 뒤의 값은 계산 전 초기화가 필요한 상태이기 때문에 오류 발생


### String Enums

문자열 열거형은 런타임에서 동작이 조금 상이하다.

> 문자열 열거형에서 각 멤버들은 문자열 리터럴 또는 다른 문자열 열거형의 멤버로 상수 초기화가 반드시 필요하다. 

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

- numeric enum처럼 auto increment 하지는 않지만 직렬화에 유리
- 가독성에 보다 유리

### Heterogeneous Enums

Numeric + String Enum을 동시에 사용하는 것. 권장하지 않는다.

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

### Computed and Constant Members

각 열거형의 멤버는 1) 상수이거나 2) 계산된 값일 수 있다.

아래의 경우는 상수 열거형 표현식이다.
1. 리터럴 열거형 표현식인 경우
2. 이전에 정의된 다른 상수 열거형 멤버에 대한 참조
3. 괄호로 묶인 상수 열거형 표현식
4. 상수 열거형 표현식에 단항 연산자를 사용한 경우
5. 상수 열거형 표현식을 이중 연산자의 피연산자로 사용할 경우 (표현식 값이 `NaN`이거나 `Infinity`이면 컴파일 시점에 오류 발생)

이외의 모든 열거형 멤버는 계산된 것으로 간주한다. (computed)

```ts
enum FileAccess {
    // 상수 멤버
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // 계산된 멤버
    G = "123".length
}
```

### Union Enums and Enum Member Types

리터럴 열거형 멤버는 초기화 값이 존재하지 않거나 아래 값들로 초기화 되는 멤버들
- 문자 리터럴 (`"foo"`)
- 숫자 리터럴 (`1`)
- 숫자 리터럴에 단항 연산자 `-`가 적용된 경우 (`-100`)

> 열거형의 모든 멤버가 리터럴 열거형 값을 가지면 특별한 의미로 쓰이게 된다.

#### 열거형 멤버를 타입처럼 사용하기

```ts
enum ShapeKind {
    Circle,
    Square,
}

interface Circle {
    kind: ShapeKind.Circle;
    radius: number;
}

interface Square {
    kind: ShapeKind.Square;
    sideLength: number;
}

let c: Circle = {
    kind: ShapeKind.Square, // 오류! 'ShapeKind.Circle' 타입에 'ShapeKind.Square' 타입을 할당할 수 없습니다.
    radius: 100,
}
```

- 특정 멤버는 열거형 멤버의 값만 가지게 할 수 있다. 마치 타입처럼...

#### 열거형 타입 자체를 열거형 멤버의 유니언으로 사용하기

```ts
enum E {
    Foo,
    Bar,
}

function f(x: E) {
    if (x !== E.Foo || x !== E.Bar) {
        //             ~~~~~~~~~~~
        // 에러! E 타입은 Foo, Bar 둘 중 하나이기 때문에 이 조건은 항상 true를 반환합니다.
    }
}
```

- 유니언 타입 열거형을 사용하면 열거형 자체에 존재하는 값들의 목록을 컴파일러한테 알려주는 셈
- 때문에 값을 잘못 비교하는 버그를 컴파일 시점에 방지할 수 있다
- `x`가 `E.Foo` 가 아니라면 (short circuit) if 안의 코드가 무조건 실행된다

### Enums at runtime and compile time

> 열거형은 런타임에 존재한다.

> 열거형이 런타임에 존재하는 실제 객체여도 `keyof` 키워드를 사용할 때에는 객체와 다르게 동작하기 때문에 조심해야 한다.

- `keyof typeof`를 사용한다면 모든 열거형의 키를 문자열로 나타내는 타입을 가져온다.

```ts
enum LogLevel {
    ERROR, WARN, INFO, DEBUG
}

/**
 * 이것은 아래와 동일합니다. :
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
    const num = LogLevel[key];
    if (num <= LogLevel.WARN) {
       console.log('Log level key is: ', key);
       console.log('Log level value is: ', num);
       console.log('Log level message is: ', message);
    }
}
printImportant('ERROR', 'This is a message');
```

### Reverse Mappings

> 숫자 열거형 멤버는 정방향과 역방향 매핑 정보를 모두 저장하는 객체로 컴파일된다.

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

위 타입스크립트 코드는 아래 자바스크립트 코드로 컴파일된다. 

```js
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

문자열 열거형은 역매핑을 생성하지 않는다! 유의하자

### `const` Enums

열거형 값에 접근할 때 추가 생성된 코드 또는 추가 간접 참조를 피하기 위해 `const` 열거형을 사용할 수 있다.
```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

- 컴파일 될 때, 일반적인 열거형과는 달리 완전히 제거된다.
- 컴파일 될 때 사용하는 공간에 인라인됨으로써 기능한다. (계산된 멤버를 가지고 있지 않기 때문에 이게 가능)

### Ambient Enums

> 이미 존재하는 열거형 타입의 모습을 묘사하기 위해 사용

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

- 일반적인 열거형에서 초기화되지 않은 멤버가 상수로 간주하는 멤버 뒤에 있다면 해당 멤버도 상수로 간주
- 하지만 ambient 열거형에서 초기화되지 않은 멤버는 항상 계산된 멤버로 간주된다

### Java Enum과의 비교

```java
enum InstanceType {
	REAL("REAL"),
	DEV("DEV"),
	LOCAL("LOCAL");
	private final String code;
  
	InstanceType(String instanceType) {
		this.code = instanceType;
  	}
  
	public String getCode() {
		return "code: " + code;
    }
    
    public boolean isExternal() {
        if (code.equals("REAL")) {
            return true;
        } else {
            return false;
        }
    }
}

class Main {
	public static void main(String[] args) {
      	InstanceType real = InstanceType.REAL;
    	System.out.println(real.getCode());
    	System.out.println(real.isExternal());
	}
}
```

자바에서 Enum은 내부적으로 `java.lang.Enum` 클래스를 상속 받는 클래스이기 때문에 위처럼 내부 메서드를 가지는 것이 가능하다.
- 때문에 자바의 Enum 내부 필드는 `static final`이 된다.

```ts
enum InstanceType {
    REAL = "REAL",
    DEV = "DEV",
    LOCAL = "LOCAL"  
}

const isExternal = (instanceType: InstanceType) => {
    if (instanceType === InstanceType.REAL) {
        return true;
    } else {
        return false;
    }
}

console.log(isExternal(InstanceType.DEV));
console.log(isExternal(InstanceType.REAL));
```

반면 타입스크립트는 내부 메서드 지원이 되지 않고, 순수한 값의 집합으로만 표현 가능하다.
- 때문에 값에 대한 집합과 메서드 일체를 분리하여 작성해야 한다.