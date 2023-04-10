---
title: "Swagger Discriminator"
date: 2023-04-10T19:58:30+09:00
draft: false
author: redjen
---

이전에는 `oneOf`, `anyOf`, `allOf` 등의 키워드를 통해서 어떻게 request 나 response에 해당하는 객체의 validation을 할지 간략히 알아봤었다.

사실 이전에 소개한 기능들은 `discriminator`와 함께 사용하면 훨씬 강력하다.

https://swagger.io/docs/specification/data-models/inheritance-and-polymorphism/

## 다형성을 API 문서와 스펙에 반영하려면..

API를 제공하는 입장에서는 특정 필드의 값으로 스키마의 종류를 분기하여 조금 더 유연한 API 설계를 하고 싶다.

그럴 경우에 사용하는 것이 `discriminator`이다.

https://github.com/OAI/OpenAPI-Specification/blob/3.0.1/versions/3.0.1.md#discriminatorObject

OAS3 스펙에서는 `discriminator`를 아래와 같이 정의한다.
> request body 또는 response 페이로드가 여러 스키마 중 하나일 수 있는 경우 discriminator 객체를 사용하여 직렬화, 역직렬화, 유효성 검사를 지원할 수 있다.

> discriminator는 스키마의 특정 객체로, 스키마와 연관된 값을 기반으로 대체 스키마의 사양을 API 소비자에게 알리는데 사용된다.

### discriminator의 구성

discriminator는 크게 두 필드를 가진다.

1. `propertyName` : (필수) 페이로드에 포함된, discriminator 값으로 작용할 속성의 이름이다. 
2. `mapping` : 페이로드의 값과 스키마 레퍼런스 간의 매핑을 풀기 위한 객체 정보를 담는 `Map<string, string>`이다.

`discriminator`는 `oneOf`, `anyOf`, `allOf` 와 같은 합성 키워드 중 하나와 같이 사용했을 때에만 유효하다.

### springdoc에서의 설정 방법

Interface 객체가 있고, 그 객체를 implement 하는 서로 다른 두 객체가 있다고 하자.

다음과 같이 구성하여 해당 필드를 구분지을 수 있다.

```java
@Schema(  
    type = "object",  
    title = "베이스 스키마",  
    discriminatorProperty = "discriminatorValue",  
    discriminatorMapping = {  
        @DiscriminatorMapping(value = "FIRST", schema = FirstTestDto.class),  
        @DiscriminatorMapping(value = "SECOND", schema = SecondTestDto.class)  
    })
```
만약 `discriminatorValue` 값이 `FIRST` 이면 `FirstTestDto`로 매핑한다.
마찬가지로 해당 값이 `SECOND`이면 `SecondTestDto` 로 매핑한다.