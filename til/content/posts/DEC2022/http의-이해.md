---
title: "Http의 이해"
date: 2022-12-18T19:50:18+09:00
draft: false
author : redjen
---

# HTTP의 이해

## HTTP

- 어플리케이션 레벨의 프로토콜
- 신뢰할만한 전송 / 세션 레이어의 **연결**을 통해 메시지를 주고 받는 무상태 요청 / 응답 프로토콜
- HTTP 클라이언트는 서버와 연결을 맺고 하나 이상의 HTTP 메시지를 보내는 프로그램
- HTTP 서버는 클라이언트의 연결을 수락하고, HTTP 요청을 처리하여 응답 전송하는 프로그램

HyperText Transfer Protocol

생각나는 응답코드들..
- 200 (ok)
- 201 (created)
- 401 (unauthorized)
- 403(forbidden)
- 404(not found)
- 500 (internal server error)
- 502 (bad gateway)

HTTP 0.9 (1991)
HTTP 1.0 (1996) - [RFC 1945](https://www.rfc-editor.org/rfc/rfc1945)
HTTP 1.1 (1999) - [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616)

URI / URL 차이점 ??
URI : 식별자 
URL : URI의 한 종류
HTTP URL 이라고 표현하는 것이 맞다.
URI는 ftp, http.. 등 전부 포함하는 개념

https://httpwg.org/specs 에서 HTTP 프로토콜 관련 규약 상세 확인 가능

브라우저에 naver.com 을 입력하면 벌어지는 일??
1. naver.com 도메인을 IP로 변환 (DNS 질의)
2. 가져온 IP 주소에 TCP 연결 시도 (3 way handshake SYN 패킷 전송)
3. 전송한 패킷이 라우팅 알고리즘에 의해 최적의 경로로 서버에 전달
5. SYN 패킷 받은 서버가 ACK 패킷 다시 클라이언트가 위치한 호스트로 전달
6. 다시 SYNACK..
7. TCP Connection 수립. 
8. HTTPS 사용한다면, 클라이언트가 사용할 대칭키를 서버에게 전달 및 TLS 암호화 적용
9. 클라이언트가 HTTP GET 요청을 서버에게 전달
10. TCP 연결로부터 받은 메시지를 알맞은 포트 바인딩되어 있는 어플리케이션에 전달
11. 어플리케이션은 GET 요청에 따른 동작 수행 - HTTP 문서 반환
12. 클라이언트가 HTTP 문서 다운로드
13. 화면 짠

Root DNS, Proxy, Gateway, CDN 키워드에 대해 찾아보자..

User Agent는 request 날리면 Origin은 request에 대한 response를 반환한다..

## HTTP의 동작 방식

HTTP/0.9에서는 GET 메소드 밖에 없었다
쓸만하게 바뀐건 HTTP/1.0 부터, POST의 등장

### HTTP 메시지의 구조

재미있는 프로토콜 - Plain ASCII 기반한 프로토콜
그 당시에는 메시지 비용이 비쌌지만 메시지 비용이 저렴해진 요즘에는 우월한 프로토콜 되었음
왜 이렇게 만들었을까? Human Readable 하게 만든 이유가 궁금..
Start Line - Header - Blank Line - Body의 구조.
- 송신할 때에는 Request-Line이라고 부름
- 수신할 때에는 Status-Line이라고 부름

HTTP 입장에서 Encoding : 압축 방식 정의, 최근에는 br 알고리즘이 효율이 좋다더라
Header 중에 X 붙이는 시리즈 : 커스텀 헤더일 가능성 높음
	request header의 q는 accept 하는 형식의 우선순위를 나타낸다

다국어를 지원하는 페이지들은 헤더를 보고 언어 간 우선순위를 적용한 리소스로 보여준다
	사용자 입장에서는 불편한 경우도 있음
		bilingual, 여러 개의 언어를 사용하는 사용자들은 language 선택할 때 좀 더 자연스러움

이전에는 Pragma 헤더를 사용해서 캐시 컨트롤했었음
지금은 Cache-Control 헤더를 사용하는 추세, 둘 다 있다면 호환성을 위해 사용하는 것

**HTTP는 Stateless한 프로토콜**

커넥션을 끊지 않고 계속 통신하고 싶어 - Connection : keep-alive로 설정
그런데 대규모 서비스를 운영하는 입장에서는 소켓 여러 개를 점유하고 있는 상태를 계속 유지해야..
-> 튜닝 포인트!! Connection: keep-alive를 키는 게 좋을까? 끄는 게 좋을까?
- 껐더니
	- 유효한 소켓 내에서 처리 다 하고 TCP 연결 종료.
	- 짧은 시간 내 더 많은 클라이언트들을 처리할 수 있게 함
	- page는 떴는데 image는 엑박뜨거나, 로그인 버튼을 눌렀는데 처리 안되는 경우가 생김

절대값이 있는 것은 아니지만..

- 짧은 시간 내에 사용자의 동작까지 받는 것을 목적한다면 timeout을 늘리는 방향으로 변화
- 더 심한 대용량 트래픽을 감당해야 하는 경우 1초 timeout을 사용하는 경우도 있음
	- 접속해서 일을 처리하고 나가는데 걸리는 시간을 timeout으로 설정하는 것이 중요

POST / PUT의 차이점
- PUT는 리소스의 location을 주도록 설정되어 있음.
	- 정확한 location을 알아야 호출 가능함
- PATCH도 리소스의 위치를 알고 있다고 가정하고 받아야

OPTIONS : preflight로 accept 하는 옵션을 미리 확인하고 싶을 떄 사용
HEAD : head만 먼저 알고 싶을 때 (e-tag만 알고 싶을 때), 어떤 리소스에 대한 정보만 알고 싶을 때
TRACE : 중간에 거쳐간 프록시가 어떤 것이 있는지 확인할 수 있음

Idempotent : 여러 번 호출해도 의도한 효과는 한번 호출한 것과 같은 성질
Safe : 근본적으로 read-only 인 효과를 의도한 요청 (GET, HEAD 등)

### 상태 코드

- 1xx : informational. 정보를 알려줌 
- 2xx : successful. 성공했다
- 3xx : redirection. 다른 데를 봐라
- 4xx : client error. 클라이언트가 잘못함
- 5xx : server error. 서버가 잘못함

## Cache

언제 사용하는가?
- Conditional Get 사용해서 트래픽 줄일 수는 있지만, Round Trip은 줄일 수 없다
- 어떤 리소스에 변경이 있었는지 확인하려면 반드시 원 서버에 요청 보내고 응답 올 때까지 대기해야만..
- 원 서버가 먼 곳에 있다면 해당 지연이 수백 ms 될수도
- 사용자가 오래 기다리지 않고 빨리 리소스 받게 하려면?

Cache : 사용자가 이전과 같은 요청을 하면 이전과 같은 응답을 준다

fresh와 stale
- fresh : freshness_lifetime > current_age 이면 fresh
	- freshness_lifetime = expires_value - date_value
	- current_age = 캐시에서 머무른 시간 + 응답 오는데 걸린 시간 + 기타 다양한 보정
- stale : freshness lifetime 지나면 다 stale

캐시 기본 동작
- 몇 가지 예외 빼면 캐시는 성공한 모든 응답 저장
- fresh한 entity의 경우, validation 없이 반환 가능
- stale한 entity의 경우, validation에 성공했다면 반환
	- 실패해도 반환하는 경우 - 인터넷 접속 끊긴 경우
- Expires 헤더 없는 경우 캐시는 적당히 Freshness Lifetime 계산
- 상기 기본 동작 수정하려면 Cache-Control 헤더 사용

Cache-Control : no-cache == Pragma : no-cache
	Expires의 절대 시각 대신 상대 시간 표시 가능

## Cookie

Set-Cookie 응답 헤더를 통해 쿠키를 서버에서 클라이언트로 전송
클라이언트는 쿠키를 보관하고 있다가 필요 시 Cookie 헤더를 통해 서버에 전송

- Session Cookie : 브라우저 닫으면 사라짐
- Persistent Cookie : 도메인 지정 여부에 따라 동작 상이
