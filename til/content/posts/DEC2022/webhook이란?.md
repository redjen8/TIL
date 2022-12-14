---
title: "Webhook이란?"
date: 2022-12-22T16:43:14+09:00
draft: false
author : redjen
---

# Webhook이란? 

## 개요

Webhook은 2개의 웹 API 간에 이벤트 기반 통신을 가능하게 하는 HTTP 콜백 기능
웹 어플리케이션이 다른 어플리케이션이 보낸 데이터 조금을 받고 싶을 때 사용하거나
Jenkins와 같은 DevOps 자동화 워크 플로우를 트리거하는데에도 사용한다.

## 동작 방식

클라이언트가 서버 API에 고유한 URL을 부여하고, 알고 싶은 이벤트를 지정한다.
webhook은 클라이언트가 서버의 상태를 체크하기 위해 비효율적으로 폴링하는 로직을 제거할 수 있다. 
webhook은 어떻게 보면 push api. 어떻게 보면 reverse api

- 서버가 응답할 때 까지 데이터 요청을 전송하는 대신 (폴링)
- 서버가 데이터 준비되면 클라이언트에게 HTTP 요청 전송.

여기서 서버와 클라이언트는 전부 웹 어플리케이션 임에 유의
그런데 서버 역할의 어플리케이션이 아무 클라이언트에게 뭔가 푸시하는 기능은 위험할 수 있다
따라서 클라이언트 역할의 어플리케이션이 서버 역할 어플리케이션에 등록할 때
나를 뜻하는 api key를 지정하여 그 키를 통해 '너는 이전에 내가 등록한 어플리케이션' 이구나를 식별

## 장점

- 폴링할 필요가 없다.
	- 폴링의 단점인 불필요한 통신이 야기하는 리소스 낭비를 없앨 수 있다
	- 필요할 때 내가 널 부른다
- 빠른 설정이 가능하다.
	- 웹 훅 url과 어떤 이벤트가 웹 훅으로 요청을 트리거할 지 정한다.
	- 또 보안을 위해 클라이언트가 미리 인증 받은 어플리케이션이라는 것을 식별하기 위한 key 값만 정하면 된다
- 데이터 전송을 자동화할 수 있다. 
	- 이벤트 발생 직후 바로 요청이 미리 정의했던 대로 자동화되므로 거의 실시간으로 플로우를 컨트롤 가능하다
- 페이로드의 양이 적을 때 유용하다.  
	- 알림과 같은 가벼운 기능을 개발할 때 매우 유용하게 사용할 수 있다. 

## 사용 예시

Github Webhook을 사용하여 특정 branch에 push 이벤트 발생 시
- 소스코드를 빌드 후 빌드 파일을 Jenkins 워크 플로우로 전달
- Jenkins에서 서버 배포까지 수행 (CI 와 CD의 연동)

어플리케이션에서 감지하고 싶은 이벤트나 오류 발생 시
- Slack과 같은 webhook을 지원하는 메신저 어플리케이션에 사전 이벤트 등록
- 이벤트 발생 시 Push 알림 수신을 통해 장애의 빠른 전파 가능 (장애의 빠른 복구)