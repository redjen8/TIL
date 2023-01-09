---
title: "Mongodb Update Operations in Java"
date: 2023-01-09T19:46:20+09:00
draft: false
author: redjen
---

Java에서 MongoDB를 다룰 때에는 여러 update 관련 operation들을 제공해준다.

각각의 operation 들에는 어떤 차이점이 있을까?

https://medium.com/geekculture/types-of-update-operations-in-mongodb-using-spring-boot-11d5d4ce88cf

## save()

save() 메서드는 criteria나 파라미터를 받지 않는다.
단지 ```_id``` 값으로 document 객체를 검색하는 것이 기본 동작이다.

document를 찾았다면, save() 메서드는 새로 document를 생성하는 대신 update한다.
save() 메서드는 새로 생성되거나 업데이트된 document 객체를 반환한다.

save() 메서드의 단점으로는, 항상 전체 document 객체를 입력 파라미터로 입력해야 한다는 것이다.
만약 20 필드 중에서 1개나 2개의 필드만을 업데이트하고 싶어도 (delta update) 할 수 없다.
이는 메모리 상 사소한 낭비를 일으킨다.

## saveAll()

save() 메서드와 두 가지를 제외하고는 정확히 똑같다.
1. document 객체 대신 document 객체들의 collection을 입력 파라미터로 받는다.
2. document 객체 대신 document 객체들의 collection을 리턴한다.

## upsert()

upsert() 메서드는 save() 메서드와 동일하다고 느껴진다.
- document가 존재한다면 업데이트하고,
- 존재하지 않는다면 새로운 document를 생성한다.

하지만 upsert()와 save() 메서드는 여러 차이점이 있다.
- upsert() 메서드는 ```_id``` 이외의 criteria를 통해 document를 검색할 수 있다.
- upsert() 메서드는 document 객체가 아닌 update 작업의 디테일에 대한 정보를 반환한다.
- upsert() 메서드는 업데이트 대상이 되는 디테일한 필드만 입력 파라미터로 입력할 수 있다.

이러한 방식은 다음과 같은 장점이 있다.
- delta update (여러 개의 필드 중 한 두 개만을 업데이트) 가 가능하다.

하지만 이런 단점도 있다.
- 해당 기준으로 검색했을 때 일치하는 document가 존재하지 않는다면 새 객체를 생성한다.
- 이는 데이터베이스에 불완전하거나 정합성이 떨어지는 데이터를 삽입할 잠재적 가능성이 있다.

## findAndModify()

findAndModify() 메서드는 save()와 upsert() 메서드의 장점을 섞은 메서드이다.

upsert() 메서드처럼,
- ```_id``` 이외의 criteria를 통해 document 객체를 검색할 수 있다.
- delta update가 가능하다.

save() 메서드처럼,
- 전체 document 객체를 반환한다.
- 업데이트 이전의 document 객체를 가져오거나, 새로이 업데이트된 document 객체를 가져올 수 있는 옵션을 제공한다.

findAndModify()는 ```upsert(boolean)``` 옵션을 통해서 조건에 만족하는 document가 존재하지 않을 경우 새 document를 생성할지에 대한 여부를 지정할 수 있다. 

## findAndReplace()

findAndReplace를 사용하면 어떤 필드로도 document를 검색할 수 있다.
document를 찾았다면, 요청에 사용된 새 document로 교체한다.

```upsert()``` 옵션을 지정하여 일치하는 document가 없을 때의 동작을 정의할 수 있다.
```returnNew()``` 옵션을 지정하여 업데이트된 document를 리턴 받을지 말지를 정의할 수 있다.

전체 document를 입력 파라미터로 입력하기 때문에 findAndReplace()를 통한 delta update는 불가능하다. 

## updateFirst()

updateFirst()는 ```_id``` 이외의 필드를 통해 업데이트 대상이 되는 document를 정할 수 있다.
또한 upsert 처럼
- delta update가 가능하다.
- 업데이트가 일어났을 때 업데이트에 관련한 정보를 리턴한다.

하지만 조건에 일치하는 document가 없을 경우, 새 document를 생성하지는 않는다.

updateFirst()와 다른 delta update를 지원하는 작업들의 유일한 차이점은 updateFirst()는 update 조건을 만족하는 가장 처음의 문서만 업데이트한다는 점이다. 나머지 작업들은 모든 document를 업데이트한다. 

## updateMulti()

updateFirst()와 유사하지만, updateFirst()와는 달리 update 조건을 만족하는 모든 document를 업데이트한다.

## UpdateOne()을 사용한 BulkOps

여러 쿼리들을 하나 이상의 입력 파라미터롤 받는다.
또한, 검색 조건과 업데이트 작업 정의를 한 쌍으로 묶어서 제공해줘야 한다.

save()나 findAndReplace()를 사용하여 데이터를 대량으로 업데이트할 때와는 달리 전체 document를 input으로 받지는 않는다.

각각의 쿼리를 만족하는 첫번째 document가 업데이트된다.

조건을 만족하는 document가 없을 경우, 업데이트 작업 정의에 명세된 정보를 가지고 새 document를 만들지는 않는다.
모든 쿼리들에 대한 업데이트 작업은 상호 배제적이다. (단일 document가 여러 쿼리를 만족해야 할 필요는 없다)

updateFirst()와 유사하게 UpdateOne()을 사용한 대량의 업데이트 작업은 업데이트 작업에 관련한 정보들을 리턴한다.

## UpdateMulti()를 사용한 BulkOps

UpdateOne()을 사용한 대량 업데이트 작업과 거의 유사하지만, 
업데이트 조건을 만족하는 첫번째 document를 업데이트하는 대신 조건을 만족하는 모든 document들을 업데이트한다는 점이 다르다.