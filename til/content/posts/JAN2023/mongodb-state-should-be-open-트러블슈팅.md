---
title: "Mongodb State Should Be Open 트러블슈팅"
date: 2023-01-22T22:14:22+09:00
draft: false
author: redjen
---

개발 중 JUnit 기반 테스트를 돌리는데 다음과 같은 에러를 마주쳤다.
`java.lang.IllegalStateException: state should be: open`

찾아보다가 좀 허무하게 해결했던 경험이지만 원인을 해결한 과정을 써보려 한다.

## 원인

원인은 테스트 클래스 의존성 주입이 잘못되어 몽고 DB 커넥션 연결 중 JUnit 테스트가 종료되며 끊어진 커넥션에 접근을 시도해서였다.

구글링을 해봐도 Timeout 이슈라고 하고, Mongo Client에 옵션을 수정해서 접속하라는 방법들이 나오길래 전혀 감을 못 잡고 있었다.

> MongoDB 커넥션 설정에 문제가 없다면, Nested한 에러 구문에 어떤 정보들이 같이 있는지 꼭 확인하자.
