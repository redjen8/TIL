---
title: "Spring Data Jpa의 Query생성"
date: 2023-01-13T18:21:15+09:00
draft: false
author: redjen
---

Spring data JPA에는 interface repository의 메서드로만 선언하면 메서드의 이름을 보고 해석하여 자동으로 쿼리를 생성해주는 편리한 기능이 있다.

현재 사용 중인 MongoRepository에서는 이런 Spring Data JPA의 편리한 기능이 존재해서 사용중에 찾아보게 되었다. 

Spring Data JPA에서는 어떻게 내부적으로 메서드의 이름만 보고 쿼리를 만들 수 있는 것일까?

https://github.com/spring-projects/spring-data-commons/tree/main/src/main/java/org/springframework/data/repository/query/parser

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation

이루고자 하는 바는 명확하다.
- 입력 : 메서드 이름
- 출력 : 그에 맞는 역할을 수행하는 쿼리

## Spring Data JPA의 파싱 개념

Spring Data JPA에서는 이를 subject와 predicate로 나누어서 파싱한다.

### Subject

find...By나 exists...By에 해당하는 메서드 이름의 첫번째 부분을 Subject라고 한다.
Subject는 말 그대로 쿼리의 subject, 주제를 정의한다. 
Subject는 추가적인 표현을 포함할 수 있다.
find와 By 사이에 존재하는 모든 텍스트들은 descriptive한 것으로 간주된다.
단, 여기에는 예외가 있는데: 
- Top / First와 같은 쿼리 결과를 제한하기 위한 키워드가 포함되어 있거나,
- Distinct 키워드를 통해서 distinct flag를 켜야 하는 경우
해당 키워드가 포함되어 있다면 descriptive한 것으로 간주하지 않는다.

### Predicate

subject 이외의 나머지 부분이 predicate이다.

## 애매한 코드에 대한 처리

Person 엔티티가 ZipCode와 관련한 Address를 가지고 있을 때,
```x.address.zipCode``` property를 찾는 것이 기본 동작이다. AddressZipCode를 전체 부분으로 도메인 클래스에서 찾다가 성공하면 그 property를 그대로 사용한다.

찾지 못한다면 (애매한 경우)
소스코드를 camel case 부분으로 쪼개서 오른쪽으로부터 파싱한다.
이 알고리즘은 AddressZipCode를 AddressZip이랑 Code로 분리한다.
여기서 찾으면 그것을 쓰는데, 또 찾지 못하면 다시 head - tail 분리 시행.
다음은 Address랑 ZipCode... 이런식으로 계속 찾는데

이런 ambiguity를 해소하기 위해서 메소드 이름에 ```_```를 넣어서 잘못된 속성을 선택하는 일을 미연에 방지할 수도 있다.
```List<Person> findByAddress_ZipCode(Zipcode zipCode)``` 와 같은 코드가 예시.

