---
title: "Webflux의 Hot Cold Publisher 개념"
date: 2023-01-16T20:11:51+09:00
draft: false
author: redjen
---

https://www.vinsguru.com/reactor-hot-publisher-vs-cold-publisher/

Reactor에는 두 가지 Publisher 인터페이스의 구현체가 있다. Mono와 Flux.
두 구현체 모두 비동기적으로 데이터를 쏴주지만, Flux는 0~N개의 원소를, Mono는 0 또는 1개의 원소를 쏜다는  것이 차이점이다.

이 구현체들의 동작과는 별개로 Publisher는 두 타입으로 범주화될 수 있다.

## Cold Publisher

우리가 일반적인 유튜브 스트리밍을 시작할 때에는 동영상이 스트리밍 되고 있지 않다가 우리가 재생 명령을 내리면 그제서야 동영상 스트리밍이 시작된다.
- 옆에 있는 친구와 내가 동일한 동영상을 동일한 시점에 재생을 해도 친구의 스트리밍과 나의 스트리밍은 다르다.
- 친구가 똑같은 동영상을 한 시간 이상 봤더라도 내가 지금 재생을 시작한 동영상에는 영향을 주지 않는다.

일반적인 상황에서의 유튜브 스트리밍은 Cold Publisher이다. (유튜브 라이브 스트리밍은 아니다)

Cold publisher는 subscribe를 시작한 시점과 관계 없이 publisher가 제공하는 모든 데이터를 손실 없이 전달 받을 수 있음이 보장된다. 
Request가 서버에 도달하면, Webflux 서버는 그제서야 downstream으로 데이터를 쏴주기 시작하는 것이다.

> Cold publisher는 subscribe 될 때마다 데이터를 처음부터 emit한다.

## Hot Publisher

영화관에서 영화를 보는 것은 직전의 케이스와는 다르다.
Hot publisher는 13:00에 시작하는 영화를 예매하고 내가 그 영화를 보러 영화관에 가는 것에 비유할 수 있다. 
내가 13:15분에 영화관에 도착하면 15분 동안의 영화는 보지 못한다.
반면 친구가 13:00에 영화관에 도착해서 앉아 있었다면 친구는 내가 보지 못한 15분 가량의 영화를 볼 수 있다.

> Hot publisher는 subscriber의 수와 상관 없이 데이터를 단 한 번만 emit한다.

Hot publisher를 subscribe하는 구독자들은 구독된 시점에 emit 받은 데이터만 전달 받을 수 있다.
