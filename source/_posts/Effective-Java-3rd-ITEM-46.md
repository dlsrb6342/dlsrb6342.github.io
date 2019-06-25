---
title: Effective Java 3rd ITEM 46
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-25 20:32:19
thumbnail:
---

# ITEM 46. Prefer side-effect-free functions in streams

스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다. 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려는 API는 말할 것도 없고 이 함수형 패러다임도 함께 받아들여야 한다.

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 각 변환 단계에서 이전 단계의 결과를 받아 처리하는 순수 함수가 적용되어야 한다. 순수 함수란 입력값만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않아야 한다. 

스트림을 사용할 때 java.util.stream.Collectors는 아주 중요한 클래스이다. 스트림의 원소들을 객체 하나에 취합해주는 총 43개의 메서드를 가지고 있다. 
```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get)).reserved()
    .limit(10)
    .collect(toList());
```
여기서 쓰인 `toList()`가 Collectors 클래스에 있는 함수이다. `toList(), toSet(), toMap(), groupingBy(), joining()`같은 대표적인 것이 있고 나머지 42개의 메서드는 직접 클래스를 확인해보자.
