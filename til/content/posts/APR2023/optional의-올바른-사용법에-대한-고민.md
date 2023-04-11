---
title: "Optional의 올바른 사용법에 대한 고민"
date: 2023-04-11T19:35:59+09:00
draft: false
author: redjen
---

자바를 맨 처음 배우고 프로그래밍을 하면서 가장 쉽게 접할 수 있는 에러를 꼽자면 바로 `NullPointerException`일 것이다.

자바의 객체들은 Heap 메모리에 존재하고, 그 ref 값은 Stack 메모리에 존재한다.

- 보통 객체를 접근할 때에는 ref 값을 통해 접근하고
- 이렇게 접근한 객체에 특정 operation을 실행하려는 순간
- ref 값에 해당하는 메모리에 실 객체가 없는 경우 발생한다.

자바8부터 등장한 `Optional<T>`의 등장은 null로 골치를 앓는 여러 사용자들을 실제로 구원해주었을까?

라고 물어본다면, 글쎄... 100% 그렇지는 않다고 대답할 것 같다. 

그 이유와, 이 논쟁에 대한 자바 사용자 커뮤니티의 반응을 살펴보았다.

## 발단

사실 `Optional`과는 크게 관련이 없는 한 Medium 글을 읽으면서부터 이 생각이 들었다.
https://medium.com/@alexeynovikov_89393/stop-using-exceptions-in-java-456f7db46ea4

내용을 요약하자면
- 자바의 `Exception`은 다른 언어 (scala, go 등) 처럼 특정 동작에 대한 결과와 에러를 동시에 나타내는 것이 불가능하기 때문에 불편하다.
- unusual 한 상황과, exceptional 한 상황을 구분할 필요가 있다.

수 년동안 자바를 써왔기 때문에 다른 언어에서 이런 기능이 있는지는 잘 몰랐기 때문에 신선한 충격이었는데, 글 작성자가 예시로 제시한 `Result<T>` 클래스와 함께 그의 주장에 대한 댓글 토론창이 열려서 재미있게 읽었다.

글쓴이가 작성한 `Result<T>` 클래스에서는 `Optional<T>` 값을 반환한다..

여기에 대한 지적 포인트가 있었기 때문에 조금 더 찾아보게 되었다.

## `Optional`을 언제 사용해야 할까?

[오라클의 Optional 아티클](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)에 따르면 `Optional`은 모든 `null` reference를 제거하기 위해 생겨난 것이 아니다.

`Optional`은 더 이해하기 쉬운 API를 설계하는 것을 돕기 위해 생겼다. 메서드의 시그니처만 읽어도 메서드가 어떤 값을 반환할지에 대한 명시적인 힌트를 주는 것이다..
- 만약 `Optional` 로 값을 반환하는 메서드라면 null이 올 수 있으니 래핑을 메서드 호출하는 쪽에서 사용해야 하고
- `Optional`로 값을 반환하지 않는 메서드라면 null이 오지 않는다는 것을 기본 전제로 해야 한다.

또 하나의 커멘트로는 `Optional`은 다음 컨텍스트에서 사용하려 만들어진 것이 아니라는 점도 있었다. 아무 이점도 없기 때문이다.
- 직렬화가 불가능한 도메인 모델 레이어 
- DTO들
- 메서드의 입력 파라미터
- 생성자의 파라미터

또 다른 관점으로는, `Optional`이 자바의 함수형 프로그래밍을 돕기 위해 나타났다는 의견도 있었다.
```java
return Arrays.asList(enclosingInfo.getEnclosingClass().getDeclaredMethods())
    .stream()
    .filter(m -> Objects.equals(m.getName(), enclosingInfo.getName())
    .filter(m ->  Arrays.equals(m.getParameterTypes(), parameterClasses))
    .filter(m -> Objects.equals(m.getReturnType(), returnType))
    .findFirst()
    .getOrThrow(() -> new InternalError(...));
```
가 아래 예시보다 명시적이고 깔끔하다.

```java
Method matching =
    Arrays.asList(enclosingInfo.getEnclosingClass().getDeclaredMethods())
    .stream()
    .filter(m -> Objects.equals(m.getName(), enclosingInfo.getName())
    .filter(m ->  Arrays.equals(m.getParameterTypes(), parameterClasses))
    .filter(m -> Objects.equals(m.getReturnType(), returnType))
    .getFirst();
if (matching == null)
  throw new InternalError("Enclosing method not found");
return matching;
```

또한, `List`와 같은 컬렉션 타입에 대해 `Optional`을 쓰는 것은 대표적인 안티패턴으로 알려져 있다.
- 대신 `List.empty`와 같은 빈 컬렉션을 리턴하는 것이 훨씬 낫다.

습관적으로 Accessor에 `Optional`을 사용하는 것은 배보다 배꼽이 훨씬 큰 상황이라고 생각한다..
`Optional`을 모든 곳에 사용하지 않고 정말 딱 필요한 곳에서만 사용해야 한다.

결국 논점을 요약하자면 아래와 같이 사용될 수 있을 것 같다.

## 결론 

- null이 값으로 리턴될 수도 있는 곳에는 `Optional`을 사용해라.
	- 대표적으로는 DB에서 쿼리 결과 값을 전달하는 데이터 레포지토리 레이어가 있다.
	- 나는 특별한 경우가 아니라면 데이터 레포지토리 레이어에서 이뤄지는 모든 조회 메서드에서는 모두 `Optional`을 리턴해야 한다고 본다.
- 그 외의 곳에서는 모두 리턴 타입에 null이 들어가지 않도록 보장해라.
	- `Optional`을 사용하지 않는 곳에서는 `null`이 들어가지 않음을 보장해야 한다.
	- 만약 `Optional`을 사용하지 않는 곳에서 `null`이 리턴된다면 설계가 잘못된 것이니, 수정해야 한다.


이런 내용을 모르고 썼기 때문에, `Optional`을 굳이 써야 할까? 라는 의문이 들었던 것 같다.
- 프로그래밍 언어 레벨에서는 null이 존재할 수 있어도
- 개발자의 시선에서는 null이 명시적으로 등장하지 않는 것이 `Optional`의 존재 의의라고 생각한다.

