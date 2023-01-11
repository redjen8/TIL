---
title: "Webflux에서 두 Publisher를 합칠때"
date: 2023-01-11T22:22:06+09:00
draft: false
author: redjen
---

주로 사용하는 연산자인 concat, merge, zip의 차이점은 무엇일까?

http://www.vinsguru.com/reactive-programming-reactor-combining-multiple-sources-of-flux-mono/

## concat()

concat은 말 그대로 source1의 데이터 뒤에 source2의 데이터를 이어서 합친다. 
그런데 hot / cold publisher 종류에 따라 좀 다르다. 
hot publisher의 경우에서 source1가 완료된 이후에 source2를 이어서 붙이게 되는데, 이 때 source1이 송출되는 동안에 유실된 source2에 대해서는 복구하지 않는다.

## merge()

merge는 단순히 두 개의 데이터 source를 eager하게 합친다.
순서는 막 섞이게 되고, source로부터 데이터가 튀어나오자 마자 바로 전달하는 것이 특징이다.

## zip()

두 개의 source가 모두 데이터 송출을 했을 때 데이터를 송출한다.
source 중 하나라도 완료되거나 에러가 발생한다면 멈춘다.
zip은 tuple 객체를 downstream으로 보낸다.
tuple 객체를 통해서 묶인 두 개의 source들의 데이터에 대한 처리를 구체적으로 할 수 있다. 