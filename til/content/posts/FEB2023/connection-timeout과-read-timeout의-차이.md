---
title: "Connection Timeout과 Read Timeout의 차이"
date: 2023-02-12T20:58:31+09:00
draft: false
author: redjen
---

Connection Timeout 과 Read Timeout은 서비스 설계에 가장 기초적이면서도, 큰 영향을 줄 수 있는 인프라 레벨의 설정 값들이다.

기본적으로 가장 앞단 서버 또는 로드 밸런서에 적용되는 Connection Timeout과 Read Timeout이 어떻게 다른지에 대해 정리해봤다.

https://stackoverflow.com/questions/3069382/what-is-the-difference-between-connection-and-read-timeout-for-sockets

## Connection Timeout

Connection Timeout은 초기 TCP 커넥션을 생성하는 과정에서의 타임아웃이다. 즉 TCP 커넥션 핸드쉐이크 (3-way로 알려진) 과정을 완료하는 단계에서의 타임아웃이다.

## Read Timeout

Read Timeout은 데이터를 읽기를 기다리는 동안에 발생하는 타임아웃이다. 클라이언트가 `read` 명령을 내린 이후에 `<timeout>` 초 만큼 서버가 어떠한 데이터도 내려주지 않으면 발생한다. 

## 무한대로 설정하게 된다면?

JVM 위에서 동작하는 서버들에 대해 설정하는 타임아웃 값들을 무한대로 설정해 놓으면 영원히 프로그램이 블록킹 상태에 돌입할까? 그렇지 않다. 이 경우 운영체제가 타임 아웃에 대한 권한을 위임 받아 역할을 수행한다.

## 타임아웃 값은 어떻게 설정해야 할까?

https://alden-kang.tistory.com/20

'좋은 타임아웃 값' 설정은 아래 두 가지 기준을 만족해야 한다.

- 네트워크 상 패킷 손실은 언제든 발생할 수 있다.
  - 타임아웃 값을 너무 tight 하게 설정할 경우 네트워크 장애 상황이 아닌 상황도 장애 상황으로 인지할 수 있다.
- 네트워크 상 문제가 발생하는 것을 빨리 인지할 수 있어야 한다.
  - 타임아웃 값을 너무 loose 하게 설정할 경우 정말 필요한 상황에 장애 상황을 더욱 심각하게 만드는 요인이 될 수 있다.

> 따라서, '한 번의 패킷 손실 정도는 retransmit을 통해 해결할 수 있을 정도'의 값으로 설정하는 것이 적절하다.

### 적절한 Connection Timeout 값?

Connection Timeout은 TCP 연결 초기화, 즉 3-way handshake 과정에서 발생할 수 있는 타임아웃이다.

이 경우 발생할 수 있는 패킷 유실은
1. SYN 패킷
2. SYNACK 패킷
3. ACK 패킷

앞서 출처를 적어놨던 @Alden 님께서 잘 설명해주신 내용이다.

일반적으로 3가지 경우에서 각각 발생할 수 있는 패킷 유실 시나리오에 대해 제너럴하게 한 번의 retransmit을 용인하기 위한 Connection Timeout 값은 3초 정도라고 한다.

### 적절한 Read Timeout 값?

Read Timeout은 Connection Timeout보다 좀 더 짧게 세팅한다고 한다. 

Read Timeout을 Connection Timeout 처럼 3초라는 긴 시간으로 설정하게 된다면 클라이언트 - 서버 간 장애 상황이 발생하는 것을 감지하기 어려울 것이다. 

통상적으로 좋은 응답 시간은 200ms 언더로 알려져 있고, 그 이상을 넘어가게 될 경우 유저 경험에 안 좋은 영향을 미치는 것으로 알려져 있다. 

https://sematext.com/glossary/response-time/
 
하지만 elastic search 쿼리를 돌린다거나, 복잡한 SQL 프로시저 또는 트랜잭션을 돌리는 등의 행위는 필연적으로 시간이 오래 걸리는 작업들이 있다. 

결국 서비스하는 작업들의 유형에 따라 Read Timeout은 신중하게 설정되어야 한다고 생각한다. 
