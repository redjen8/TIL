---
title: "Mongodb의 16mb 악마"
date: 2023-03-29T21:31:52+09:00
draft: false
author: redjen
---

## 들어가며

현재 MongoDB의 한 Document 크기 제한은 16MB이다.

맨 처음 MongoDB가 출시된 2009년에는 4MB였고, 얼마 지나지 않아 그 제한이 16MB로 변경되었다.

많은 개발자들이 2023년에도 왜 MongoDB의 Document 크기가 16MB인지 궁금해한다. 

확실히 요즘의 어플리케이션에서는 대용량의 데이터를 다루는 시스템이 많아졌고, 단일 document 크기가 16MB 이상이 되었으면 좋겠다는 사람들이 늘어나고 있는 것 같다.

오늘은 MongoDB의 메모리 제한과, 그 이유와, 커뮤니티 전반에 걸친 논쟁을 가볍게 정리해보았다.

## MongoDB 쓰지 마세요

https://www.bloter.net/news/articleView.html?idxno=13677

무려 2012년, 이 글을 쓰는 현재 10년이 훌쩍 넘은 글이다. 꽤나 자극적인 아티클 제목이기에 예전에도 꽤 화제가 되었던 것으로 기억한다. 
(블로터 기자님이 쓴 글이 아님에 유의..!)

출시한지 3년 째, 혜성처럼 등장한 NoSQL 데이터베이스인 MongoDB는 혁신적인 특징으로 당시 RDBMS가 지배적이었던 데이터베이스 시장을 뒤흔들었지만 대신 몇 가지를 포기했었다.
- 실제로 꽤 최근까지 (2018년 이전) multi document 간 트랜잭션 지원이 되지 않았던 것이 그 예시이다.

기사에서 다루던 '**MongoDB를 쓰지 말아야 할 주요 이유**'로는 그 중에서도 불안한 데이터 정합성 이야기를 다루고 있었다.

![](https://velog.velcdn.com/images/redjen/post/f0941e07-984a-4274-aad6-b439ae45cca7/image.png)

3년 차의 MongoDB는 아직 미성숙한 솔루션이었으며, 내 데이터를 믿고 맡기기에는 너무나도 부족한 데이터베이스였으리라.

때문에 2012년의 비판 글을 다시 돌아보며 [이제는 MongoDB가 성숙했음을 이야기하는 글](https://itlab.co.kr/v7/?c=news&sort=d_regis&orderby=desc&where=subject%7Ctag&keyword=%EB%AA%BD%EA%B3%A0DB+4.2&uid=141320)도 다시 나왔다.

MongoDB는 이제 샤딩과 레플리케이션을 통해 안정적으로 데이터 정합성을 보장하고, 멀티 트랜잭션을 지원하면서 여전히 빠른 성능과 뛰어난 수평적 확장을 보장하는 매력적인 데이터베이스가 되었다.

데이터 정합성이 중요한 금융권에서도 MongoDB를 도입하기 시작했고, ETL과 같이 데이터에 복잡한 비즈니스 로직을 태우는 여러 빅테크들에서도 MongoDB를 사용하는 비율을 늘리고 있다. 

위의 기사에서도 언급했던, [Stackoverflow 선정 개발자들에게 가장 사랑받는 데이터 베이스 순위 2022](https://survey.stackoverflow.co/2022/)에서는 MongoDB가 당당히 4위를 차지하고 있다.

쟁쟁한 RDBMS 솔루션들을 뚫고 순수 NoSQL 데이터베이스로만 놓고 따지면 단연 1등이다. 그만큼 2023년 현재에도 NoSQL 진영에서 MongoDB는 독보적인 행보를 보인다.

## Embedded vs Reference

![](https://velog.velcdn.com/images/redjen/post/32c6dcdd-5cb5-44c4-b227-c1255434a7a7/image.webp)

MongoDB에는 `JOIN`이 존재하지 않는다. JOIN과 비슷한 `lookup`이 존재하지만, 이걸 사용할 바엔 나라면 그냥 MySQL이나 PostgreSQL을 쓴다. 

[lookup을 사용한다는 것은 무언가가 잘못되어가고 있다는 것](https://medium.com/@liams_o/lookup-in-mongodb-if-you-are-using-it-something-is-wrong-45d3fd47ac61)

`lookup`은 본질적으로 RDBMS의 Join과 내부 동작이 완전히 상이할 뿐만 아니라, 내용을 들여다보면 RDBMS의 Join보다는 데이터 레포지토리 레이어 레벨에서의 데이터 Aggregation 지원에 가깝다는 생각이다. 

심지어 동일하게 File IO를 수행하면서 수십년동안 성능 최적화를 위해 피똥싸며 고민한 결과물이 들어가 있는 RDBMS 진영의 Join과는 달리 별다른 최적화도 수행하지 않는다. 필연적으로 성능 면에서는 처참할 수 밖에 없다. 프로덕션 환경에서는 사용하지 않는 것이 일반적으로 권고되는 사항이다.

그럼 어떤 방식으로 데이터를 의미있게 분리하여 데이터베이스를 구성할 수 있을까? MongoDB에서는 두 가지 방식을 통해 이를 이룰 수 있다. 

### 1. Referenced Document

MyBatis를 사용하며 손수 한땀 한땀 Join 쿼리를 짜던 나 같은 사람에게는 이 방법이 오히려 편했다.

필드의 value로 `_id` 값을 저장해서, 그 `_id` 값으로 다시 read를 하는 방식으로 데이터를 풀어내는 방법이다.

MongoDB가 Join을 지원하지 않기 때문에 `SELECT`를 두 번이나 해야 한다니! 이게 무슨 낭비야! 라고 한다면 괜찮다. MongoDB의 `findOne` 몇 번은 비슷한 수의 Join 문이 들어간 단일 `SELECT` 쿼리와 비교하여 [성능이 그렇게 차이나지 않는다 (심지어는 더 빠르다)](https://blog.knoldus.com/mongodb-vs-rdbms-and-its-adavanatges/).

### 2. Embedded Document

반면 무식하게 child 관계의 데이터를 parent 데이터 스키마 구조에 넣어버리는 방법이 있다.

하지만 document의 크기가 커지게 되잖아요..! 괜찮다. MongoDB의 성능이 커진 데이터 크기를 어느 정도 보완해준다.
하지만 데이터 정규화는요..! 그런건 애초에 NoSQL에는 존재하지 않는 개념이다. 정규화의 이점을 버리고 확장성과 유연성을 가지자.

### 어떤 것을 사용해야 할까

https://betterprogramming.pub/embedded-vs-referenced-documents-in-mongodb-how-to-choose-correctly-for-increased-performance-d267769b8671

- 전체 parent document의 필요 없이 대용량 데이터를 write 해야 하는 상황이라면 referenced document를 사용
- 대부분의 데이터를 한 document에서 read 해야 하고, 그 곁다리로 관련 document가 존재한다면 embedded document를 사용
- 물론 두 방법을 동시에 사용할 수도 있다.

물론 그 이전에 데이터 스키마를 잘 구성한다면 둘 다 사용하지 않을 수도 있다. 진정한 MongoDB 고수라면 상황에 맞춰서 두 방법을 적절히 사용해야 한다.

> 소프트웨어 개발에서 모든 요구 사항들을 기분 좋게 해결하는 은탄환은 존재하지 않는다.

## MongoDB의 메모리 제한

https://www.mongodb.com/docs/manual/reference/limits/

MongoDB에서는 BSON Document를 사용한다. 그리고 **BSON Document의 최대 크기는 16MB이다.**

https://www.mongodb.com/json-and-bson

BSON은 Binary JSON이다. 그리고 BSON의 바이너리 구조는 타입과 길이 정보를 인코딩한다. 때문에 BSON은 단순히 캐릭터 문자 기반으로 저장되는 JSON과 비교하여 훨씬 더 빠르게 데이터를 처리할 수 있다는 장점이 있다.

BSON은 JSON 네이티브한 데이터 타입도 포함하여 지원한다. 날짜(`Date`)와 순수 이진 데이터(`BinData`)가 그 예시이다. MongoDB에서는 이러한 데이터 유형 추가를 통해 훨씬 더 다양한 데이터를 네이티브 단계에서 지원할 수 있도록 했다.

아래는 JSON 객체와 그에 상응하는 BSON 표현이다.

```bash
{"hello": "world"} → 
\x16\x00\x00\x00 // total document size 
\x02 // 0x02 = type String
hello\x00 // field name 
\x06\x00\x00\x00world\x00 // field value 
\x00 // 0x00 = type EOO ('end of object') 

{"BSON": ["awesome", 5.05, 1986]} → 
\x31\x00\x00\x00 
\x04BSON\x00 
\x26\x00\x00\x00 
\x02\x30\x00\x08\x00\x00\x00awesome\x00
\x01\x31\x00\x33\x33\x33\x33\x33\x33\x14\x40 
\x10\x32\x00\xc2\x07\x00\x00 \x00 
\x00
```

MongoDB는 데이터를 내부적으로 저장할 때나, 네트워크 상에서도 BSON 포맷을 사용한다. 하지만 이게 MongoDB를 JSON 데이터베이스로 생각할 수 없다는 것은 아니다.
- JSON으로 표현할 수 있는 모든 것은 MongoDB에 저장될 수 있다.
- MongoDB에 저장된 데이터는 쉽게 JSON으로 표현될 수 있다.

이미 어플리케이션에서 MongoDB 드라이버를 사용한다면 데이터를 BSON으로 변환하고, 쿼리의 결과를 다시 역직렬화한 객체로 만들어주는 역할을 수행하고 있을 것이다. 

JSON과 BSON의 상세 비교는 아래 표와 같다.

구분 | JSON | BSON
--- | --- | ---
인코딩 | UTF-8 String | 바이너리
데이터 지원 | String, Boolean, Number, Array, Object, Null | String, Boolean, Number (Integer, Float, Long, Decimal128 ...), Array, null, Date, BinData
가독성 | 사람과 기계가 읽을 수 있음 | 기계만 읽을 수 있음

## 왜 BSON의 크기를 하필 16MB으로 제한할까

서버의 메모리는 굉장히 비싼 리소스이다. MongoDB에서는 서버의 메모리를 효율적으로 사용하기 위해서 메모리 관리 정책을 지금처럼 설정한 것으로 보인다. 친절도 해라!

거의 15년 전이었던 그 당시와는 달리 상용 데이터베이스 서버의 메모리를 백 단위의 GB로 장착하는 것이 기본이 된 요즘에도 OS 메모리 사용량에 이슈가 있을지는 궁금하긴 하다.

그 밖에도 MongoDB가 표방하고자 하는 빠른 작업 속도를 보장하기 위해서는 확실히 한 Document의 최대 크기가 작은 편이 훨씬 도움이 된다. 애초에 읽어오는 파일의 크기가 커지게 되면 그만큼 느려지게 되는 것이니까.

그럼 이 제한을 뚫고, 굳이 MongoDB에서 한 Document에 16MB가 넘는 크기의 데이터를 넣겠다 하면 GridFS API를 사용하면 된다. 삽입하는 데이터를 청크 단위로 쪼개서 저장하고, 쿼리할 때에는 드라이버가 메타데이터를 참조하여 청크들을 다시 취합하여 반환한다. 

https://www.mongodb.com/docs/manual/core/gridfs/

- GridFS는 바이너리 청크를 저장하는 `chunks` 컬렉션과, 파일의 메타데이터를 저장하는 `files` 컬렉션을 사용한다.
- GridFS는 무려 인덱스도 지원한다! `chunks`와 `files` 컬렉션들에 대한 인덱스를 구성해서 read 성능을 향상시킬 수 있다.
- 하지만 너무 아쉽게도, 현재 Multi Document 트랜잭션은 지원하고 있지 않다.

## 왜 16MB로는 부족할까?

다루는 데이터의 종류가 점차 변화했기 때문이다. 

데이터 스키마 설계 시 앞서 설명했던 Embedded document 방식을 사용해서 주로 read에 사용될 데이터를 한 document에 넣어 편리하게 데이터를 다루는 경우가 많은데, 필연적으로 이는 Referenced document와는 달리 한 document의 크기를 크게 만든다. 그렇기 때문에 Embedded Document 방식은 MongoDB의 16MB 제한에 있어서는 불리한 면이 많다.

MongoDB Maintainer들은 공식적인 입장으로써 이는 어플리케이션의 데이터 스키마 설계가 잘못되었다는 태도를 보인다. 
이게 썩 유쾌하지 않다고 느껴지는 것과는 별개로, MongoDB 커뮤니티에 올라온 내용들을 보면 claimer들의 정말 데이터 스키마 설계가 잘못되었다고는 생각이 들지 않는 경우가 종종 있었다. (왜 이렇게 방어적인 태도를 취하는지는 사실 이해가 되지 않는다)

https://www.mongodb.com/community/forums/t/increase-max-document-size-to-at-least-64mb/124424

가령 커뮤니티에서 거론된 예시 중에는 
- 연산 등을 위해 미리 준비된 매우 큰 크기의 행렬을 영속적인 저장소에 저장한 후 메모리로 불러와 곧바로 사용하고 싶은 경우
  - 16MB 크기가 얼마나 작냐면, `16MB = 2^24(16,777,216) byte = 4 * (2^11) * (2^11)` 이다. 
  - 즉 2048 * 2048 크기의 정수 (4 byte) 행렬을 단순 저장만 해도 16MB이다!
- 움직이는 물체의 실시간 위치 정보를 저장하기 위해서 사용하는 케이스. 분석을 위해서는 주변 시간의 위치 데이터가 반드시 필요하지만 낮은 latency를 위해서 이를 embedded로 단일 document에 저장하고 싶은 경우
- 별개로 이미지와 같은 raw 데이터를 저장해서 16MB가 넘었다는 경우는.. 나도 데이터 스키마 설계가 잘못 되었다는 데에 동의한다.

비슷한 모든 경우에서 단순히 한 Document의 크기가 16MB를 넘는다는 이유로 다른 데이터 솔루션을 사용해야 한다면 개발자 입장에서는 그야말로 재앙이고, 짜증나는 일이 아닐 수 없다. 단지 16MB가 넘는 데이터를 한 document에 저장하지 못한다는 이유로 MongoDB의 유연성과 확장성을 포기하기에도 너무 아깝다. 

> 2023년, 앞으로 10년은 더 사용할지 모르는 내 어플리케이션에서 단일 데이터의 16MB 제한이 부족하지 않을 것이라고 보장할 수 있을까? 

> 또한 액티브하게 제공되고 있는 프로덕트의 데이터 크기가 점진적으로 커졌을 때, 저장하는 데이터의 크기 제한 때문에 데이터 솔루션을 변경해야 한다면 이 이유로 다른 사람들을 쉽게 납득시킬 수 있을까?

과거 2009년 [한 Document의 크기가 4MB에서 현재의 16MB로 증가되었을 때](https://jira.mongodb.org/browse/SERVER-431)에도 사람들은 16MB면 대부분의 데이터를 다 담을 수 있을 것이라고 생각했다... 
그리고 나는 그 예상이 가까운 시일 내에 깨질 것이라 확신한다.

## 맺음말

16MB가 단순 상수 값이 아니라, BSON 단위로 데이터를 저장하는 MongoDB의 특성 상 설정된 값이다 보니 쉽게 이 upper limit 값이 바뀔 것 같지는 않다.

심지어 이를 무시하고 큰 파일을 저장하기 위한 트릭성 추상화인 GridFS API라는 방법도 존재하는 상태이기 때문에 더 그런 생각이 든다.

물론 MongoDB 소스를 직접 수정하여 BSON 크기 제한을 64MB로 바꾼 후 컴파일하여 사용하는 방법도 있겠지만... 이렇게 사용할 개발자가 과연 얼마나 있을까?

솔루션을 사용하는 개발자의 입장으로 보았을 때에는.. 개발자가 툴에 맞추는 것이 아니라 툴을 개발자가 맞게 커스텀할 수 있어야 한다고 생각한다.

만약 16MB 제한이 풀리고 최대 64MB 크기의 document를 다루게 된 이후에도 비슷한 일이 생기지 않을 것이라는 보장을 할 수 있을까? 그렇기 때문에 최소한 설치 시점에 커스텀 가능한 상수로써 관리할 수 있도록 지원하면 참 좋을 것 같다. 대신 공식적으로 권고하면 될텐데. **16MB 이상으로 document 최대 크기를 설정할 때에는 사용 중인 서버의 리소스를 고려해야 합니다!** 등의 경고 문구도 넣고.

MongoDB의 커뮤니티에도 [예전부터 지속적인 건의](https://jira.mongodb.org/browse/SERVER-5923)는 이뤄지고 있지만, 여전히 Maintainer들의 입장은 쉽사리 변하지 않을 것 같다. 

관심이 있다면, 이 글을 읽고 관심이 생겼다면 [MongoDB 커뮤니티에 공감 투표](https://feedback.mongodb.com/forums/924280-database/suggestions/45455761-raise-maximum-bson-document-size-bigger-than-16-mb?_ga=2.88181919.524346435.1679972925-972986555.1672797943)해보는 것은 어떨까? 