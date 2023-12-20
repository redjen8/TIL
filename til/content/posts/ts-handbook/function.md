---
title: "타입스크립트 핸드북 3장 - 함수"
date: 2023-12-20T22:19:51+09:00
draft: false
author: redjen
---

https://typescript-kr.github.io/pages/functions.html

### Function

JS와 마찬가지로 named function과 anonymous function을 제공
- 함수는 외부 변수를 참조 (변수 캡쳐) 가능

### Typing the Function

- 함수의 타입은 매개변수의 타입과 반환 타입이 존재
   - 캡쳐된 변수는 타입에 반영 불가
- 매개변수 타입과 반환 타입 사이에 `=>`를 사용해 반환 타입을 명시가능
   - 함수가 값을 반환하지 않을 경우 `void` 사용

### Optional and Default Parameter

> 모든 매개변수는 함수에 필요하다

하지만 `null` 값이나 `undefined`를 줄 수 없는 것은 아니다.
- 함수의 호출 시점에 컴파일러는 각 매개변수에 값이 채워졌는지 검사
- 함수에 주어진 인자의 수 === 받아야 할 매개변수의 수 (이 점이 JS와 다르다)


**선택적으로 매개변수 전달을 원한다면 `?`를 붙여서 전달 가능** (선택적 매개변수)

이 친구는 반드시 순서 상 필수 매개변수 뒤에 와야 한다

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // 지금은 바르게 동작
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```

**매개변수에 기본 값을 채워서 전달 가능** (기본 - 초기화 매개변수)

이 친구는 필수 매개변수 뒤에 오는 것이 강요되지는 않는다. 
- 하지만 이 경우 사용자가 명시적으로 `undefined`를 전달해야 기본 값을 사용할 수 있다

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 올바르게 동작, "Bob Smith" 반환
let result2 = buildName("Bob", undefined);       // 여전히 동작, 역시 "Bob Smith" 반환
let result3 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result4 = buildName("Bob", "Adams");         // 정확함
```

### Rest Parameters

- 다수의 매개변수를 그룹 지어 작업하는 경우
- 얼마나 많은 매개변수를 최종적으로 취하는지 모르는 경우
- JS에서는 `arguments` 변수를 사용할 수 있지만
- TS에서는 이들을 하나의 변수로 모을 수 있다

```ts
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

// employeeName 은 "Joseph Samuel Lucas MacKinzie" 가 될것입니다.
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

### this와 arrow function

```js
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

- JS에서, 최상위 컨텍스트에서의 non method 문법의 호출은 `this`를 `window`로 설정한다.
- 때문에 `createCardPicker` 내부에서 `this`의 `suits` 참조 호출은 오류를 발생시킨다
- 아래와 같은 사용은 괜찮다
   - 화살표 함수는 함수가 호출된 곳이 아닌 함수가 생성된 쪽읜 `this`를 캡쳐하기 때문

```js
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: 아랫줄은 화살표 함수로써, 'this'를 이곳에서 캡처할 수 있도록 합니다
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```
### this parameter

> `this` 매개변수는 매개변수 목록에서 가장 먼저 나오는 가짜 매개변수이다.

```ts
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: 아래 함수는 이제 callee가 반드시 Deck 타입이어야 함을 명시적으로 지정합니다.
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

`createCardPicker` 함수는 반드시 `Deck` 타입에 대해서 호출된다는 것을 명시적으로 전달할 수 있다

### this parameters in callback

콜백 함수를 라이브러리에 전달할 때는 몇 가지 어려움이 존재한다.
- 라이브러리는 콜백을 일반 함수처럼 호출한다.
- `this` = `undefined`가 된다.
- 이를 방지하기 위해 `this` 매개변수를 사용할 수 있다.

```ts
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

`this`가 `void` 타입 매개변수로 전달되면 `this` 타입을 요구하지 않는 함수라고 알려주는 것

### Overloading

- JS는 기본적으로 오버로딩을 지원하지 않음
- 하지만 JS에서 마치 오버로딩 처럼 함수 내에서 `typeof`를 사용해 구현을 할 수는 있다
- TS의 타입 시스템에서는 오버로드 목록을 통해 이를 구현할 수 있다.

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // 인자가 배열 또는 객체인지 확인
    // 만약 그렇다면, deck이 주어지고 card를 선택합니다.
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // 그렇지 않다면 그냥 card를 선택합니다.
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

- 컴파일러는 오버로드 목록에서 첫 번째 오버로드를 진행하면서 함수 호출을 시도
- 일치하는 경우 해당 오버로드를 사용하여 작업 수행
- 때문에 가장 구체적인 것부터 오버로드 목록을 구성하는 것이 효율적