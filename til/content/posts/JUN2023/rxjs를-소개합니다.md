---
title: "Rxjs를 소개합니다"
date: 2023-06-12T22:07:17+09:00
draft: false
author: redjen
---

## Reactive Streams와 ReactiveX에 대해

[ReactiveX의 공식문서](https://reactivex.io/)에서는 `Observable`한 시퀀스를 사용해 비동기적인, 이벤트 기반의 프로그램들을 다루기 위한 라이브러리로써 ReactiveX로 소개한다.

ReactiveX는 옵저버 패턴을 적용해서
- 데이터나 이벤트의 시퀀스들을 지원하거나
- 시퀀스들을 선언적으로 다룰 수 있는 연산자들을 지원하면서도
- 로우 레벨에서 이뤄지는 쓰레드 제어, 동기화, thread safety, 동시성을 지원하는 자료구조, 논블락킹 IO에 대해 신경쓰지 않게 해준다. (개인적으로는 이게 가장 큰 장점이라고 생각한다)

ReactiveX의 `Observable`은 여러 아이템의 비동기적인 시퀀스를 다룰 수 있는 이상적인 방법을 제공한다.

구분 | 단일 아이템 | 복수 아이템
--- | --- | ---
동기 | `T getData()` | `Iterable<T> getData()`
비동기 | `Future<T> getData()` | `Observable<T> getData()`

ReactiveX 라이브러리들은 가끔 함수형 반응형 프로그래밍으로써 불리지만, 이건 잘못된 명명이라고 소개한다.

> ReactiveX may be functional, and it may be reactive, but “functional reactive programming” is a different animal. One main point of difference is that functional reactive programming operates on values that change continuously over time, while ReactiveX operates on discrete values that are emitted over time.

> ReactiveX는 함수형일수도 있고, 반응형일수도 있지만, '함수형 반응형 프로그래밍'은 전혀 다른 종류의 것이다. 한 가지 큰 차이점은 함수형 반응형 프로그래밍은 시간에 따라 지속적으로 변화하는 값을 다루지만, ReactiveX는 시간에 따라 방출되는 이산 값에 대해 작동한다는 것이다.

ReactiveX의 `Observable`은 Java의 `Future`처럼 단일 스칼라 값의 emission이 아니라 값의 시퀀스 또는 무한한 스트림을 다룬다. `Observable`은 여러 유즈 케이스를 다룰 수 있도록 설계된 단일 추상화이다. 

### Iterator 패턴이 어떻게 적용되었나

> `Observable`은 비동기 / 푸시 방식을 활용하고, `Iterable`은 동기 / 풀 방식을 활용한다.

이벤트 | Iterable (pull) | Observable (push)
--- | --- | ---
데이터 수신 | `T next()` | `onNext(T)`
에러 처리 | throw `Exception` | `onError(Exception)`
완료 처리 | `!hasNext()` | `onCompleted()`

### 리액티브 프로그래밍의 장점

ReactiveX는 이런 `Observable`들을 필터링하고, 선택하고, 변화시키고, 합칠 수 있는 연산자를 제공한다.

Iterator 패턴에서 컨슈머가 프로듀서로부터 값을 풀하는 것과 반대로 `Observable`은 프로듀서가 값이 준비되자 마자 컨슈머에게 값을 밀어넣는 방식으로 동작한다.

`Observable` 타입은 GoF의 옵저버 패턴에 존재하지 않는 두가지 의미를 부여한다.

1. 프로듀셔가 더 이상 사용할 수 없는 데이터가 없는 상태임을 컨슈머에게 알려줄 수 있다.
2. 프로듀서가 컨슈머에게 값을 전달하던 중 오류가 발생했음를 알릴 수 있다. (Iterable은 iteration 도중 에러가 발생하면 Exception을 던지지만, Observable은 옵저버의 `onError` 메서드를 호출한다)

RxJava, RxJS, Rx.NET, RxScala와 같이 다양한 언어들을 위한 포팅이 완료되어 널리 사용중이다.

## FE에의 적용

https://yozm.wishket.com/magazine/detail/1753/

### 앵귤러에서의 활용

앵귤러 프레임워크에서는 RxJS를 내부 상태 관리에 적극적으로 활용한다. (AngularJS가 아니다)

https://angular.io/guide/rx-library

#### Promise로 Observable 만들기

```ts
import { from, Observable } from 'rxjs';

// Create an Observable out of a promise
const data = from(fetch('/api/endpoint'));
// Subscribe to begin listening for async result
data.subscribe({
  next(response) { console.log(response); },
  error(err) { console.error('Error: ' + err); },
  complete() { console.log('Completed'); }
});
```

Promise와 Observable은 다르다..! (Observable이 Promise의 상위 호환이다)

구분 | 동기 | 비동기
--- | --- | ---
단일 값 | value | Promise
복수 값 | Array | Observable

여러 개의 Promise를 (무한한 개수일 수도 있다), 미리 구성한 파이프라인을 통해 어떻게 처리할 것인지 약속해놓는 것이 Observable이다.

#### Event로 Observable 만들기

```ts
import { fromEvent } from 'rxjs';

const el = document.getElementById('my-element')!;

// Create an Observable that will publish mouse movements
const mouseMoves = fromEvent<MouseEvent>(el, 'mousemove');

// Subscribe to start listening for mouse-move events
const subscription = mouseMoves.subscribe(evt => {
  // Log coords of mouse movements
  console.log(`Coords: ${evt.clientX} X ${evt.clientY}`);

  // When the mouse is over the upper-left of the screen,
  // unsubscribe to stop listening for mouse movements
  if (evt.clientX < 40 && evt.clientY < 40) {
    subscription.unsubscribe();
  }
});
```

#### 샘플 요구사항을 구현해보자

> 버튼을 한번 누를 때에는 아무 동작하지 않다가, 버튼을 3번째 누를 때마다 버튼을 몇 번 눌렀는지 alert하는 요구사항이 생겼다고 가정해보자.

리액트로는 어떻게 할까? `useState`와 `useEffect`를 적절히 사용한다면..

```jsx
const [count, setCount] = useState<number>(0);

const handleClick = () => {
    setCount(count + 1);
}

useEffect(() => {
    if (count % 3 === 0) {
        alert("!!!");
    }
}, [count]);
```

앵귤러와 rxjs를 적절히 사용한다면.. (절대 두 라이브러리 / 프레임워크 중 어떤 것이 더 좋다고 얘기하는 것이 아니다)

```ts
export class AppComponent implements OnInit, OnDestroy {

  subscription!: Subscription

  private click$ = new Subject<void>()

  count$: Observable<number> = this.click$.pipe(
    scan(previous => previous + 1, 0),
    tap(count => (count % 3 === 0) ? alert("!!!") : console.log(count))
  )

  ngOnInit(): void {
    this.subscription = this.count$.subscribe()
  }

  ngOnDestroy(): void {
    this.subscription?.unsubscribe()
  }

  onClick() {
    this.click$.next()
  }
}
```

- `scan` operator는 상태를 캡슐화하고 관리하기 유용한 함수이다.
  - accumulator 함수를 사용하여 초기 상태로부터 다음 값을 도출해낼 수 있다.
  - [설명](https://rxjs.dev/api/index/function/scan)
- `tap` operator는 개발자가 부수적인 효과를 특정한 위치에서 부여할 수 있는 함수이다.
  - `map`이나 `mergeMap` 내부에서 이를 행할 수도 있지만, 매핑 함수를 순수하지 못하게 만들 때 `tap`을 사용한다.
  - [설명](https://rxjs.dev/api/operators/tap)

FE 개발을 하다 보면 흔히 마주치게 되는 기능 구현에 대한 요구사항은 보통 다음과 같은 것들이다.
- 채팅방에 5명 이상이 들어온다면 '현재 인기 있는 채팅방' 라벨을 표시하게 해주세요.
- 실시간 차트에서 1분 단위로 실시간 데이터를 가져오고, 특정 값이 변화했을 때 토스트 팝업을 띄워주세요.
- 마우스 스크롤 했을 때 새로운 아이템 목록을 불러오게 해주세요.

> 공통점은 무엇인가? ~ 했을 때 (if) ~ 해주세요. 

js는 동기적인 언어이지만, 요구 사항들이 비동기처리로 이루어져야 하기 때문에 우리는 이벤트 + Promise의 조합을 써왔다.

하지만 요구 사항이 복잡해지고, 인터랙션해야 하는 다양한 컴포넌트들이 화면에 계속해서 추가된다면 Promise에 + Promise에.. 코드가 복잡해지고 따라서 상태 관리도 복잡해지는 경우가 많다. 

이처럼 비동기 처리할 이벤트가 여러 개라면, rxjs를 사용해서 우아하게 요구사항을 처리할 수 있다.
- 채팅방에 들어오는 이벤트를 `Observable`로 만들어, 현재 들어와 있는 인원이 5명이라면 (`filter`) 라벨 표시 
- 실시간 차트에서 1분 단위로 실시간 데이터를 가져오고 (counter를 통한 `Observable` 생성)+ 특정 값이 변화했을 때 (특정 값에 대한 `Observable`) 특정 동작 행하기
- 마우스 스크롤 이벤트로부터 `Observable`을 만들어, `HttpService` 특정 메서드 실행

https://www.learnrxjs.io/learn-rxjs/recipes

앵귤러 + RxJS 폼 미쳤다! 특수한 경우에 대한 예시도 이렇게 잘 구비되어 있다.

#### 앵귤러와 RxJS를 잘 사용해서 debounce 이벤트를 잘 구현한 예시

https://stove99.github.io/javascript/2020/06/29/rxjs-keyword-debounce/

```ts
import { Component, OnInit } from '@angular/core';
import { Subject, Observable } from 'rxjs';
import { map, filter, debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
import { HttpClient } from '@angular/common/http';

@Component({
    selector: 'app-root',
    template: `
        <input type="text" class="ml-2" [ngModel]="''" (ngModelChange)="onTextChange($event)" />

        <div *ngIf="(searchResult$ | async) as searchResult">
            <h2>저장소 목록</h2>

            <ng-container [ngSwitch]="searchResult.items.length">
                <ng-container *ngSwitchCase="0">
                    No results found
                </ng-container>

                <ng-container *ngSwitchDefault>
                    <div *ngFor="let result of searchResult.items">
                        
                    </div>
                </ng-container>
            </ng-container>
        </div>
    `,
    styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit {
    queries$ = new Subject<string>();
    searchResult$: Observable<any>;

    constructor(private http: HttpClient) { }

    ngOnInit() {
        this.searchResult$ = this.queries$.pipe(
            map((query: string) => query ? query.trim() : ''),  // 검색어 트림처리
            filter(Boolean),    // 트림결과 문자가 있는 경우
            debounceTime(500),  // 500ms debounce 처리
            distinctUntilChanged(), // 이전 입력값과 다른경우
            switchMap((id: string) => this.find(id)) // 검색 api 호출해서 결과값으로 Observable 변경
        );
    }

    onTextChange(id: string) {
        this.queries$.next(id);
    }

    private find(id: string): Observable<any> {
        return this.http.get<any>('https://api.github.com/search/repositories', { params: { q: id } });
    }
}
```

#### 참고) 어떤 Operator를 써야 할까

https://rxjs.dev/operator-decision-tree

[Reactor 공식 문서](https://projectreactor.io/docs/core/release/reference/#which-operator)에는 상당히 불친절하게 되어 있는 반면, rxjs는 원하는 연산자를 굉장히 쉽게 찾을 수 있도록 해놓은 모습이다 ㅠ

## BE에의 적용

내가 써봤던 JS 백엔드 라이브러리는 NestJS가 유일해서, 잘 정리되어 있었던 포스트를 소개한다.

https://blog-ko.superb-ai.com/nestjs-interceptor-and-lifecycle/

### Nestjs에서 `intercept`를 사용하여 요청 처리 도중에 원하는 로직 넣기

https://docs.nestjs.com/interceptors

nestjs의 인터셉터는 AOP 기법을 사용해서 컨트롤러 / 서비스 / 도메인 간 공통된 로직을 분리 적용하기 아주 좋은 기능이다.

모든 인터셉터들은 `intercept()` 메서드를 구현해야 한다. `intercept()` 메서드는 두 개의 인자를 받는다.
1. `ExecutionContext` 인스턴스 (가드가 사용하는, `ArgumentHost`를 상속받는 그 객체와 동일하다)
2. `CallHandler` 인터페이스

- `CallHandler` 인터페이스는 `handle()` 메서드를 구현하여 인터셉터의 한 부분에서 라우트 핸들어 메서드를 사용할 수 있게 한다. 
- `handle()` 메서드를 구현하지 않는다면 라우트 핸들러 메서드는 아예 실행되지 않는다.

즉 이는 `intercept()` 메서드를 통해 효과적으로 요청 / 응답 객체 스트림을 감쌀 수 있다는 것을 의미한다.
요청과 응답을 감싸기 위한 커스텀 로직이 있다면, 최종 라우트 핸들러가 실행 되기 전이나 되고 난 후에 실행하도록 처리할 수 있는 것이다. 그리고 **이는 `handle()` 메서드가 `Observable`을 반환하기 때문에 가능하다.**

예를 들어 `POST /cats` 요청을 처리하는 API가 있다고 가정해보자.
- 요청은 `CatsController` 내부의 `create()` 핸들러에 의해 처리된다.
- 만약 인터셉터가 `handle()` 메서드를 정의하지 않는다면 `create()` 메서드는 실행되지 않는다.
- `handle()` 메서드가 실행되고, `Observable`이 리턴된다면 `create()` 핸들러는 트리거된다.
- 응답 스트림이 `Observable`로부터 수신된다면, 부가적인 연산이 스트림에 행해진 후 요청자에게 최종 결과가 리턴된다.

```ts
import { Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor {
  intercept(context, next) {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

`Observable`을 사용한다면 매우 간단하게 로깅 인터셉터를 만들 수 있다.

> Reactive Stream의 가장 큰 장점은 비동기로 이뤄지는 여러 이벤트에 대한 복잡한 비즈니스 로직 구현을 보기 쉽게 처리할 수 있다는 것이다.

