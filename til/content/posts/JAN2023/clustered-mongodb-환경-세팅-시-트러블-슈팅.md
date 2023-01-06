---
title: "Clustered Mongodb 환경 세팅 시 트러블 슈팅"
date: 2023-01-06T15:49:16+09:00
draft: false
author: redjen
---

## 문제 상황

MongoDB 기존 스키마에서 변경되어야 할 부분이 있어 (스키마 개념은 따로 없지만 새 필드를 추가하는 의미에서 워딩을 이렇게 가져갔다) 개발 서버에서 테스트하기 전 로컬에서 먼저 테스트하는 과정에 생긴 일..

1. ReplicaSet으로 구성된 개발 DB를 ```mongodump```로 로컬로 저장
2. ```mongorestore```로 저장된 bson 파일들을 로컬 환경에서 데이터베이스로 생성
3. 어플리케이션 시작

..! 분명 mongodb compass와 ```mongosh``` 접속은 되는데, 어플리케이션에서 로컬 DB 접속 시도할 때 아래와 같은 메시지를 볼 수 있었다. 

```bash
org.springframework.dao.DataAccessResourceFailureException: 
Timed out after 2000 ms while waiting for a server that matches WritableServerSelector. 
Client view of cluster state is {type=REPLICA_SET, servers = []... }
```

구글링을 1시간 넘게 해봐도 별다른 해결 방법이 나오지 않는 듯하여 팀원 분께 도움을 청했다.

## ClusterType 변경

해답은 어플리케이션의 설정 중, 로컬에 적용되는 MongoDB의 ClusterType 설정이 REPLICA_SET으로 되어 있었다.

STANDALONE 타입으로 ClusterType을 변경하여 재시작하니, 정상적으로 MongoDB에 접속할 수 있었다.

그런데, mongodb에서는 STANDALONE Cluster Type에서 트랜잭션을 지원하지 않는다.
이유를 찾아보니 [ Retryable Writes](https://www.mongodb.com/docs/manual/core/retryable-writes/) 때문이라고 한다.
MongoDB에서는 트랜잭션을 보장하기 위해 네트워크 에러가 발생하면 replica set이나 sharded cluster에 retryable write를 시도한다. 

https://dba.stackexchange.com/questions/265236/how-can-we-use-transaction-in-mongodb-standalone-connection

트랜잭션을 사용하는 기존 코드가 있기 때문에, 로컬에 설치된 STANDALONE MongoDB를 Single Node ReplicaSet cluster type으로 변경하기로 했다. 

방법은 [공식 문서](https://www.mongodb.com/docs/manual/tutorial/convert-standalone-to-replica-set/)를 참고했다.
나는 homebrew로 MongoDB를 설치했기 때문에, 기본 config 설정이 ```/opt/homebrew/etc/mongod.conf```에 위치해 있었다. replication 설정을 마치고 ```application.yml```에 mongodb url 설정에 replicaSet 명시를 param으로 변경해주었다. 이제 될까?

## ReplicaSet 설정

그런데 다시 어플리케이션을 구동하니 DB의 ClusterType이 REPLICA_SET_GHOST이기 때문에 접속에 실패했다는 메시지가 또 떴다. 

찾아보니 REPLICA_SET_GHOST는 host들의 목록이나 set 이름이 선언되지 않은 replica set 멤버 타입이었다. https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/connection/ServerType.html

어플리케이션을 구동하기 전에 ```mongosh```를 켜서 rs.initate()를 해줬다. 다시 어플리케이션을 켜니 정상적으로 API 호출 시 기존 만들어놨던 트랜잭션 코드들이 정상적으로 실행되는 것을 볼 수 있었다. 

## 3줄 요약

1. MongoDB는 STANDALONE 타입에서 트랜잭션을 지원하지 않는다.
2. ReplicaSet 설정하여 DB 구동 시 Replica들의 host 목록과 set 이름을 선언하여야 한다.
3. 분산 DB를 제대로 잘 활용하려면 제대로 잘 알고 사용해야 고생을 덜 한다. 
