---
title: "Java Unused Import"
date: 2023-03-26T22:47:20+09:00
draft: false
author: redjen
---

IDE에서는 사용하지 않은 import 문에 대해서 indication을 해준다.
SonarQube와 같은 코드 퀄리티 도구를 사용한다면 code smell 항목 등으로 아예 이를 제거하라고 알려주기도 한다.

그렇다면, unused import문은 왜 안 좋을까? 단순히 존재만 하는 것도 내 어플리케이션에 영향을 줄까?

https://stackoverflow.com/questions/18153690/does-an-unused-import-declaration-eat-memory-in-java

## 이유

사실 unused import 문은 어떠한 추가 메모리도 사용하지 않는다. 

import 문들은 단순히 컴파일러들이 컴파일 타임에 클래스 이름을 resolve하기 위한 용도로 사용된다.

### 동일한 이름의 클래스를 동시에 사용하려면

동일한 이름의 import문을 동시에 사용하려면 어떻게 해야 할까?

https://stackoverflow.com/questions/2079823/importing-two-classes-with-same-name-how-to-handle

```java
java.util.Date javaDate = new java.util.Date()
my.own.Date myDate = new my.own.Date();
```

위와 같이 선언하면 된다. 애초에 컴파일러가 바이트 코드로 변환하기 전에 위와 같은 동작을 수행하기 떄문에 전혀 문제가 될 것이 없다.

- 컴파일러는 각각의 클래스 이름들을 fully qualified name으로 변환한다.
- 그 후에 import 선언들을 제거한다.
- 때문에 import 선언들은 애초에 바이트 코드 내에 존재하지 않는다.
- 애초에 바이트 코드로 존재하지 않으므로 성능에 영향을 미칠 일은 전혀 존재하지 않는다.

그럼 왜 IDE와 코드 퀄리티 솔루션에서는 성능에 영향을 미치지 않는 unused import 문을 제거하라고 할까?

## 진짜 이유

사실 어플리케이션의 실행에는 전혀 지장이 없기 때문에 위에서 언급한 도구들은 unused import에 대해 에러 수준의 경고를 발생시키지 않는다.

단순히 코드 퀄리티 향상을 위해서 제거하는 것이 맞다고 생각이 든다.

1. 어떤 목적을 위해서 import한 내용을 사용하다가, 더 이상 필요 없어져 코드 본문에서는 제거한 후에
2. 나중에 동일 이름의 라이브러리를 사용해야 하는 경우에는 실제로 문제가 될 수 있다.

또 지금이야 IDE의 도움을 받지만, IDE 레벨에서 사용하지 않는 import 문을 따로 표시해주지 않는다면 무슨 의도로 이런 import 문을 썼는지 해당 클래스 파일을 전부 훑어봐야 하는 불상사가 생길 수도 있다.

얌전히 IDE와 코드 퀄리티 솔루션의 조언을 들어야겠다.