---
title: "Spring Webflux와 Mediatype"
date: 2023-01-03T22:29:58+09:00
draft: false
author: redjen
---

## 개요
WebTestClient와 StepVerifier를 사용한 Spring Webflux API 엔드포인트 테스트를 하고 있었는데,
다음 코드가 예상과는 다르게 test fail하여 하루 종일 헤맸다.

```java
@RestController  
@SpringBootApplication  
public class WebfluxStreamapiTestApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(WebfluxStreamapiTestApplication.class, args);  
    }  
    
    @GetMapping("/hello")  
    Flux<String> hello() {  
        return Flux.just("Hello", "world!");  
    }
}
```

```java
@SpringBootTest  
public class ReactiveApiControllerTest {  
  
    private WebTestClient client;  
  
    @BeforeEach  
    public void setUp() {  
        this.client = WebTestClient.bindToServer()  
            .baseUrl("http://localhost:8080")  
            .build();  
    }  
    @Test  
    void helloApiTest() {  
        FluxExchangeResult<String> result = client  
            .get()  
            .uri("/hello")  
            .accept(MediaType.APPLICATION_STREAM_JSON)  
            .exchange()  
            .returnResult(String.class);  
  
        StepVerifier.create(result.getResponseBody())  
            .expectNext("Hello")  
            .expectNext("world!")  
            .verifyComplete();  
    }}
```

"Hello", "World" 두 개의 String 객체를 Flux로 리턴하는 /hello 엔드포인트 컨트롤러에서 아무리 테스트를 해도 "HelloWorld!" 하나의 String만 리턴되는 현상이었다. 

blocking 코드는 없다고 생각했지만 혹시 몰라서 WebTestClient 내부 로직에서 결과를 모아서 한번에 주는 block 로직이 존재하나? 라는 생각에 레퍼런스를 엄청 찾아봤지만 없었다.

## 해결...?

그러다 도중에 들었던 생각은 컨트롤러에 produce 하는 형식을 별도로 지정하지 않았으니, MediaType에 따라 curl 요청이 다르게 응답할 수도 있지 않을까? 였다.

실제로 curl로 요청해봤을 때는 

![](https://velog.velcdn.com/images/redjen/post/37bba833-d948-477c-bfc5-81724949d09c/image.png)


MediaType으로 application/stream+json을 지정했을 때와, text/event-stream으로 지정했을 때가 동작이 상이했다.

결국 테스트 코드를 아래와 같이 수정하니 정상적으로 test pass 하는 것을 볼 수 있었다.
```java
@Test  
void helloApiTest() {  
	FluxExchangeResult<String> result = client  
		.get()  
		.uri("/hello")  
		.accept(MediaType.TEXT_EVENT_STREAM)  
		.exchange()  
		.returnResult(String.class);  

	StepVerifier.create(result.getResponseBody())  
		.expectNext("Hello")  
		.expectNext("world!")  
		.verifyComplete(); 
```

## application/stream+json과 text/event-stream은 어떻게 다른가?
그래서 둘의 차이점을 찾아보니
https://stackoverflow.com/questions/52098863/whats-the-difference-between-text-event-stream-and-application-streamjson

```text/event-stream``` media type과 ```application/stream+json``` media type의 차이점으로는 다음과 같았다. 

특징 | text/event-stream | application/stream+json
--- | --- | ---
prefix의 존재 | ```data: ``` prefix가 존재 | prefix 없음
목적 대상 | 브라우저 | 서버 - 서버 통신
공식 여부 | SSE 이벤트 구현 위한 공식 스펙 | 현재 ```application/x-ndjson```이 공식 타입이 됨 (deprecated)

```Flux<T>``` 형태를 리턴하는 컨트롤러의 api 엔드포인트를 테스트할 때에는 media type에 유의해야 한다. 컨트롤러에 produce 하는 MediaType을 명시하는 것이 좋은 방법이 될 수 있을 듯 하다. 

## 'application/x-ndjson' Media Type에 대해

ndjson 포맷은 Newline Delimited JSON 포맷의 줄임말이다.
NDJSON은 한번에 하나의 데이터를 저장하거나 스트리밍할 때 편리한 포맷이다.

ndjson 포맷을 사용하면 한번의 API 콜로 여러 개의 삽입 / 삭제 작업을 수행할 수 있고,
이는 인덱싱 속도가 매우 중요한 ElasticSearch 등에서 유용하게 사용할 수 있다.

ndjson 포맷의 데이터의 마지막 라인은 ```\n```으로 끝나야 한다. 각 라인은 캐리지 리턴 ```\r```을 동반할 수 있다.
JSON 배열로 보내면 되지 않느냐? 라고 생각이 들 수 있는데, 
동료 노드가 벌키한 요청을 받았을 때 단순히 얼마나 많은 줄이 있는가?를 기준으로 데이터를 자를 수 있고,
협업 노드에 이 잘려진 덩어리를 보내서 효율적으로 처리할 수 있다는 장점이 있기 때문에 ndjson 포맷을 사용한다고 한다. 
만약 json 포맷이었다면 협업에 참여하는 노드가 전체를 전부 파싱해야 하고, 또 처리 대상 데이터가 수 MB에 이르는 대용량 쿼리라면 성능에 악영향이 있을 수 있기 때문이다.

## String은 특별해

찾아보니 String은 Flux로 래핑했을 때 다른 객체와 다르다고 그래서 또 테스트를 해봤다.

```java
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TestDomain {
    private String name;
    private LocalDateTime createdAt;
}
```

```java
@GetMapping("/test")
Flux<TestDomain> testDomainFlux () {
        return Flux.just(
                TestDomain.builder().name("test1").createdAt(LocalDateTime.now()).build(),
                TestDomain.builder().name("test2").createdAt(LocalDateTime.now()).build()
        );
    }
```

```java 
@Test
void testDomainApiTest() {
        Flux<TestDomain> result = client.get()
                .uri("/test")
                .accept(MediaType.APPLICATION_JSON)
                .exchange()
                .returnResult(TestDomain.class)
                .getResponseBody();

        StepVerifier.create(result)
                .assertNext(t -> assertThat(t.getName()).isEqualTo("test1"))
                .assertNext(t -> assertThat(t.getName()).isEqualTo("test2"))
                .verifyComplete();
    }
``` 
**테스트를 통과한다!** 이게 무슨 경우일까.. 이전 케이스에서 분명 ```application/json```일 때는 테스트를 통과하지 못했었다.
WebTestClient에서 accept할 MediaType을 ```application/stream+json```, ```text/event-stream```으로 지정해도 전부 통과하는 기염을 토했다 (...)

이 현상을 정리해보면

1. ```Flux<String>```을 스트리밍하는 API에서는 'Accept:text/event-stream' 헤더가 있을 때에만 의도한대로, 데이터 스트림이 분리되어 전달된다.
2. ```Flux<SomeObject>```를 스트리밍하는 API에서는 ```text/event-stream```, ```application/stream+json```, ```application/json``` MediaType들에서 전부 의도한대로 데이터 스트림이 분리되어 전달된다.

정말 다행히도 [관련 Issue](https://github.com/spring-projects/spring-framework/issues/20807)가 있었다. 요약하자면..

1. ```Flux<String>```은 각 string이 write 된 후에 바로 flush되면서 스트리밍된다.
2. Spring의 기본 json deserializer인 Jackson 라이브러리는 String 요소를 처리할 때 기본적으로 **사용되지 않는다.**
3. ```Flux<String>```은 json serializer/deserializer를 아예 거치지도 않으니 json 관련 MediaType에서 의도한대로 동작하지 않는다. (데이터 스트림이 분리되어 전달되지 않는다.)
4. ```Flux<SomeObject>```는 정상적으로 Jackson 라이브러리를 거쳐 객체로 역직렬화되므로, Json 관련 MediaType에서도 의도한대로 동작하고 ```text/event-stream```에서도 의도한대로 동작한다.

곰곰이 생각해보니까 당연한 것 같다. ```Flux<String>```은 Json 객체의 스트림이 될 수 없다. ```application/json```에서 의도한대로 동작하는 것이 이상하다. 다만 StepVerifier로 검증했을 때 fail하는 케이스인 "Helloworld!" 처럼 여러 스트림을 하나의 response body로는 읽도록 해준다.

또 이어지는 이유에 대해 써보자면, 

1. String을 리턴할 때에는 Json보다는 List로 리턴하는 경우가 더 자연스럽다.
2. ```flux.collectToList```를 통해 List로 만들기 편하기 때문이다. (!) - 여기서 원래 의도가 무엇이었는지 이해했다. 😅
3. Spring MVC에서도 ```application/json```일 때 String은 지금과 똑같은 방식으로 리턴되었다. 다만 내가 그 차이를  Reactive 스택일 때 발견한 것이다. 

![](https://velog.velcdn.com/images/redjen/post/91603e71-ab01-49eb-9d7a-beeb0733e8c5/image.png)

string이 아닐 때에는 application/json일 때에만 한번에 리턴되었다.

## 요약 (TL;DR)

스프링에게 억까를 당했다고 생각했지만... 내가 스프링 설계 전반에 깔려 있는 생각과 기본에 충실하지 않아서 범한 실수이다. 

> 
1. Spring에서 String element를 입력받거나 출력하는 Controller에서는 Jackson 라이브러리의 개입이 일체 일어나지 않는다.
2. Spring Webflux도 마찬가지이다. ```Flux<String>```으로 리턴하는 Controller에서는 Jackson 라이브러리의 개입이 없으므로 ```application/json```, ```application/stream+json``` (deprecated 되었다) 에서 데이터 스트림이 의도했던 대로 전달되지 않는다.
3. ```Flux<String>```을 리턴하는 controller에서 produce하는 MediaType을 ```text/event-stream```으로 명시해주자.
```Flux<SomeObject>```는 Jackson 라이브러리가 직렬화 / 역직렬화에 관여하므로 API 스펙에 맞게 설정해주면 된다. 