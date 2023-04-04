---
title: "Swagger oneOf, anyOf, allOf"
date: 2023-04-04T21:57:30+09:00
draft: false
author: redjen
---

API를 설계하고, 이를 API 문서로 만들다 보면 API의 응답을 상태에 따라 분기해서 여러 종류로 나눠주고 싶은 경우가 있다.

예를 들어 (절대 좋은 설계라는 말과는 무관하다)
- 사용자가 가진 권한으로 이 API를 요청했을 때에는 1~3까지에 해당하는 데이터만 주고,
- 더 높은 권한으로 동일한 API를 요청했을 때에는 1~5까지에 해당하는 데이터를 주고 싶을 때가 있다.
- 데이터가 같은 형태라면.. json list로 묶어서 응답으로 주면 되겠지만
- 데이터가 다른 형태라면? API 문서로 나타내기 매우 골치 아파진다. 

결국 같은 API에 대해 다른 응답 값을 나타날 때 분기를 어떻게 할 것인가?

대표적인 API 문서 도구 Swagger는 이런 경우를 지원하기 위해 여러 키워드를 지원한다.

https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/

## oneOf

`oneOf` 키워드를 사용하면 특정한 스키마들 중에서 주어진 데이터가 유효한지 알 수 있다.

```yaml
paths:
  /pets:
    patch:
      requestBody:
        content:
          application/json:
            schema:
              oneOf:
                - $ref: '#/components/schemas/Cat'
                - $ref: '#/components/schemas/Dog'
      responses:
        '200':
          description: Updated
components:
  schemas:
    Dog:
      type: object
      properties:
        bark:
          type: boolean
        breed:
          type: string
          enum: [Dingo, Husky, Retriever, Shepherd]
    Cat:
      type: object
      properties:
        hunts:
          type: boolean
        age:
          type: integer
```

상기 예제는 PATCH 요청에 들어가야 할 request body를 어떻게 검증할 지를 제시한다. 이러한 형태는 request body가 업데이트 대상이 되는 객체가 필요한 모든 정보를 가지고 있는지 검사한다.

서론에서 이야기 했던 것처럼, 객체가 달라질 경우에는 request body에 대한 검증 또한 달라져야 한다.

request body에 두 개의 객체에 대한 검증을 합쳐 놓으면 원하는 결과가 나오기 어렵기 때문에, `oneOf`를 사용해야 한다.

```json
{
  "bark": true,
  "breed": "Dingo" 
}
```
예를 들어, 위 request body는 유효하다.

```json
{
  "bark": true,
  "hunts": true
}
```
반면 위 request body는 유효하지 않다.

```json
{
  "bark": true,
  "hunts": true,
  "breed": "Husky",
  "age": 3 		
}
```
이건 어떨까? `Dog` 와 `Cat`의 구조를 모두 가지고 있는 경우이다.
이 경우에는 동시에 여러 개의 스키마 구조를 만족하는 구조이므로, request body는 유효하지 않다.

> `oneOf`는 오직 하나의 스키마만 유효하도록 강제하는 키워드이다.

## allOf

앞서 보았던 `oneOf`처럼, OpenAPI 스펙은 `allOf` 키워드를 사용해서 모델 정의를 병합하고 확장할 수 있도록 지원한다.

`allOf` 키워드는
- 객체 명세의 배열을 받아
- 독립적인 검증을 수행한다.
- 각 객체의 배열은 함께 모여 하나의 객체를 구성한다.
- 하지만 모델 간의 계층 구조를 지원하지는 않는다.

아래 예제에서 `allOf`는 특정 사례에 사용되는 스키마를 일반적인 스키마와 결합하는 도구 역할을 한다.

보다 명료한 사용을 위해 `allOf` 키워드는 `discriminator`와 같이 사용되기도 한다.
```yaml
paths:
  /pets:
    patch:
      requestBody:
        content:
          application/json:
            schema:
              oneOf:
                - $ref: '#/components/schemas/Cat'
                - $ref: '#/components/schemas/Dog'
              discriminator:
                propertyName: pet_type
      responses:
        '200':
          description: Updated
components:
  schemas:
    Pet:
      type: object
      required:
        - pet_type
      properties:
        pet_type:
          type: string
      discriminator:
        propertyName: pet_type
    Dog:     # "Dog" is a value for the pet_type property (the discriminator value)
      allOf: # Combines the main `Pet` schema with `Dog`-specific properties 
        - $ref: '#/components/schemas/Pet'
        - type: object
          # all other properties specific to a `Dog`
          properties:
            bark:
              type: boolean
            breed:
              type: string
              enum: [Dingo, Husky, Retriever, Shepherd]
    Cat:     # "Cat" is a value for the pet_type property (the discriminator value)
      allOf: # Combines the main `Pet` schema with `Cat`-specific properties 
        - $ref: '#/components/schemas/Pet'
        - type: object
          # all other properties specific to a `Cat`
          properties:
            hunts:
              type: boolean
            age:
              type: integer
```
상기 예제는 request body에 대해 pet 데이터를 업데이트하기 위해 필요한 모든 정보가 다 있는지 검증한다. 

사용자는 업데이트 대상이 되는 아이템의 타입을 명세해야 하고, 선택한 아이템의 타입에 따라 사전 정의된 스키마 구조를 지키는지 검증한다.

인라인이나 ref로 풀어낸 스키마는 단순한 표준 JSON 스키마가 아니라 `스키마 객체`여야 한다는 점을 명심해야 한다.

예를 들어, 아래 request body들은 전부 유효한 입력이다.

```json
{
  "pet_type": "Cat",
  "age": 3
}
```
```json
{
  "pet_type": "Dog",
  "bark": true
}
```
```json
{
  "pet_type": "Dog",
  "bark": false,
  "breed": "Dingo"
}
```

반면 아래와 같은 입력들은 유효하지 않다.
```json
{
  "age": 3        # Does not include the pet_type property
}
```

```json
{
  "pet_type": "Cat", 
  "bark": true    # The `Cat` schema does not have the `bark` property 
}
```

## anyOf

`anyOf` 키워드는 입력된 하위 스키마 중에서 하나라도 매칭되는 것이 있는 것이 있는지 검증한다.
즉, 데이터는 동시에 하나 이상의 하위 스키마를 상대로 유효해야 한다.

```yaml
paths:
  /pets:
    patch:
      requestBody:
        content:
          application/json:
            schema:
              anyOf:
                - $ref: '#/components/schemas/PetByAge'
                - $ref: '#/components/schemas/PetByType'
      responses:
        '200':
          description: Updated
components:
  schemas:
    PetByAge:
      type: object
      properties: 
        age: 
          type: integer
        nickname: 
          type: string
      required:
        - age
          
    PetByType:
      type: object
      properties:
        pet_type:
          type: string
          enum: [Cat, Dog]
        hunts:
          type: boolean
      required: 
        - pet_type
```
상기 예시에서, 아래 입력들은 전부 유효한 입력이다.

```json
{
  "age": 1
}
```

```json
{
    "pet_type": "Cat",
    "hunts": true
}
```

```json
{
  "nickname": "Fido",
  "pet_type": "Dog",
  "age": 4
}
```
반면 아래와 같은 입력은 유효하지 않다. 왜냐하면 명시한 두 가지의 스키마의 필요한 속성들 중 아무것도 포함하고 있지 않기 떄문이다.

```json
{
  "nickname": "Mr. Paws",
  "hunts": false
}
```

### anyOf와 oneOf의 차이점

- `oneOf`는 딱 하나의 하위 스키마와 매칭되어야 유효하다.
- `anyOf`는 하나 이상의 스키마와 매칭된다면 유효하다.

## not

`not` 키워드는 스키마를 결합하기 위함이 아니라, 스키마를 수정하고 더 세세하게 사용할 수 있도록 돕는 키워드이다.
```yaml
paths:
  /pets:
    patch:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PetByType'
      responses:
        '200':
          description: Updated
components:
  schemas:          
    PetByType:
      type: object
      properties:
        pet_type:
          not:
            type: integer
      required:
        - pet_type
```

이 예제에서 사용자는 `pet_type`에 대한 값을 정수를 제외하고 입력해야 한다. 때문에 아래와 같은 request body는 유효하다.
```json
{
  "pet_type": "Cat"
}
```
반면 이런 입력은 유효하지 않다.
```json
{
  "pet_type": "Cat"
}
```
