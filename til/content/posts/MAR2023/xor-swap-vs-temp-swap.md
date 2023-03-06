---
title: "Xor Swap vs Temp Swap"
date: 2023-03-06T19:52:30+09:00
draft: false
author: redjen
---

엔지니어링 과정에서 극한의 효율을 추구해야 하다보면 항상 로우 레벨이나 그 원리에까지 범위가 닿게 되는 것 같다.

각종 정렬 알고리즘에서는 원소의 위치를 서로 뒤 바꾸는 swap 연산을 사용할 일이 많다.

XOR swap은 무엇이고, 왜 효율적이지 못한지 정리해봤다.

## XOR swap

https://ko.wikipedia.org/wiki/XOR_%EA%B5%90%EC%B2%B4_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98

> XOR 스왑 알고리즘은 임시 변수를 두지 않고, 두 변수를 XOR 비트 연산을 사용하여 교체하는 알고리즘이다.

변수 x, y가 있고 두 값을 바꿔야 한다. XOR swap은 pseudo code로는 아래와 같이 나타낼 수 있다.
- x <- x XOR y
- y <- x XOR y
- x <- x XOR y

### 증명

XOR 이항 연산은 다음의 특징을 가진다.

1. 교환법칙이 성립
2. 결합법칙이 성립
3. 항등원이 존재한다.
4. 각 원소에 대해 유일한 역원이 존재한다.
	1. 각 원소의 역원은 자기 자신이다.

과정 | 수행된 명령 | r1의 값 | r2의 값 | 사용된 성질
--- | --- | --- | --- | ---
1 | (초기 상태) | A | B | -- 
2 | r1 <- r1 XOR r2 | A^B | B | --
3 | r2 <- r1 XOR r2 | A^B = B^A | (A^B)^B = A^(B^B) = A^0 = A | 1, 2, 4, 3
4 | r1 <- r1 XOR r2 | (B^A)^A = B^(A^A) = B^0 = B | A | 2, 3, 4

### C에서의 구현

```c
void swap(int *x, int *y)
{
    if (x != y) {
        *x ^= *y;
        *y ^= *x;
        *x ^= *y;
    }
}
```


## 왜 XOR swap 보다 temp swap이 효율적인가?

https://en.wikipedia.org/wiki/XOR_swap_algorithm#Reasons_for_avoidance_in_practice

1. 현대의 x86 CPU에서는 temp swap에서 사용하는 레지스터 간 값의 이동이 거의 zero-latency에 수행될만큼 성능이 개선되었다. (MOV-elimination)
	1. 기계어 레벨에서 XOR 연산 세 번 보다 move 연산 세 번이 훨씬 빠르기 때문이다.
2. 현대의 CPU는 연산 파이프라인에 연산을 태워서 병렬적으로 수행하려는 시도를 하는데, XOR swap을 사용하면 지금 실행되어야 할 연산이 이전 결과에 종속적이기 때문에 작업을 병렬적으로 실행할 수 없다. instruction 레벨에서의 parallelism 부재.

효율의 문제를 떠나 정확성에 대한 문제도 존재한다.
- 같은 메모리 공간을 가르키는 두 변수가 있다면, XOR swap을 실행했을 때 값을 잃어버린다.
- 때문에 고수준 언어에서의 aliasing이 이루어져야 한다.
- 구현에 명시적으로 `if`를 사용해서 branch 연산을 넣을 경우 오히려 더 성능을 떨어뜨린다.

## Add Swap

XOR 보다 더 제한적인 상황에서 사용할 수 있는 swap 방법도 존재한다. Add Swap을 c로 구현하면 아래와 같다.

```c
void AddSwap( unsigned int* x, unsigned int* y )
{
  if (x != y)
  {
    *x = *x + *y;
    *y = *x - *y;
    *x = *x - *y;
  }
}
```

단순히 값을 더하고, 빼서 두 변수의 값을 바꿀 수 있는 방법이다. 이 swap 방법은 아래와 같은 제한 사항을 가진다.
1. 정수 오버플로우가 일어나지 않는 자료형에서만 사용해야 한다.
	1. 다만 C에서는 정수 오버플로우의 발생 여부와 상관 없이 항상 동작한다 : unsigned int의 덧셈과 뺄셈은 모듈러 산술의 규칙을 따르기 때문이다.
	2. unsigned가 아닌 signed 자료형의 경우 오버플로우가 일어난다.

## 정렬에서 사용하는 swap에 따라 성능에 유의미한 차이가 있나?

> 그렇지 않다.

https://stackoverflow.com/questions/36906/what-is-the-fastest-way-to-swap-values-in-c

정렬에서는 swap 자체의 성능 보다는 swap 호출이 일어나는 횟수를 줄이는 것이 유의미한 차이를 만들기 때문이다. 

- 정렬 알고리즘은 일반적으로 메모리에 매우 불규칙한 방식으로 접근하기 때문에 메모리 접근을 더 악화시킨다.
- L2, RAM, 하드 디스크에서 데이터를 가져오는 비효율적인 오버헤드를 일으킨다.
- 따라서 swap 자체를 최적화하는 것은 의미가 없다.
	- swap을 적게 호출하면 적게 호출하기 때문에 최적화가 의미가 없어진다.
	- swap을 많이 호출하면 cache miss의 빈도가 잦아지기 때문에 비효율적이게 된다.

정렬 알고리즘에서 많이 사용하는 빅오 표기법도, `O(N)` 이 항상 `O(logN)` 보다 빠른 것은 아니다.
N 크기가 유의미하게 커지지 않는다면 `O(N)`은 `O(logN)`보다 빠를 수 있다.

또한 이미 정렬되어 있는 배열의 경우 퀵소트가 다른 정렬보다 훨씬 비효율적이다.

때문에 정렬 알고리즘의 성능은 정렬 알고리즘도 중요하지만, 데이터의 성질과 크기도 그만큼 중요하다. 