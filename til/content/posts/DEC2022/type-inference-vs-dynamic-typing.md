---
title: "Type Inference vs Dynamic Typing"
date: 2022-12-26T19:25:51+09:00
draft: false
author: redjen
---

# Type Inference vs Dynamic Typing

Java 10에서부터는 var 키워드를 통한 타입 추론 (type inference)가 가능하다.
var 키워드를 사용하게 되면 보다 간결한 코드 작성이 가능하다고 한다. 그런데 타입 추론이 정확히 뭘까?

## 타입 추론 (Type Inference)

### 정적 타입과 동적 타입 언어

정적 타입 언어에서는 컴파일 타임에 데이터의 타입이 결정된다.
컴파일 이후에는 천지가 개변해도 다른 타입의 데이터가 할당되는 순간 컴파일 에러를 뱉는 IDE를 볼 수 있다.
C, C++, Java와 같은 '전통적인 프로그래밍 언어들'이 정적 타입 언어이다.

파이썬, 자바스크립트, PHP와 같은 언어는 동적 타입 (dynamic type)을 지원한다.
파이썬의 경우 컴파일이 아닌 인터프리터를 통해 바이트 코드로 변환되어 인터프리터에서 실행되는 언어이니까.
동적 타입 언어는 새로운 타입의 데이터가 객체에 할당되는 것을 허용한다.
따라서 잘만 사용한다면 굉장히 강력한 무기가 될 수 있다.

### 타입 추론이란

https://en.wikipedia.org/wiki/Type_inference

헷갈릴 수도 있지만, 타입 추론은 정적 타입 언어에서의 영역이라고 생각한다. 
컴파일러가 데이터의 추론된 타입을 컴파일 시점에 결정해서 어쨌든 '타입을 정한다'는 개념이기 때문이다.
컴파일러는 표현식 (expression)의 정보를 빠르게 수집해서 객체가 어떤 타입이 될 수 있는지에 대한 단서를 얻는다.
Typescript와 같은 언어들은 컴파일러가 표현식에서 충분히 정보를 얻을 수 없는 단서를 제공하지 못하는 경우도 있다.
이런 경우 타입 어노테이션과 같은 추가 정보를 통해 객체가 어느 데이터 타입으로 결정될 수 있는지에 대한 단서를 제공해줘야 한다.

```
let age : number = 25
```

여담이지만, 파이썬의 아버지 귀도 반 로썸은 동적 타입 언어인 Javascript를 보완하기 위한 정적 타입 언어인 Typescript를 굉장하다고 평가했다. 모던 파이썬에도 점점 타입을 강제할 수 있는 여러 옵션들이 생겨나고 있으며 앞으로 파이썬이 나아가는 방향에는 타입스크립트가 해냈던 몇 가지 일들을 참고할 수도 있다고 밝혔다. ([출처](https://developers.slashdot.org/story/21/05/22/0348235/what-python-creator-guido-van-rossum-thinks-of-rust-go-julia-and-typescript))

## 자바에서의 타입 추론

자바는 때려 죽어도 strongly statically type를 가지는 언어로 태어났다.
자바 개발자들 또한 이 관점에서 굉장히 보수적이기 때문에 아마 자바가 망하는 날까지 최신 버전의 자바에서 dynamic type을 지원할 일은 없을 것이다.

때문에 자바스크립트의 'var'과 자바에서의 'var' 키워드는 전혀 다른 역할을 수행한다.
자바에서의 'var' 키워드는 자바스크립트의 'var' 처럼 동적 타입 변수를 가지는 것이 아니다. 
자바에서의 'var' 키워드는 로컬 변수에서 효율적으로 중복되는 코드를 없애는 데 사용될 수 있다.  

그런데, 왜 '슈퍼 울트라 강 정적 타입 언어'인 자바에서는 타입 추론이라는 MZ스러운 개념을 도입한걸까?
몇 가지 이점이 있다. 

https://openjdk.org/projects/amber/guides/lvti-faq

### 타입 추론의 장점

자바에서는 유독 장황한 네이밍을 가지게 되는 객체가 많다. 
물론 Human readable한 네이밍을 가지는 것이 그렇지 않은 것보다 백 배 낫다.
객체의 이름만 보고도 이 객체가 어떤 responsibility를 가지는지, 어떤 역할을 수행하는지 추론할 수 있어야 한다는 생각에는 전적으로 동의한다.
하지만 실제로 코딩할 때에는 이 장황한 객체 이름이 불편할 때가 있다.
```
UserInvestmentHistoryFactory<UserInvestmentHistory> factory = new UserInvestmentHistoryFactory<>();
```

UserInvestmentHistory라는 단어가 3번이나 반복되는 끔찍한 코드를 쓸 수 밖에 없다.
현대 라이브러리에서는 제네릭을 더욱 적극적으로 지원하는 만큼, 많은 자바 사용자들은 불편함을 느낀다.
Collection 객체에 제네릭을 사용하게 되면 상황은 더 악화된다.
```
for (Map.Entry<Integer, Map<Integer, UserInvestmentInfo<User, T>>> eachInvestmentInfoEntry : investmentTable.entrySet()) {
...
}
```
과장됐긴 하지만 이런 코드를 보면 잠시 뇌가 멈추게 된다. 저 한 줄만 봤을 때 어떤 의미인지 단번에 알 수 없다. 
코드를 작성하는 입장에서는 이미 아는 정보를 여러 번 기입해야 하는 것이 짜증나고, 
코드를 읽는 입장에서는 이미 어떤 맥락에서 사용되는지 알지만 어쩔 수 없이 들어간 단어들 때문에 코드의 정확한 의도를 읽기 어렵다는 단점이 있다.

자바의 var 키워드를 사용한 타입 추론을 사용하면 이런 단점을 줄일 수 있다.
```
var factory = new UserInvestmentFactory<UserInvestmentHistory>();
```
훨씬 읽기 편하지 않은가? 꼰대 정적 언어인 자바를 위해 컴파일러는 타입 추론을 통해 factory 객체가 UserInvestmentFactory\<UserInvestmentHistory\> 객체임을 '프로그래머 대신' 알려준다. 

개발을 하다 보면 코드를 쓰는 일보다 읽는 일이 훨씬 많다.
타입 추론의 주 목적은 코드를 읽는데 들어가는 오버헤드 / 비용을 줄이고 더 깔끔한 코드를 만들기 위한 것이라는 생각이 든다.

### 타입 추론의 단점

아이러니컬하게도 타입 추론은 적절히 사용하지 않으면 오히려 코드 가독성에 악영향을 줄 수도 있다.
```
var item = SomeObjectInitializationProcess();
```

나는 명시적으로 타입을 추론할 수 있는 쉬운 경우에만 타입 추론을 사용하는 편이다. 위와 같은 코드는 item 변수가 어떤 타입인지 프로그래머에게 힌트를 주지 않는다. 본래의 취지를 잃고 비틀거리는 주정뱅이 코드이며 도움이 되지 않는 misuse 케이스이다. 

또한, 운영 체제 커널과 같이 로우 레벨에서 동작하는 시스템을 위한 언어에서는 타입 추론이 의미가 없어진다.
로우 레벨 시스템에서는 주로 시스템 메모리 / 파일에 있는 타입 없는 데이터를 프로그램의 메모리 속 타입을 가진 데이터로 만들고, 이후 불러온 데이터는 안전하게 사용할 수 있다는 것을 가정하고 쓰여진다.
때문에 시스템 프로그래밍에서는 쓸모 없는 개념이기도 하다. 타입 추론과 완전히 반대되는 작업을 이미 코드에서 하고 있기 때문이다. 


### 번외 : 타입스크립트의 type system이 튜링 완전한가?에 대해

타입스크립트 타입 추론 시스템이 튜링 완전한가?에 대한 논의도 있어서 재미있게 읽었다.
https://github.com/microsoft/TypeScript/issues/14833

타입스크립트의 타입 시스템은 튜링 완전하다.
타입스크립트가 turing complete인지 판별하려면 아래 조건 중 하나라도 충족시키면 되는데, 
1. 임의의 튜링 머신이 타입스크립트로 translate 될 수 있는가?
2. 타입스크립트가 universal turing machine을 구현할 수 있는가?
3. 타입스크립트가 다른 잘 알려진 turing complete 시스템을 구현할 수 있는가?
타입스크립트는 이 중 3번을 만족할 수 있기 때문이다.
https://itnext.io/typescript-and-turing-completeness-ba8ded8f3de3

