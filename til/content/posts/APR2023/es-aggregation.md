---
title: "Es Aggregation"
date: 2023-04-13T19:20:04+09:00
draft: false
author: redjen
---

Elasticsearch 에서는 aggregation 연산을 지원한다.

https://esbook.kimjmin.net/08-aggregations

- 이 기능은 `_search` API에서 `aggregation` 또는 `aggs`를 명시하고
- 임의의 aggregation 이름을 입력한 뒤
- 사용할 aggregation 종류와 옵션을 넣는다.

## aggregation의 구성

Metrics와 Bucket으로 나뉜다.

Aggregations에 어떤 것은 metric, 어떤 건 bucket으로 나뉜다고 명시를 하지는 않지만

- Aggregation 종류 중 숫자 / 날짜 필드 값을 가지고 연산하는 것들 = metrics
- Aggregation 종류 중 범위나 키워드 값을 가지고 도큐먼트들을 그룹화하는 것들 = bucket

## 버킷 구성

### range

- `field`: 해당 필드의 이름 지정
- `ranges`:
	- `from`: 범위 시작 지정
	- `to`: 범위 끝 지정

### histogram

range와 달리 interval 옵션을 지정해서 고정된 간격 크기 대로 버킷을 구성한다.
- `field`: 해당 필드의 이름 지정
- `histogram`: 범위를 구분할 간격 지정

### date_range, date_histogram

range, histogram과 동일하다. 그런데 `date` 타입을 끼얹은..

### terms

키워드 필드의 문자열 별로 버킷을 나누어 집계한다.
- 필드 문자열을 기준으로 버킷을 나눌수도 있고
- 가져올 버킷의 개수를 선택하는 `size`  옵션을 지정할 수 있다. (default 10)

인덱스의 특정 키워드 필드에 있는 모든 값들을 대상으로 버킷을 만들면 결과 셋이 너무 커직 ㅣ때문에
 - 먼저 도큐먼트 개수 / metrics 연산 결과가 가장 많은 버킷을 샤드 별로 계산
 - 상위 몇 개의 버킷만 coordinate 노드로 가져오고
 - 다시 취합해서 결과 도출

이러한 과정은 검색의 query / fetch 과정과 유사하다.