---
title: "Mongodb Transaction 개념과 역사"
date: 2023-01-07T00:23:12+09:00
draft: false
author: redjen
---

MongoDB에는 원래 Transaction 개념이 없었다. (4.0 버전부터 트랜잭션을 지원하기 시작했다.)

**그렇다면 왜 없었으며, 무슨 바람이 들어서 최근에서야 생기게 된 것일까?**

## 4.0 버전 이전의 MongoDB

https://stackoverflow.com/questions/18042255/why-doesnt-mongodb-have-transactions-pre-v4-0

> MongoDB가 처음 탄생했을 때에는 트랜잭션 개념이 MongoDB를 확장 가능한 구조를 허용하는데 제약을 주었기 때문이다.

트랜잭션은 여러 작업이 이뤄지는 동안 전체 데이터베이스를 일관성 있게 유지하는 것을 목적으로 한다. 전통적인 RDB와는 다르게, MongoDB에서는 Standalone 형태로 구동되는 것을 목적으로 하지 않는다. MongoDB는 다수의 샤드를 가진 분산 클러스터 서버 형태로 구성되는 것이 이상적이다. (각 샤드는 다수 서버의 Replica Set 구조를 유지한다.)

트랜잭션은 분산 데이터베이스에서 잠재적으로 큰 영향을 끼친다. 분산되어 있는 클러스터에서 트랜잭션을 보장하기 위해서는 모든 클러스터들이 동기화 되어야 하고 이는 꽤 많은 오버헤드와 비용을 요구할 뿐만 아니라 데이터베이스가 수평 확장될 때 상황을 훨씬 악화시킨다.

때문에 태어났을 때부터 분산 클러스터였던 MongoDB가 트랜잭션을 지원하지 않는 것은 어떻게 보면 당연하게 느껴진다.

## 4.0 버전 이상의 MongoDB

최신 버전의 (4.0 버전 이상의) MongoDB는 여러 document에 대한 R/W 연산의 원자성을 필요로 하는 상황에서 트랜잭션을 지원한다.
이 분산 트랜잭션을 사용하면 여러 작업, 컬렉션, 데이터베이스, document와 샤드에 걸쳐 트랜잭션을 사용할 수 있게 된다. 

그럼 왜 최근 버전의 MongoDB에서는 트랜잭션을 지원하게 된 것일까?
앞서 기술한 것처럼 트랜잭션은 분산 기반 데이터베이스에서 지원하기에는 너무 비싼 작업인데..

https://www.mongodb.com/docs/manual/core/transactions/

4.0 버전 미만에서도 MongoDB의 단일 Document에 대한 모든 연산은 atomic했다. 
데이터베이스를 RDB처럼 정규화해서 사용하는 것 대신 MongoDB에서는 Embedded document와 Array를 적극적으로 사용해서 document 안에서 데이터 간의 관계를 표현한다.

> 그런데 MongoDB가 지원하는 이 단일 document에 대한 atomic한 연산은 MongoDB가 데이터를 정규화해서 사용하지 않기 때문에 실제로는 많이 사용하지 않게 되었다.

실무에서는 훨씬 복잡한 비즈니스 요구사항을 구현하기 위해서 단일 document가 아닌 훨씬 다양한 document의 데이터를 접근하고 저장해야 할 필요가 생겼다.

그래서 많은 개발자들이 MongoDB를 사용하면서 document가 여러 컬렉션에 나눠서 저장되면 편할 것 같다고 생각했고, 이렇게 된다면 기존 4.0 미만의 버전의 MongoDB에서는 복잡한 비즈니스 로직을 여러 document에 나눠 구현할 때 작업의 원자성이 보장되지 않기 때문에 곤란한 경우가 생겼을 것으로 추측된다.

때문에 MongoDB는 WiredTiger 스토리지 엔진을 사용하면서 RDB와 비슷한 (똑같지는 않다!) ACID 개념을 지원하게 되었다. 다만 MongoDB의 ACID 개념은 RDB의 그것과는 약간 다르다.

https://rastalion.me/mongodb-transaction-management/

- 최고 레벨의 isolation은 REPEATABLE_READ로 구성
- 트랜잭션의 Commit과 Checkpoint을 통해 영속성을 보장
- Commit되지 않은 변경 데이터는 공유 캐시 크기보다 작아야 함

또, MongoDB에서 트랜잭션 구현은 세션을 통해 이뤄지기 때문에 WriteConflict 상황이 발생하긴 한다. 여러 사용자가 동시에 동일한 document를 수정하는 일종의 race condition이다.

MongoDB는 변경하려는 document가 이미 다른 connection에 의해 lock이 걸려 있다면 바로 업데이트 작업을 취소하고 WriteConflictException 에러를 띄운다.

업데이트 시도한 세션은 해당 에러를 받았을 때 MongoDB의 스토리지 엔진 레벨에서 동일한 업데이트 명령을 재시도하기 때문에 어플리케이션 사용자는 이 Conflict 상황을 모르는 것이 특징이다.

결론적으로는 4.0 버전 이상의 MongoDB 부터는 단일 document가 아닌 다수 document에 대한 트랜잭션만을 지원하게 되었다.
