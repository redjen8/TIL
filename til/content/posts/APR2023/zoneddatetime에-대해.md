---
title: "ZonedDateTime에 대해"
date: 2023-04-21T18:07:10+09:00
draft: false
author: redjen
---

엘라스틱서치의 기본 `Date` 타입 필드를 자바로 역직렬화하는 과정에서 `ZonedDateTime`에 대해서 알게 되었다.

뭔지 몰라서 찾아보다가 알게 된 내용을 정리한다.

https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html

## ZonedDateTime에 대해

`ZonedDateTime`은 TimeZone을 끼얹은 immutable한 DateTime 표현체이다.

이 클래스는 모든 날짜와, 시간 필드를 저장한다.
- 정확도는 나노 세컨드 단위까지 지원한다.
- TimeZone의 경우 zone offset을 통해서 임의의 local date time을 다룰 수 있다.
- 예를 들어 Europe/Paris 타임 존의 '2007년 10월 2일 13시 45분 30.123456789초 + 02:00' 시간은 `ZonedDateTime`에 저장될 수 있다.

이 클래스는 `LocalDateTime`이 제공해주는 로컬 타임 라인에서 `Instant` 가 제공해주는 인스턴트 타임라인으로의 변환도 처리해준다.
`LocalDateTime`과 `Instant`의 차이는 UTC, 그리니치 표준시로부터의 오프셋인 `ZoneOffset`으로 표현한다.

## 두 타임라인 간 Instant 계산 엣지 케이스

두 타임라인 간 변환에는 `ZoneId`에서 얻을 수 있었던 규칙을 사용해서 오프셋을 계산하는 작업이 포함된다.
- 인스턴트로부터 오프셋을 구하는 것은 간단하다.
- 하지만 로컬 DateTime에 대한 오프셋을 구하는 것은 어렵다.

1. 유효한 오프셋이 하나 있는 경우 : 대부분의 경우 로컬 DateTime에 대해 유효한 오프셋이 하나 존재한다.
2. 유효한 오프셋이 없는 경우 : 봄철 서머타임이 겨울에서 여름으로 변경되어 시계가 앞당겨지는 경우이다. 이 경우에는 '유효한 오프셋이 없는' 로컬 DateTime이 존재한다.
3. 두 개의 유효한 오프셋이 있는 경우 : 가을 서머타임이 여름에서 겨울로 변경되어 시계가 뒤로 설정되는 경우이다. 이 경우에는 두 개의 유효한 오프셋이 있는 로컬 DateTime이 존재한다.

## 정리

즉, `ZonedDateTime`은 날짜와 시간을 같이 저장하는 인스턴스에다 `Zone` 정보를 더한 인스턴스이다.

`ZonedDateTime`에서 `LocalDateTime`으로의 변환은 아래와 같이 이뤄질 수 있다.

https://stackoverflow.com/questions/49853999/convert-zoneddatetime-to-localdatetime-at-time-zone

```java
ZonedDateTime utcZoned = ZonedDateTime.of(LocalDate.now().atTime(11, 30), ZoneOffset.UTC);
ZoneId swissZone = ZoneId.of("Europe/Zurich");
ZonedDateTime swissZoned = utcZoned.withZoneSameInstant(swissZone);
LocalDateTime swissLocal = swissZoned.toLocalDateTime();
```

- `LocalDateTime` : 기본적으로 날짜와 시간을 예쁜 문자열로 표현하기 위함이다. 타임존에 구애받지 않기 때문에 타임라인의 어떤 특정 시점을 나타내지는 않는다.
- `Instant` : epoch 이후 경화한 시간을 **밀리초** 단위로 나타낸다. 타임라인에서 특정 시점을 나타내기에 용이하다.
- `ZonedDateTime` : 타임라인에서 특정 시점을 나타내지만, 시간대가 존재하는 날짜 및 시간으로 표시된다.

