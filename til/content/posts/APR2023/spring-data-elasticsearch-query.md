---
title: "Spring Data Elasticsearch Query"
date: 2023-04-14T19:01:07+09:00
draft: false
author: redjen
---

spring data elasticsearch에서, 보다 복잡한 쿼리를 객체지향적으로 작성하고 보내보자.

https://www.baeldung.com/spring-data-elasticsearch-queries

## Analyzer

모든 저장된 string 필드들은 기본적으로 analyzer에 의해 처리된다.

- Analyzer는 하나의 tokenizer와 다수의 토큰 필터들로 구성된다.
- Analyzer는 보통 하나 이상의 문자 필터가 먼저 위치하게 된다.

기본 analyzer는 문자열을 common word separator (공백이나 punctuation 등)으로 분리하고 그렇게 해서 생성된 모든 토큰을 lowercase로 변환한다. 

엘라스틱서치는 analyzer가 어떤 필드를 처리할지에 대한 설정을 할 수 있다. 

```java
@MultiField(
  mainField = @Field(type = Text, fielddata = true),
  otherFields = {
      @InnerField(suffix = "verbatim", type = Keyword)
  }
)
private String title;
```

상기 예시에서 `title` 필드는 전형적인 analyzed field이다. suffix로 `verbatim`을 가지는 필드들은 analyze 되지 않는다.

`@MultiField` 어노테이션은 Spring Data에게 이 필드가 여러 방법으로 인덱스가 될 가능성이 있다고 이야기 해주는 것이다. 메인 필드는 title을 name으로 사용할 것이고, 상기한 규칙에 의거하여 analyze 될 것이다.

`@InnerField` 어노테이션은 title 필드의 부가적인 인덱싱을 기술한다.
- `FieldType.keyword`를 사용해서 필드에 대한 부가적인 인덱싱을 수행할 때 analyzer를 사용하지 말라고 지시할 수 있다.
- 이 값은 nested field 안쪽의 'vertatim' suffix로 존재해야 한다.

### Analyzed Fields

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "elasticsearch data"))
  .build();
```

"Spring Data Elasticsearch" 가 인덱스에 추가된다면, spring, data, elasticsearch로 토큰으로 쪼개져서 analyzer가 분석한다. 상기 코드를 통해 해당 document를 검색할 수 있다.

### Non analyzed Fields

Analyze 되지 않는 필드는 토큰화도 되지 않기 때문에 전체에 대한 `match`나 `term` 쿼리를 통해서만 매칭된다.

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title.verbatim", "Second Article About Elasticsearch"))
  .build();
```

match 쿼리를 통해 full title을 통해서만 검색할 수 있다. 이 경우에는 대소문자도 구분한다.

## Match Query

Match query는 text, number, date를 허용한다. 아래 3가지 타입이 존재한다.
- boolean
- phrase
- phrase_prefix

boolean match query에 대해서만 정리해보자면,,

### Matching with Boolean Operators

boolean은 match query의 가장 기본 타입이다. 
```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title","Search engines").operator(Operator.AND))
  .build();
SearchHits<Article> articles = elasticsearchTemplate()
  .search(searchQuery, Article.class, IndexCoordinates.of("blog"));
```

상기 쿼리는 title로 "Search engines"를 가지는 아티클을 리턴한다. 위 예시에서 `and`를 기본 연산자인 `or`로 변경하면 어떻게 될까?

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "Engines Solutions"))
  .build();
SearchHits<Article> articles = elasticsearchTemplate()
  .search(searchQuery, Article.class, IndexCoordinates.of("blog"));
assertEquals(1, articles.getTotalHits());
assertEquals("Search engines", articles.getSearchHit(0).getContent().getTitle());
```

"Search engines" 아티클은 여전히 매칭된지만, 모든 term들이 매칭된 것이 아니기 때문에 더 낮은 스코어를 가지게 된다.

각각의 매칭되는 term에 대한 스코어의 합산이 결국 결과로 리턴되는 도큐먼트의 총합 스코어가 된다.

특정 도큐먼트에서 포함하고 있는 희귀한 term이 쿼리에 들어간다면 다른 평범한 term을 포함하는 다른 도큐먼트보다 찾고자 하는 도큐먼트의 스코어가 확연하게 높을 것이다.

### Fuzziness

사용자가 입력한 문자열에 오타가 있을 수 있으므로, 엘라스틱서치에서는 이런 것도 감안하여 쿼리를 태울 수 있게 한다.

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "spring date elasticsearch")
  .operator(Operator.AND)
  .fuzziness(Fuzziness.ONE)
  .prefixLength(3))
  .build();
```

`fuzziness`는 편집 거리(하나의 단일 문자에 대한 변화가 다른 문자열로의 변화가 생기는 거리)이다.

`prefix_length` 파라미터는 성능을 향상시키기 위함이다. 상기 예제에서는 처음 3개의 문자가 정확하게 매칭되어야 한다고 명시하는 것이다. 이런 접근은 가능한 조합의 개수를 확실히 줄이기 때문에 성능 향상에 도움이 된다.

## Phrase search

Phrase search는 더 엄격하다.

`slop` 파라미터는 phrase query에게 매칭되는 도큐먼트를 고려하면서도 얼마나 term이 관계없어질 수 있는지를 지정해준다.

즉 쿼리와 도큐먼트가 서로 매칭되기 위해서 term을 얼마나 많이 이동해야 하는지에 대한 정보이다.
```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchPhraseQuery("title", "spring elasticsearch").slop(1))
  .build();
```


## Multi Match Query

다수의 필드에 대해 검색을 수행하고 싶을 때에는 `QueryBuilders#multiMatchQuery()`를 사용할 수 있다.
```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(multiMatchQuery("tutorial")
    .field("title")
    .field("tags")
    .type(MultiMatchQueryBuilder.Type.BEST_FIELDS))
  .build();
```

엘라스틱서치는 다큐먼트 중에서 가장 스코어가 높은 필드를 문서의 스코어로 사용한다.


