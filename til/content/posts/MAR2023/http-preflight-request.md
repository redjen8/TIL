---
title: "Http Preflight Request"
date: 2023-03-16T23:02:59+09:00
draft: false
author: redjen
---

HTTP Preflight Request란 무엇이고, 어떤 내용을 포함하고 있을까?

https://developer.mozilla.org/ko/docs/Glossary/Preflight_request

## Preflight Request란

- 본격적인 HTTP 요청 전 서버 쪽에서 그 요청에 대한 메서드와 헤더에 대해 capable한지를 체크하는 요청이다.
- 다음 헤더 항목을 포함하는 HTTP OPTION 요청이다.
  - Access-Control-Request-Method
  - Access-Control-Request-Headers
  - Origin

## 언제 사용하는가

일반적인 상황이라면 브라우저가 이를 자동으로 발생시킨다.

때문에 FE 개발자가 이를 고의적으로 발생시킬 필요는 없다. DELETE 요청을 보내기 전 미리 서버가 이를 지원하는지 확인하는 용도로?

## CORS와의 관계

어라, 그런데 Preflight Request 연관 검색어에 CORS가 있었다.

브라우저는 어떻게 Preflight Request를 사용해서 CORS 정책을 구현할까?

### CORS 발생 과정

https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

지금까지 설명했듯이 기본적으로 브라우저는 어떤 HTTP 요청을 하기 전에 Preflight Request를 날린다.

**그리고 이 Preflight Request에서 브라우저는 실제로 전송될 HTTP Method와 Header를 보낸다.**

CORS를 공부해봤다면 이미 알고 있는 내용이겠지만..
- CORS 정책은 서버가 브라우저에서 해당 정보를 읽을 수 있도록 허용된 출처를 명시하는 HTTP 헤더를 제공하여 동작한다.
- 때문에 서버로 요청을 보낼 때, 브라우저는 HTTP OPTION 메서드를 사용한 요청을 통해 나는 이런 요청을 할거야~ 라고 알려주고
- 정책 위반이 아니라면 서버는 요청에 대한 승인과 함께 실제 데이터를 리턴한다.
- CORS 정책 위반이라면, 서버는 요청을 거절한다.

더 자세히 들어가면, CORS 정책 대상에는 크게 두 가지 종류가 있다.
1. Simple Requests
2. Simple Request가 아닌 요청들

### Simple Request는 어떤 녀석들일까

- `GET`, `HEAD`, `POST` 메서드 중 하나
- User-Agent에 의해 자동으로 설정된 헤더가 없는 요청, 허용되는 헤더들은 다음과 같다.
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type`
  - `Range`
- 허용되는 Media Type은 아래와 같다.
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `text/plain`
- `ReadableStream` 객체가 요청에 포함되지 않는 요청

그리고 OPTION 요청을 실제 요청 전에 보내는 경우는 2번, Simple Request가 아닌 경우이다. 