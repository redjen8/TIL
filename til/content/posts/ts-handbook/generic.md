---
title: "타입스크립트 핸드북 8장 - 제네릭"
date: 2024-01-10T22:00:00+09:00
draft: false
author: redjen
---
https://typescript-kr.github.io/pages/generics.html

> 재사용 가능한 컴포넌트를 생성하는 하나의 유용한 방법

### 제네릭 처음 접하기

```ts
function identity(arg: number): number {
    return arg;
}
```

`number` 말고도 다른 타입을 인자로 받는 함수를 만들고 싶어~

```ts
function identity(arg: any): any {
    return arg;
}
```

두 코드의 차이점은 무엇일까?
- 인자의 타입이 `number`인 것과 `any` 인 것
- `any`를 쓰게 된다면 함수 내부에서 인수의 타입을 추론 불가 (타입이 좁혀지는게 아니라 더 큰 타입으로 뭉개진다)
- 함수 내부에서 인수의 타입을 추론 가능하면서 다른 타입에 대한 함수를 만들고 싶으면 그 때마다 함수를 새로 만들어야 하나? 
	- ex. `identityForNumber(arg: number)`, `identityForString(arg: string)`
	- 이건 너무 비효율적이다

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

그래서 등장한 제네릭
- 위 코드에서 `T`라는 타입 변수를 통해 **함수 내부에 전달되는 인수 타입을 캡쳐할 수 있다**
- `T`는 고정된 문법은 아님.. `G`든 `A`든 마음대로 사용 가능
	- 하지만 `Type`의 의미로 `T`를 많이 쓰는 것 같다

#### 제네릭을 사용하는 방법

```ts
let output = identity<string>("myString");
```

- 위 에시에서는 `T`를 string으로 명시적으로 전달
```ts
let output = identity("myString");
```
- 위 예시에서는 굳이 명시적으로 타입 정보를 전달하지 않음
- 컴파일러가 타입 인수를 직접 추론하게 한다
- 코드의 가독성 증가
- 하지만 복잡한 타입을 사용하게 되어서 컴파일러도 모를 때는 명시적으로 타입 전달해줘야 할지도

### Working with Generic Type Variables

> 제네릭을 통해 전달된 타입에는 멤버가 없지만, 타입을 감싸는 녀석의 멤버에는 접근이 가능하다

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // 배열은 .length를 가지고 있습니다. 따라서 오류는 없습니다.
  return arg;
}
```

- 위와 같이 전체 타입 변수를 쓰지 않고 하나의 타입으로써 제네릭 타입 변수 `T`를 사용할 수도 있다
- 코드의 유연함.. 제네릭으로 전달되는 타입에 해당 멤버가 있다면 접근이 가능하다 (배열이나 `Array`처럼)

### Generic Types

> 함수 자체의 타입으로 제네릭을 쓸 수도 있고, 제네릭을 사용해서 인터페이스를 만들 수 있다

```ts
interface GenericIdentityFn {
  <T>(arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```
1번

```ts
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```
2번

두 예시에서 아래 코드가 위 코드에 비해 가지는 강점은
- `GenericIdentityFn` 제네릭 함수를 사용할 때 메서드 시그니쳐가 사용할 타입을 효과적으로 전달 가능 (`number`)
- 타입 매개 변수를 호출 시그니처에 바로 넣는 두 번째 예시가 타입의 제네릭을 설명할 때 훨씬 도움이 된다

### Generic Classes

인터페이스에서 사용하는 것과 매우 비슷하다.

```ts
class Log<A> {

	logType: A;
	content: string;

	static printFormat: string = "Log: ";
	
	setContent(content: string): void {
		this.content = content;
	}
	setType(logType: A): void {
		this.logType = logType;
	}

	print(): string {
		return Log.printFormat + this.content;
	}
}

let testLog = new Log<number>();
testLog.setContent("warn");
testLog.setType(22);
console.log(testLog.print(), testLog.logType);
```

다만 위 예시에서, 정적 멤버인 `printFormat`은 제네릭의 타입을 가질 수 없다
- `static printFormat: A` 처럼 전달이 불가능
- 왜냐면 클래스는 정적 멤버와 인스턴스 멤버가 나눠져 있기 때문

### Generic Constraints

앞서 살펴 본 제네릭 타입은 모든 타입이 될 수 있는 것처럼 보였지만 (마치 `any`), 제네릭으로 전달되는 타입 또한 narrowing 할 수 있다

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length); 
    return arg;
}
```

- `<T extends Lengthwise>` 가 아니라 `<T>`를 쓴다면 컴파일러는 해당 타입에 `length` 멤버가 있는지 모른다
- 하지만 제약 조건을 통해 이젠 모든 타입이 아니라 `Lengthwise` 타입을 상속하는 타입에 대해서만 동작하도록 제한할 수 있음

### Using Type Parameters in Generic Constraints

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // 성공
getProperty(x, "m"); // 오류: 인수의 타입 'm' 은 'a' | 'b' | 'c' | 'd'에 해당되지 않음.
```

- 하나의 타입 매개변수(`T`)로 제한을 건 다른 타입 매개변수(`K`)를 선언할 수 있음
- 위 예시는 `obj`에 존재하지 않는 프로퍼티를 가져오지 않도록 제약 조건을 걸었음

### Why Use Generics

https://docs.oracle.com/javase/tutorial/java/generics/why.html

- 제네릭은 클래스, 인터페이스, 함수를 정의할 때 **타입을 파라미터로 가능하게 한다**
- 타입 파라미터를 쓰게 된다면 서로 다른 입력이 있어도 동일한 코드를 재사용하게 할 수 있다
- 즉 제네릭을 사용하지 않는 것에 비해서
	- 컴파일 타임에 보다 강한 타입 체크가 가능하다 (`any`를 사용하는 것보다 type narrow됨)
		- 자바에서는 제네릭을 사용하는 경우 명시적인 타입 캐스팅을 하지 않아도 되는 경우가 있음
		- ts도 그렇지만 컴파일러가 이미 타입에 대한 힌트를 들고 있기 떄문에 가능
	- 서로 다른 타입에 대해 동일한 알고리즘을 다시 작성할 필요가 없다 —> 생산성 증대