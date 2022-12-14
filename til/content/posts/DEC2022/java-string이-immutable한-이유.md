---
title: "Java String이 Immutable한 이유"
date: 2022-12-13T16:45:27+09:00
draft: false
---

# Java String이 Immutable한 이유

[원문 링크](https://www.baeldung.com/java-string-immutable)

3줄 요약
1. 캐싱
2. 보안
3. 동기
4. 성능

## 1. 캐싱했을 때 이점이 있어서

String은 가장 많이 쓰이는 data structure
그래서 캐싱했을 때 다른 객체보다 이점이 크다.

동일한 String 객체 "Hello, world!" 가 있을 때
```
String s1 = "Hello, world!";
String s2 = "Hello, world!";
```
s1과 s2는 서로 같은 힙 공간의 주소를 가르킨다. 그래서 **s1 == s2는 true**이다. 
그런데 
```
String s3 = new String("Hello, world!");
```
은 다른 주소 공간을 레퍼런스한다

## 2. 보안적으로 중요한 정보를 저장하는 경우가 많기 때문

String 형이 mutable 하다면, 데이터를 업데이트 했을 때 안전한 정보를 받았을 것이라는 보장을 할 수가 없음
SQL injection과 같은 공격이 더 잘 생길 것 같긴 하다
기존 String 쿼리를 뭔가 수정할 수 있다고 했을 때 발생하는 문제들..
C와 같은 언어에서 쿼리를 작성한다고 했을 때 char 배열에서 인덱스 바꿔가면서 쿼리를 수정할 수 있지 않나? 그런 예시인듯?

## 3. thread safe한 자료형이 가지는 이점 때문

immutable한 객체들은 보통 다수의 쓰레드에 의해 접근되었을 때에도 잘 공유된다.
AtomicInteger 같은 객체는 있지만 AtomicString 과 같은 객체는 없잖아
한 쓰레드가 String의 값을 변경하면 그 String을 저장하는 레퍼런스 객체가 새 String pool에 있는 새 객체를 가르킨다. 그래서 String은 쓰레드 세이프

## 4. Hashcode 캐싱

String이 immutable 하기 때문에 String의 hashCode 또한 불변한다.
따라서 같은 String을 가르키고 있는 레퍼런스 객체라면 hashCode가 똑같음을 보장할 수 있으므로 캐싱의 효과가 있다..
해시를 사용하는 컬렉션들과 같이 사용했을 때에 강력한 성능을 발휘할 수 있다.
[String.hashCode()에 대해](https://leedo.me/36)

## 5. 성능 상 이점

힙 메모리 사용을 줄이고 해시 구현체들에 immutable한 String을 사용했을 때 이점이 있다
특히 String은 자주 사용되는 자료형이기 때문에..