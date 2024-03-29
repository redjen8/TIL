---
title: "Javascript Math Min Max의 동작"
date: 2023-05-14T20:54:03+09:00
draft: false
author: redjen
---

자바스크립트의 내부 동작은 언뜻 보았을 때 잘 이해가 되지 않는 부분들이 많이 있는 것 같다.

오늘은 그 중 하나라고 생각되는, `Math.min()`과 `Math.max()`의 리턴값이 왜 각각 `Infinity`, `-Infinity`인지 알아봤다.

https://stackoverflow.com/questions/8848779/why-does-math-min-return-positive-infinity-while-math-max-returns-negative

## 최댓값과 최솟값을 구하는 방법

배열 중에서 가장 큰 수를 어떻게 골라낼까?

떠올릴 수 있는 가장 간단한 방법은 `O(n)`에 배열 전체를 순회하면서 최댓값을 갱신하는 것이다.
- 이 때 다음 차례에 오는 다음 수가 최댓값보다 크다면, 그 수는 현재 임시 최댓값보다 크기 때문에 임시 최댓값을 갱신하게 된다.
- 배열을 이렇게 전부 순회하게 된 이후에 존재하는 임시 최댓값은 그 배열의 모든 수보다 같거나 크다는 것이 보장되기 때문에
- 해당 수가 배열에서의 최댓값임을 보증할 수 있다.

비슷하게 배열 내에서의 최솟값도 구할 수 있다.

## `Math.min` 과 `Math.max`는 어떻게 동작할까?

`Math.min()`과 `Math.max()`는 파라미터에 대해 각각 최솟값, 최댓값을 구해주는 함수이다.

때문에

`Math.min(3)`은 3을 리턴해야 한다. 자연스럽게 3은 자바스크립트가 비교해야 할 가장 큰 수와 비교해서 작아야 한다. 

최솟값을 판정하기 위해서는 임시 최솟값을 설정해야 하는데, 이 값이 충분히 크지 않다면 실제로 이 수가 입력한 파리미터 중 최솟값을 리턴할지 아닐지 알 수 없기 때문이다.

문제는 파라미터를 입력하기 않았을 때 애매하다는 점이다!

누가 `Math.min()` 자체만 보고 이 수가 입력 받지 않는 파라미터에 대해 최솟값 연산을 수행하려 한다고 생각할까?
static한 메서드이기 떄문에, 당연히 언어 내에서의 가장 작은 수를 리턴할 것이라고 기대하게 된다. (내가 그랬다)

빈 집합이 파라미터로 전달될 때, 집계 함수가 수행해야 할 작업을 올바르게 설정하는 것은 어렵다.

때로는 직관적으로 이 작업들이 명백한 경우도 있는데, 공집합의 합계는 모두가 0이라고 대답할 것이다.
하지만 직관적으로 빈 집합에 대한 작업이 모호한 경우가 있다.

공집합에서의 원소 product 곱은 무엇일까? 답은 1이다. 수학적으로 정의되어 있기 때문에 1이라는 것을 이미 아는 사람들은 쉽게 답하겠지만 대부분의 사람들이 이를 직관적으로 알기는 쉽지 않다.

## 결론

자바스크립트의 집계 함수 중 하나인 `min`, `max`는 연산을 연관지어서 수행하는 방식을 허용하기 때문에 이런 결과를 낳았다.
때문에 [ECMA-262](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/) 표준에는 아래와 같은 내용이 포함되어 있다.

> max
>
> 0 또는 하나 이상의 인자가 주어지면, 각각의 인자에 대해 `ToNumber`를 호출하고 결과 값 중에서 가장 큰 값을 리턴한다.
> 만약 아무 인자도 주어지지 않는다면 결과는 -`Infinity`이다.
> 어떤 값이라도 `NaN`이라면 결과는 `NaN`이다.
> 최댓값을 판정하기 위해 값의 비교를 수행하는 것은 본 표준의 11.8.5와 같이 수행한다. (+0은 -0보다 크다고 정의하기에 이는 예외이다)

min은, 정의만 빼놓고는 전부 동일하다.
> 0 또는 하나 이상의 인자가 주어지면, 각각의 인자에 대해 `ToNumber`를 호출하고 결과 값 중에서 가장 작은 값을 리턴한다.

