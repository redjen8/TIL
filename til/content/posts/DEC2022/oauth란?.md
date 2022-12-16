---
title: "Oauth란?"
date: 2022-12-16T15:19:18+09:00
draft: false
author : redjen
---

# OAuth란?

## OAuth에 대해

인터넷 사용자들이 비밀번호 제공 없이 다른 웹 사이트 상의 자신들의 정보에 대해 웹 사이트나 어플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로 사용되는, **접근 위임**을 위한 개방형 표준

Resource Server : 클라이언트가 제어하고자 하는 **자원을 보유하고 있는 서버** (FB, Google, Twitter 등)

Resource Owner : 자원의 소유자. 클라이언트가 제공하는 서비스 통해 로그인하는 **실제 유저**

Client : Resource server에 접속해서 정보를 가져오려 하는 클라이언트 (**웹 어플리케이션**)

## 동작 플로우

1. 클라이언트 (웹 어플리케이션)이 리소스 서버 사용을 위해 사전에 서비스를 등록. 이 때 등록되는 정보로는
	1. 클라이언트 식별자 (ID)
	2. 클라이언트 시크릿 (비밀키)
	3. Authorized Redirect URL : Authorization Code를 전달 받을 리다이렉트 주소
2. Resource Owner의 승인 - 실제 유저는 웹 어플리케이션 사용하다가 소셜 로그인 버튼을 클릭
	1. 이 때 실제 유저가 Resource Server에 접속해서 로그인 수행
	2. 로그인 완료되면 Resource Server는 쿼리 스트링으로 넘어온 파라미터 통해 클라이언트 검사
		1. 파라미터로 전달된 클라이언트 ID와 동일한 ID값이 존재하는지
		2. 해당 클라이언트 ID에 해당하는 Redirect URL이 파라미터로 전달된 Redirect URL과 일치하는지
	3. 검증 후에 Resource Server는 실제 유저에게, 명시한 스코프에 해당하는 권한을 클라이언트 (웹 어플리케이션)에게 정말로 부여할 것인지 물어봄
3. Resource Server의 승인 - 실제 유저의 승인이 떨어지면 명시된 Redirect URL로 클라이언트를 리다이렉트. 이 때 Resource Server는 클라이언트가 자신의 자원을 사용할 수 있는 엑세스 토큰 발급 전에 임시 암호인 Authorization code 함께 보냄
	1. 클라이언트는 ID와 비밀키, Authorization Code를 실제 유저의 개입 없이 Resource Server에 직접 전달.
	2. Resource Server는 정보를 검사한 다음 유효한 요청이라면 엑세스 토큰 발급
	3. 클라이언트는 해당 토큰을 서버에 저장해두고 Resource server의 자원을 사용하기 위한 API 호출 시 해당 토큰을 헤더에 담아 전송


