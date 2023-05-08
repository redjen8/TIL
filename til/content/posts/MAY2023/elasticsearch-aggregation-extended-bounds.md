---
title: "Elasticsearch Aggregation의 extended_bounds"
date: 2023-05-08T21:59:09+09:00
draft: false
author: redjen
---

ES의 aggregation은 기본적으로 aggregation 결과 중 데이터 버킷 count가 0이 아닌 녀석들만 리턴해준다.

그런데 만약 차트 등을 그리기 위해서 값이 0인 구간도 필요한 경우가 있다면 어떻게 해야 할까?

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-histogram-aggregation.html#search-aggregations-bucket-histogram-aggregation-extended-bounds

## extended bound 속성

> 기본 histogram은 데이터 자신의 범위 안에 있는 모든 버킷들을 리턴한다. 

즉 가장 작은 값을 가진 다큐먼트들이 min bucket으로 결정되고, 
가장 큰 값을 가지는 다큐먼트가 max bucket을 결정된다.

하지만 빈 버킷을 요청할 때에는 이런 기본 동작이 혼란을 줄 수 있으며 특히 필터링을 거친 데이터라면 더더욱 그렇다.

### 예시

아래의 상황을 가정해보자.
1. 필터링을 통해 0부터 500 사이의 값을 가진 모든 다큐먼트들을 요청.
2. 데이터를 50의 간격으로 슬라이스한 히스토그램으로 반환 받는다.
3. 빈 버킷 또한 리턴받기 위해 `min_doc_count`를 0으로 설정했다.

이 경우에 만약 모든 다큐먼트의 값이 100보다 크다면, 처음 리턴받는 데이터는 값 100과 함께 그 버킷의 키를 리턴받게 될 것이다. 엄청 헷갈린다!

### 해결 방법

`extended_bounds` 설정을 통해서 히스토그램 aggregation에게 구체적인 최솟값과 최댓값에 대해 버킷을 만들어내도록 명령할 수 있다. 

당연하게도 `extended_bounds` 속성은 `min_doc_count`가 0일 경우에만 말이 된다. (`min_doc_count`가 0보다 크다면 빈 버킷은 절대 리턴되지 않기 때문이다)

하지만 `extended_bounds` 속성은 **버킷을 필터링하는 것이 아님**에 유의해야 한다.

즉 `extended_buckets.min` 값이 다큐먼트로부터 뽑아내어진 값보다 크다면 
- 다큐먼트는 첫 번째 버킷에 대해서 리턴하고자 하는 것을 그대로 리턴한다.
- `max` 값과 마지막 버킷에 대해서도 동일한 동작을 수행한다.

때문에 '버킷을 필터링'하기 위해서는 히스토그램 aggregation을 `filter` aggregation 아래의 sub aggregation으로 추가해야 한다.

```bash
curl -X POST "localhost:9200/sales/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "constant_score": { "filter": { "range": { "price": { "to": "500" } } } }
  },
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
'
```

aggregation 범위에 대해서 버킷들은 항상 리턴된 다큐먼트의 값에 기반한다. 즉 응답은 쿼리 범위 바깥의 버킷을 항상 포함할 수 있다.

예를 들어 100보다 큰 값에 대해 찾는 쿼리를 수행하기 위해 50~150까지의 범위, 50의 인터벌을 지정했다면 다큐먼트는 50, 100, 150 총 3개의 버킷에 나누어 들어가게 된다.

하지만 웬만하면 **쿼리와 aggregation step은 별개로 생각하는 것이 좋다**.

쿼리는 다큐먼트의 집합을 선택하고, aggregation은 그 다큐먼트들을 어떻게 선택되었는지와는 무관하게 버킷으로 만든다.

## hard_bounds

`hard_bounds` 속성은 `extended_bounds` 속성의 정반대에 있는 속성이다.

때문에 `hard_bounds` 속성은 히스토그램의 버킷 범위를 지정해버릴 수가 있다.

때문에 아주 큰 크기의 버킷을 얻기 위한 open 범위에 대한 질의에 유용하다.

```bash
curl -X POST "localhost:9200/sales/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "constant_score": { "filter": { "range": { "price": { "to": "500" } } } }
  },
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "hard_bounds": {
          "min": 100,
          "max": 200
        }
      }
    }
  }
}
'
```

상기 예제에서 범위가 500까지 지정되었음에도 불구하고 (`to: 500`) 히스토그램은 100과 150, 딱 두 개의 버킷만 가지게 된다. 

해당 버킷으로 이동할 다큐먼트가 결과에 있더라도 다른 모든 버킷들은 생략된다.