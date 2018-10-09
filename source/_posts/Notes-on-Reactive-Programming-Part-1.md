---
title: Notes on Reactive Programming Part 1
categories:
  - Spring
  - Reactor 
tags:
  - spring
  - reactor
date: 2018-10-09 19:02:53
thumbnail:
---

# The Reactive Landscape
spring blog를 보고 정리. 2016년 7월 글이다.

## What is it
Micro Architecture에서 routing과 events의 변경 및 소비에 관한 programming style.
Reactive Programming의 흐름을 spreadsheet에 비교.
* 변수 하나를 cell에 대입하고 cell이 변화했을때 그 cell을 참조하는 다른 cell도 변화가 일어남.

> The basic idea behind reactive programming is that there are certain datatypes that represent a value "over time". Computations that involve these changing-over-time values will themselves have values that change over time.
An easy way of reaching a first intuition about what it's like is to imagine your program is a spreadsheet and all of your variables are cells. If any of the cells in a spreadsheet change, any cells that refer to that cell change as well. It's just the same with FRP. Now imagine that some of the cells change on their own (or rather, are taken from the outside world)  


## Reactive use cases (what is it good for?)
**External Service Calls**
대부분의 backend는 HTTP를 이용한 RESTful이기 때문에 blocking, synchronous하다.
response는 여러 서비스들을 call하면서 그 응답에 따라 값이 변한다. 따라서 **External Service Calls**을 모아야 하는 상황에서 Reactive Programming으로 optimizing하기 좋다.

**Highly Concurrent Message Consumers**
message 하나를 event 하나로 바로 대응되기 때문에 Message Processing에는 Reactive Pattern이 어울린다.

## Reactive Programming in Java
[David Karnok’s Generations of Reactive classification](https://akarnokd.blogspot.com/2016/03/operator-fusion-part-1.html)
Reactor는 Reactive Streams을 Pivotal에서 만든 framework이다. Asynchronous sequence API로 Flux(N개)와 Mono(0, 1개)를 제공하고 Java 8 functional APIs로 만들어져있다.

> Reactor is a fully non-blocking reactive programming foundation for the JVM, with efficient demand management (in the form of managing "backpressure"). It integrates directly with the Java 8 functional APIs, notably CompletableFuture, Stream, and Duration. It offers composable asynchronous sequence APIs Flux (for [N] elements) and Mono (for [0|1] elements), extensively implementing the [Reactive Streams](http://www.reactive-streams.org/) specification.

## Why now?

> Well, it’s not (all) just a technology fad  —  people jumping on the bandwagon with the shiny new toys. The driver is efficient resource utilization, or in other words, spending less money on servers and data centres.


## 참고할만한 링크
* [spring blog 본문링크](https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape)
* [Reactor](http://projectreactor.io/docs/core/release/reference/)
* [Reactive Glossary](https://www.reactivemanifesto.org/ko/glossary)
