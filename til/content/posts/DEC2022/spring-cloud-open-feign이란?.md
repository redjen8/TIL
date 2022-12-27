---
title: "Spring Cloud Open Feign이란?"
date: 2022-12-27T16:47:33+09:00
draft: false
author : redjen
---

# Spring Cloud Open Feign이란?

https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/

Feign은 Netflix에서 개발한 Http client binder
Feign interface 작성 후 어노테이션 선언만 하면 웹 서비스 클라이언트를 뚝딱

MSA 사용할 때에는 하나의 어플리케이션에서 각 서버로 서비스를 요청하는 경우가 많다.
이 때 FeignClient 사용하면 RestTemplate 보다 좀 더 편리하게 사용할 수 있다는 장점

넓은 의미의 repository로 사용할 수 있다
DB도 네트워크 통신 통해서 데이터 질의 하듯이..
다른 서버로 데이터 질의 해서 가져오는 것이 repository의 역할이라면 FeignClient들은 repository로써 기능하는 것이 맞다는 생각이 들었음

회사에서는 ReactiveFeign을 쓰고 있었는데, 설정이 매우 복잡하더라..
아마도 비동기 기반의 HTTP client 일 것.
Reactive 스택을 쓴다면 HTTP client 또한 reactive 여야..
그런데 reactive-feign은 아직 공식 라이브러리가 아니네?
https://github.com/PlaytikaOSS/feign-reactive

예제 코드 출처
https://www.baeldung.com/spring-cloud-openfeign

```java
@FeignClient(value = "jplaceholder", url = "https://jsonplaceholder.typicode.com/")
public interface JSONPlaceHolderClient {

    @RequestMapping(method = RequestMethod.GET, value = "/posts")
    List<Post> getPosts();

    @RequestMapping(method = RequestMethod.GET, value = "/posts/{postId}", produces = "application/json")
    Post getPostById(@PathVariable("postId") Long postId);
}
```
이런 식으로 작성해서, GET /posts 요청을 지정한 url로 보낼 수 있다. POST도 마찬가지.
