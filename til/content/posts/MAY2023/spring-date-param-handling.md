---
title: "Spring Date Param Handling"
date: 2023-05-10T21:56:55+09:00
draft: false
author: redjen
---

스프링 컨트롤러 레벨에서 `@RequestParam`으로 날짜 객체를 파라미터로 입력 받고 싶었다.

단순히 스프링은 똑똑하니까~ 아래와 같이 설정해주면 부드럽게 작동할 줄 알았다.

```java
@GetMapping("/some-path")
public Mono<Void> doSomeJob(
    @RequestParam Date inputDate
) {
    ...
}
```

하지만 결과는 예상과 다르게 Date 객체로 파싱하는데 실패했다는 오류 메시지를 볼 수 있었다.

즉, ISO 8601 형식의 날짜-시간 문자열 (`2023-05-10T20:00:00Z`) 은 스프링에서 자동으로 매핑해주지 않는다.

그렇다면 어떻게 이를 처리해야 할까? 생각보다 간단한 문제이지만 헷갈릴 수 있어 정리해봤다.

https://www.baeldung.com/spring-date-parameters

## 요청 레벨에서 Date 파라미터 받기

> 결론부터 말하자면, `@DateTimeFormat` 어노테이션을 `@RequestParam`과 함께 사용하면 된다.

```java
@RestController
public class DateTimeController {

    @PostMapping("/date")
    public void date(@RequestParam("date") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) Date date) {
        // ...
    }

    @PostMapping("/local-date")
    public void localDate(@RequestParam("localDate") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate localDate) {
        // ...
    }

    @PostMapping("/local-date-time")
    public void dateTime(@RequestParam("localDateTime") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime localDateTime) {
        // ...
    }
}
```

스프링이 날짜 문자열을 입력 받았을 때 어떤 객체로 파싱해야 하는지 정확하게 알기란 어렵다. 그렇기 때문에 일종의 힌트를 쥐어줌으로써 컨트롤러야~ 이 문자열 포맷이 들어온다면 이런 객체로 파싱해~ 라고 알려주는 격이다.

## 어플리케이션 레벨에서 변환하기

음.. 이 방법은 별로 내키진 않지만 대부분의 작업이 날짜 문자열을 공통으로 다뤄야 하는 어플리케이션이라면 활용할 수 있겠다.

```java
@Configuration
public class DateTimeConfig extends WebMvcConfigurationSupport {

    @Bean
    @Override
    public FormattingConversionService mvcConversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        DateTimeFormatterRegistrar dateTimeRegistrar = new DateTimeFormatterRegistrar();
        dateTimeRegistrar.setDateFormatter(DateTimeFormatter.ofPattern("dd.MM.yyyy"));
        dateTimeRegistrar.setDateTimeFormatter(DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm:ss"));
        dateTimeRegistrar.registerFormatters(conversionService);

        DateFormatterRegistrar dateRegistrar = new DateFormatterRegistrar();
        dateRegistrar.setFormatter(new DateFormatter("dd.MM.yyyy"));
        dateRegistrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

`WebMvcConfigurationSupport` 설정을 확장해서 `FormattingConversionService`를 빈으로 등록한다.
- 해당 빈은 요청으로 들어온 특정 포맷의 문자열에 대해 자동으로 `Date` 계열의 객체로 변환해주도록 하는 일을 맡는다.

