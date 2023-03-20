---
title: "Sharding vs Replication"
date: 2023-03-20T22:06:30+09:00
draft: false
author: redjen
---

MongoDB에서 sharding과 replication은 어떻게 다를까?

데이터를 뭔가 여러 곳에 저장하는 것 같긴 한데.. 구체적으로 어떻게 다른지 알아보자.

https://dba.stackexchange.com/questions/52632/difference-between-sharding-and-replication-on-mongodb

## ReplicaSet

ReplicaSet은 MongoDB 인스턴스를 단순히 여러 개 가진다는 것을 의미한다.
- 그리고 각각의 인스턴스는 서로를 미러링한다.

ReplicaSet 구성에서는 Primary와, 하나 이상의 Secondary로 이루어진다.

모든 쓰기 연산은 Primary를 통해서 이루어지고, primary에 새로이 써진 데이터는 secondary로 복제된다. 때문에 ReplicaSet 구성에서는 쓰기 연산의 속도가 향상되지는 않는다.

반면 모든 Secondary는 읽기 연산을 수행할 수 있다. 그렇기 때문에 다음의 특징을 갖는다.
- 읽기 연산의 횟수가 증가하면 ReplicaSet의 secondary 수를 증가시켜서 읽기 성능을 향상 시킬 수 ㅣㅆ다.
- ReplicaSet의 서로 다른 멤버들이 요청을 나눠서 처리하는 컨셉이다.

ReplicaSet은 또한 Fault Tolerance 기능도 가진다.
- 만약 ReplicaSet의 한 멤버가 다운되었을 때에는 다른 멤버가 그 역할을 대신 수행한다.
- 만약 Primary가 다운되었다면 Secondary 중에서 새로운 primary를 선출한다.

> 그렇기 때문에 프로덕션 환경에서 ReplicaSet을 사용한다면 반드시 3개 이상의 서버를 사용하도록 권장된다. 

## Sharded Cluster

Sharded cluster 구성은 클러스터의 각 샤드가 데이터의 한 부분을 담당하는 구조이다.

그리고 읽기 / 쓰기 각각의 요청은 데이터가 들어있는 각 클러스터에 전달된다. 이는 읽기와 쓰기 연산의 성능이 둘 다 샤드 개수의 추가로 향상될 수 있다는 것을 뜻한다.

어떤 샤드에 어떤 다큐먼트가 들어가게 되는지는 각각의 컬렉션의 샤드 키에 의해 결정된다. 샤드키는 모든 클러스터에 데이터를 골고루 적재하기 위한 방법으로 선택된다. 

보통은 자주 질의하게 되는 쿼리의 내용으로 샤드 키를 선택했을 때 데이터가 골고루 분산될 수 있도록 염두하고 샤드키를 선택하는 것이 보편적이다.

Sharded cluster의 단점으로는 클러스터의 각 샤드가 데이터의 조각만 가지고 있다는 점이다.

클러스터의 한 샤드가 다운되었을 때에는 모든 데이터가 접근 불가능한 상태가 된다.
때문에 클러스터의 각 멤버는 ReplicaSet이어야 한다. 만약 HA를 고려하지 않는다면 샤드는 복제 설정이 되어 있지 않은 단일 mongod 인스턴스가 된다. 하지만 이는 절대로 프로덕션 환경에서 사용되어서는 안된다.

> 프로덕션 환경에서는 반드시 replication을 사용해야 한다.

## 예시

만약 75GB의 데이터를 25GB 샤드 3개로 나누고 싶다면,
- 최소 6개의 데이터베이스 서버가 필요하다.
- 각각의 샤드는 하나의 primary + 하나의 secondary로 구성된 ReplicaSet이어야 한다.
- primary와 secondary는 모두 25GB의 데이터를 가진다.
- 이러한 구성을 사용할 경우 필요한 용량은 총 `25 * 6 = 150GB`이다.


