---
title: "Service Worker의 이해"
date: 2023-05-17T21:47:08+09:00
draft: false
author: redjen
---

## 개요

https://developer.mozilla.org/ko/docs/Web/API/Service_Worker_API

서비스 워커는 웹 애플리케이션, 브라우저, 네트워크 사이 프록시 서버 역할을 수행한다.

> 서비스 워커의 개발 의도는 여러 가지가 있지만, 그 중에서도 효과적인 오프라인 경험을 생성하고 네트워크 요청을 가로채서 네트워크 사용 가능 여부에 따라 적절한 행동을 취하고, 서버의 자산을 업데이트 할 수 있습니다.

즉 웹 애플리케이션을 개발하는 입장에서 서비스 워커를 잘 활용한다면 웹 애플리케이션의 확장성을 크게 늘릴 수 있다는 장점이 있다.

### 개념

서비스 워커는 출처 / 경로에 대해 등록하는 **이벤트 기반 워커**이다.

자바스크립트 파일의 형태를 가지고 있으며, 연관된 웹 페이지를 통제하여 탐색과 리소스에 대한 요청을 가로채 수정하고 세부적인 캐싱을 돕는다.

### 서비스 워커의 특징

- 서비스 워커는 [워커](https://developer.mozilla.org/ko/docs/Web/API/Worker)이다. 그렇기 때문에 브라우저 내의 DOM에 접근 또는 DOM 조작이 불가능하다. (서로 다른 컨텍스트 공간을 가지기 때문이다)
- 서비스 워커는 애플리케이션이 실행되는 쓰레드와는 서로 다른 쓰레드에서 동작한다. 때문에 애플리케이션의 연산을 블락킹하지 않는다.
- 서비스 워커는 완전 비동기적으로 설계되었기 때문에 동기적 API인 XHR이나 웹 스토리지 API를 서비스 워커 내에서 사용할 수 없다.
- 서비스 워커는 보안 설정 때문에 HTTPS에서만 동작한다. 
	- 어떻게 보면 서비스 워커 자체가 MITM이기 때문에 신원에 대한 최소한의 검증을 HTTPS 인증을 통해 수행한다라고 생각


## 서비스 워커의 활용

https://medium.com/commencis/what-is-service-worker-4f8dc478f0b9

### 이건 할 수 있어요

1. 네트워크 트래픽을 통제할 수 있다. 공항의 관제탑처럼 모든 트래픽은 서비스 워커의 통제를 받도록 설정할 수 있다.
	1. 때문에 페이지가 CSS 파일을 요청했을 때, response를 plain text로 변경하는 작업을 수행하거나
	2. HTML 파일을 요청했을 때 응답을 png 파일로 변환해서 내려주는 작업도 수행할 수 있다.
2. 캐싱을 수행할 수 있다.
	1. 서비스 워커가 가장 빈번하게 사용되는 시나리오이며, 캐시 API를 사용해 네트워크 상황이 끊어졌을 때에도 (오프라인일 때에도) 요청에 대한 응답을 받도록 할 수 있다.
3. 푸시 알림을 관리할 수 있다.
4. 인터넷 연결이 끊어졌을 때에도 서비스 워커의 백그라운드 싱크를 통해 작업을 이어갈 수 있다.

### 이건 하기 어려워요

1. DOM 조작 (`window`에의 접근)이 불가능하다.
	1. 서비스 워커에서 작업한 결과를 직접적으로 DOM에 내려주고 싶어도 `postMessage` 등의 한 단계를 거쳐서 표시해줘야 한다.
2. 80 포트에서는 동작하지 않는다.
	1. HTTPS를 기반으로 동작하기 때문이다.
	2. 하지만 개발 중엔 로컬 호스트에서 동작할 순 있다.

## 서비스 워커의 라이프 사이클

서비스 워커를 사용하려면 서비스 워커의 라이프 사이클이 어떻게 돌아가는지를 알아야한다.

크게 등록 (Registration) - 설치 (Installation) - 활성화 (Activation)의 3단계를 따른다.

### 1. 등록

서비스 워커는 `window.navigator` 객체의 프로퍼티이다. 때문에 등록 과정에서는 `register` 함수를 통해서 서비스 워커를 등록할 수 있다. (`register` 함수가 프로미스를 리턴하는 것에 유의)

https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer

https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register

이 `register` 함수는 두 개의 인자를 가지는데,
1. 서비스 워커 스크립트의 URL.등록된 서비스 워커 파일은 유효한 자바스크립트 MIME 타입을 가져야 한다.
2. 옵션 (optional) - 서비스 워커의 동작을 정의할 스코프나 타입 등을 정의할 수 있다.


```js
if ("serviceWorker" in navigator) {
  // Register a service worker hosted at the root of the
  // site using the default scope.
  navigator.serviceWorker.register("/sw.js").then(
    (registration) => {
      console.log("Service worker registration succeeded:", registration);
    },
    (error) => {
      console.error(`Service worker registration failed: ${error}`);
    }
  );
} else {
  console.error("Service workers are not supported.");
}
```

상기 예시에서는 `register` 함수에 하나의 인자만을 전달했다. (option을 따로 지정하지 않음)

이 경우 서비스 워커 코드가 `example.com/index.html`에 포함되면 그 하위 페이지에 대해서도 제어권을 가지게 된다. (`example.com/product/description.html` 등)

서비스 워커가 동작하는 스코프를 제어하려면 두 번째 인자로 `{ scope : "/product/" }` 와 같이 전달해주면 된다.

### 2. 설치 이벤트

https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/install_event

설치 이벤트가 발생했을 때에 대한 동작을 정의할 수 있다.

그리고 설치 이벤트는 `sw.js` 파일이 존재하지 않거나 업데이트 되었을 때 발생한다.

```js
self.addEventListener("install", async e => {
	console.log("sw installed")l
	let cache = await caches.open("pwa-static");

	cache.addAll([
		"./",
		"./main.js",
		"./styles.css"
	]);
});
```

동적인 컨텐츠가 아닌 정적인 컨텐츠를 캐싱해야 할 필요가 있다면 설치 이벤트가 발생했을 때 저장하는 것이 이상적이다.

### 3. 활성화 이벤트 

활성화 이벤트가 발생했을 때에 대한 동작을 정의할 수 있다.

활성화 이벤트는 설치 이벤트가 완료되었을 떄 트리거되는데, 현재 페이지가 등록 시에 설정한 스코프 바깥에 있다면 활성화 이벤트는 트리거 되지 않는다.

이 이벤트는 새로운 버전의 컨텐츠가 있을 때 이미 캐시되었던 정적 컨텐츠를 제거하기 위한 이상적인 장소이다.

알아둬야 할 것은 만약 `sw.js`가 업데이트되고 현재 페이지를 새로 고침한다고 해서 새로운 파일을 가져오는 것은 아니라는 점이다.
- 존재하는 모든 윈도우와 탭들을 전부 닫고 업데이트를 위해 다시 시작해야 한다.
- 이 과정을 백그라운드에서 실행하고 싶다면 `self.skipWaiting()` 함수를 사용해서 설치 이벤트가 발생할 때 수행할 수 있다.

### 4. 페치 이벤트

페치 이벤트를 다루면 네트워크 트래픽을 추적하고 관리할 수 있다. 

크게 두 가지 동작으로 구분되는데,
1. 먼저 존재하는 캐시를 체크하는 `Cache First` 전략
2. 먼저 네트워크 요청을 보내는 `Network First` 전략

Cache First을 사용한다면 이전 요청에 대한 응답이 캐시되어 있다가 리턴된다. 만약 캐시에 저장된 응답이 없다면 새 응답을 네트워크를 통해 받아온다.

Network First를 사용한다면 먼저 네트워크 요청을 통해 업데이트 된 응답을 받아오는 시도를 한다.
- 이 과정이 성공적으로 완료된다면 새롭게 받아온 해당 응답이 캐시된 후에 리턴된다.
- 이 과정이 실패한다면 (네트워크 상황에 따라) 해당 요청이 캐시되었는지 확인한다.
	- 해당 요청에 대한 캐시가 존재한다면 페이지로 리턴된다.
	- 해당 요청에 대한 캐시가 없을 때에 대한 동작은 커스텀 정의할 수 있다. (더미 컨텐츠를 리턴하거나 페이지로의 특정 정보를 전달하거나)

### 5. 싱크 이벤트

백그라운드 싱크는 인터넷 연결 상태가 안정화 될 때까지 '작업을 미루는' 웹 API이다.

이 정의를 실 생활에 적용할 수 있는데, 브라우저에서 동작하는 이메일 클라이언트 애플리케이션이 있고 이 툴을 사용해서 이메일을 발송하려 하는 상황을 가정해보자.
- 이메일 컨텐츠를 작성하는 동안 인터넷 커넥션이 끊어졌지만 사용자는 이를 눈치채지 못했다.
- 컨텐츠를 전부 작성한 이후에, 전송 버튼을 클릭한다.
- 하지만 전송 버튼을 누른 직후에 작성한 컨텐츠는 커넥션이 끊어졌기 때문에 즉시 발송되지 못한다.
- 이 때 백그라운드 싱크를 사용할 수 있다.

전송 버튼을 눌렀을 때, 실제로 컨텐츠를 즉각적으로 발송하는 것이 아니라
1. [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)에 작성한 컨텐츠를 임시 저장한다.
2. 백그라운드 싱크를 등록한다.
3. 인터넷 연결이 안정 상태라면, 저장한 이메일 컨텐츠를 꺼내어 메일 서버로 전송한다.
4. 인터넷 연결이 불안정하다면 서비스 워커는 커넥션이 가용 상태가 될 때까지 기다린다. (윈도우가 닫힌 상태에서도 동작한다) 이후 커넥션 상태가 괜찮아진다면 이메일 컨텐츠는 메일 서버로 보내진다.

### 6. 푸시 이벤트

https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push_event

서비스 워커는 푸시 메시지를 받았을 때에 대한 이벤트 핸들러도 등록할 수 있다.

```js
self.addEventListener(
  "push",
  (event) => {
    let message = event.data.json();

    switch (message.type) {
      case "init":
        doInit();
        break;
      case "shutdown":
        doShutdown();
        break;
    }
  },
  false
);
```

상기 예제는 간단한 JSON 데이터를 파싱해서 메시지에 따른 동작을 사전 정의한다.

## 서비스 워커의 보안

https://chromium.googlesource.com/chromium/src/+/main/docs/security/service-worker-security-faq.md

서비스 워커는 기존 웹 애플리케이션의 한계를 넘어 더 강력한 기능을 사용할 수 있도록 한다.

그렇기 때문에 개발자가 더 많은 책임을 가지고 개발해야 한다는 단점도 분명 존재한다.
서비스 워커는 마냥 위험하기만 한 물건일까??

- 서비스 워커는 격리된 환경에서 돌아간다. DOM에 접근할 수 없을 뿐만 아니라 샌드박스 환경에서 실행됩
- 서비스 워커는 영원히 동작하지도 않는다. 최대 24시간마다 HTTP 캐시가 재검증되기도 하고, 브라우저에서 동작하는 모든 서비스 워커의 쓰레드를 언제라도 종료할 수 있다. (크롬의 경우 30초 이상 유휴 상태인 서비스 워커를 종료시킨다)
- xss 취약점이 있는 사이트의 경우 공격자가 서비스 워커를 등록하여 임의의 스크립트를 실행시킬 수 있기 때문에 (!) 
clear-site-data와 같은 전략들을 사용한 서비스 워커를 개발해야 한다. 
(https://www.w3.org/TR/clear-site-data/)