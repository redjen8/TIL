---
title: "Java SecureRandom"
date: 2023-04-09T21:18:31+09:00
draft: false
author: redjen
---

자바 프로젝트를 하다 보면 랜덤한 데이터를 즉석에서 만들어 뿌려줘야 할 때가 있다.

랜덤한 정수를 만들어내는 예제에서 많이 사용하는 `SecureRandom`은 내부적으로 어떻게 랜덤한 정수를 만들까?

https://www.baeldung.com/java-secure-random

## `java.util.Random` 과 차이점

`java.util.Random`을 사용하는 표준 JDK 구현체는 Linear Congruential Generator 알고리즘을 사용하여 난수를 생성한다.

이 알고리즘의 문제는 암호학적으로 강하지 않다는 것이다. 즉 생성된 값은 예측 가능하기 때문에 공격자의 입장에서는 시스템을 공격할 수 있는 요소가 될 수 있었다.

이 이슈를 해결하기 위해서, 보안과 관련된 모든 난수는 `java.security.SecureRandom`을 사용해야 한다.

이 방법은 cryptographically strong pseudo-random number generator (CSPRNG)를 사용하여 암호학적으로 강한 난수를 생성한다.

- 씨드를 매뉴얼하게 지정할 수 있다.
  - 기본 씨드는 `/dev/urandom`에서 가져온다.
  - 그리고 이 방식은 사용하는 시스템에 따라 이슈가 있을 수도 있다. (후에 언급)
  - 동일한 씨드를 가진 서로 다른 인스턴스는 매번 호출하는 메서드가 리턴하는 시퀀스가 동일하다.
    - 즉 동일 씨드 == 동일한 시퀀스 생성이다.
- 난수 생성 알고리즘을 따로 선택할 수도 있다.
  - 기본적으로는 `SHA1PRNG` 알고리즘을 사용한다.
  - 다른 알고리즘을 사용하려면 `getInstance(algorithm: string)` 정적 메서드를 사용해서 해당 알고리즘의 인스턴스를 사용한다.


## 은탄환은 없다

보안성이 보장될 정도로 강한 난수를 생성하면 마냥 좋지는 않다. 

https://stackoverflow.com/questions/137212/how-to-deal-with-a-slow-securerandom-generator

`SecureRandom`은 매우 느릴 수 있다. 그리고 CPU-bounded 되는 작업이다.

그리고 CPU bounded 되는 작업은 비동기 논블락킹 방식의 서버인 node.js나 Spring Webflux에서는 주의를 기울여서 사용해야 한다.

난수 생성 하는 로직이 포함되어 있을 때에는 실 사용 시에 성능 이슈가 없는지 체크해 봐야겠다.