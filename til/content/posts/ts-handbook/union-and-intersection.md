---
title: "타입스크립트 핸드북 5장 - 유니언과 교차 타입"
date: 2024-01-10T21:58:47+09:00
draft: false
author: redjen
---
https://typescript-kr.github.io/pages/unions-and-intersections.html

### Union Types

> `any`를 사용하기 보단 예상되는 타입의 개수를 줄인다

```ts
function padLeft(value: string, padding: string | number) {
  // ...
}

let indentedString = padLeft("Hello world", true);
```

- `|` 로 타입을 구분
- `number | string | boolean`은 값 타입이 `number`, `string`, `boolean` 중 하나가 될 수 있음을 의미
### Unions with Common Fields

> 유니언 타입을 사용할 때에는 유니언으로 연결된 모든 타입에 공통으로 존재하는 값에만 접근 가능

```ts
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}
```

위 예제에서는 `Bird | Fish` 유니언 타입이 있을 때 공통으로 존재하는 멥버인 `layEggs()`에만 접근 가능

### Discriminating Unions

> 컴파일러가 type narrowing에 도움이 되도록 리터럴 타입을 가지는 단일 필드를 사용하자

```ts
type NetworkLoadingState = {
  state: "loading";
};

type NetworkFailedState = {
  state: "failed";
  code: number;
};

type NetworkSuccessState = {
  state: "success";
  response: {
    title: string;
    duration: number;
    summary: string;
  };
};

// 위 타입들 중 단 하나를 대표하는 타입을 만들었지만,
// 그것이 무엇에 해당하는지 아직 확실하지 않습니다.
type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;
```

- 위 예시에서 각 타입에 따라 그외 다른 필드도 가지고 있음
- 하지만 `state` 필드는 모든 타입에 존재 
	- 때문에 필드 유무 검사 없이 컴파일러는 `NetworkState` 유니온 타입에 `state` 필드가 존재함을 알 수 있음
	- 이렇게 사용했을 때 런타임에 switch 문 등을 사용해서 유니언 타입의 타입을 더 좁혀 나갈 수 있다

### Intersection Types

> 교차 타입은 여러 타입을 하나의 단일 타입으로 결합한 타입\

- 교차 타입의 객체는 교차된 모든 타입의 모든 멤버를 가지게 된다.

```ts
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

interface ArtistsData {
  artists: { name: string }[];
}

// 이 인터페이스들은
// 하나의 에러 핸들링과 자체 데이터로 구성됩니다.

type ArtworksResponse = ArtworksData & ErrorHandling;
type ArtistsResponse = ArtistsData & ErrorHandling;

const handleArtistsResponse = (response: ArtistsResponse) => {
  if (response.error) {
    console.error(response.error.message);
    return;
  }

  console.log(response.artists);
};
```

- 위 예시에서는 각 인터페이스를 교차타입으로 결합
- 교차 타입을 사용함으로써 추가적인 boilerplate 코드 없이 해당 멤버를 참조 가능

### Interface Segregation Principle

- SOLID 원칙 의 I (ISP)
- 인터페이스는 **가능한 적은 양의 메서드를 가지고 있어야 한다**
- 인터페이스를 구현하는 concrete 클래스는 반드시 인터페이스의 모든 메서드를 구현해야 하기 때문

### Intersection Types in Java

https://dev.to/rnowif/intersection-types-in-java-1jk8

자바에서는 제네릭과 결합해서 다음과 같이 사용 가능

```java
<T extends FileReader & FileWriter> void readAndWrite(T file) {
  file.readLines();
  file.write("Hello");   
}
```

- 비슷하게 여러 인터페이스를 교차한 인터페이스가 필요할 때 사용