---
title: "Transfer Encoding에 대해"
date: 2023-02-11T00:03:56+09:00
draft: false
author: redjen
---

여느 때와 다름 없이 개발을 하던 중, `ERR_CONTENT_LENGTH_MISMATCH` 에러를 만나게 되었다. 찾아보니 response header의 `content-length`와 실제 body로 오는 응답의 길이가 상이해서 생기는 문제였다.

문제 자체가 복합적인 상황에서 발생할 수 있기 떄문에 처해 있는 종합적인 컨텍스트를 파악하는 것이 문제 해결에 도움이 될 수 있다.

나는 이번 에러를 두 가지 방법으로 해결했다.
1. node.js 프록시 서버의 `Transfer-Encoding` 헤더 값을 `chunked`로 설정
2. 애초에 응답 값을 받지 않아도 되는 API 였기 때문에, 리턴 형식을 `Mono<Void>` 으로 변경

두 가지 해결책 모두 문제가 발생하는 근본적인 원인을 제거하는 것이 아니라 임시 방편에 불과하다. 나중에 심심한 나를 위해 단서만 이 곳에 던져 놓고 가겠다.

> 웹플럭스 서버에서 사용하지 않는 Request Body에 큰 용량의 데이터를 보내는 동안 요청이 완료되는 경우 요청에 대한 응답 값은 어떻게 될까?

이와는 별개로, `Transfer-Encoding`에 대해 알게 된 것을 이 곳에 정리해보려 한다.

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding

## 정의

`Transfer-Encoding` 헤더는 클라이언트에게 페이로드로 body 값을 안전하게 전달하기 위해 사용하는 인코딩 형식을 정의한다.

`Transfer-Encoding` 헤더는 hop-by-hop 헤더이다. 즉 리소스 자체가 아니라 네트워크 상의 두 노드에서 전달되는 메시지에 붙게 되는 것이다. 다수의 노드를 지나는 연결에서 각각의 세그먼트는 서로 다른 `Transfer-Encoding` 값을 사용할 수 있다. 만약 전체 커넥션에서 데이터를 압축하기를 원한다면, `Content-Encoding` 헤더를 써야 한다.

만약 본문이 없는 HEAD 요청에 대한 응답에 `Transfer-Encoding` 헤더가 있는 경우는 해당 GET 메시지에 적용된 값을 미리 나타낸다.

## 문법과 종류

아래 4개의 디렉티브들을 단일로도 사용할 수 있으며 (`Transfer-Encoding: chunked`) 콤마로 구분하여 여러 값을 사용할 수도 있다. (`Transfer-Encoding: gzip, chunked`)

### 1. chunked

- 데이터가 여러 청크의 시리즈로 전송된다.
- `Content-Length` 헤더가 생략된다.
- 각 청크의 시작 부분에 현재 청크의 길이를 16진수로 추가한 후 `\r\n`을 추가한다.
- 다음 청크를 추가한 이후에 다시 `\r\n`을 추가한다.
- 마지막 청크는 보통 청크와 동일하다. 길이가 0인 경우만 빼고.
- 그 후에는 (비어 있을 수 있는) 헤더 필드의 시퀀스가 나타난다.

### 2. compress

- LZW 알고리즘을 사용하는 포맷이다. (UNIX 압축 프로그램에서 따온 이름)
- 특허 이슈가 있어 원래 이름을 따온 압축 프로그램처럼 현재는 거의 사용되고 있지 않다.
  
### 3. deflate

- RFC 1950에 정의된 zlib 구조를 사용하는 deflate 압축 방식을 사용한다.

### 4. gzip

- LZ77 알고리즘을 사용하는 포맷이다.
- 원래 UNIX gzip 프로그램에 들어 있던 포맷이다.
- HTTP 1.1 표준에서는 이 content-encoding을 사용하는 서버는 호환성을 위해 `x-gzip` 또한 인식해야 한다고 명시되어 있다.

## 사용에 대해..

Chunked Encoding은 다음과 같은 경우에 유용하다.
- 큰 크기의 데이터가 클라이언트로 보내질 때
- 응답의 전체 크기를 요청이 완전히 완료되기 전까지 알 수 없을 때

예를 들어, 데이터베이스 쿼리에서 가져온 데이터를 담은 큰 HTML 테이블을 생성하거나 큰 크기의 이미지들을 전송하는 경우에 고려해볼 수 있다.
