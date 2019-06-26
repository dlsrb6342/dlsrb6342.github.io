---
title: Effective Java 3rd ITEM 48
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-26 18:55:41
thumbnail:
---

# ITEM 48. Use caution when making streams parallel

동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 얘서야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.
<br/>

### 데이터 소스 / limit()
데이터 소스가 `Stream.iterable`이거나 중간 연산으로 `limit()`가 들어가 있다면 병렬화로는 성능 개선을 기대할 수 없다. 
`Stream.iterable`은 스트림 라이브러리가 병렬화할 방법을 찾지 못하기 때문이고, `limit()`은 CPU 코어가 남는다면 limit 갯수 이상으로 처리하고 필요없으면 버리게 되는데 그 이상으로 처리되는 원소들이 오래 걸릴 수 있기 때문이다.
<br/>

### 자료구조
대체로 `ArrayList, HashMap, HashSet, ConcurrentHashMap`이나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋게 나타난다.
이 자료구조들은 데이터를 다수의 스레드로 분배하기 좋다. 이 나누는 작업은 `Spliterator`가 담당한다. 또 이 자료구조들은 원소들을 순차적으로 실행할 때 참조 지역성(locality of reference)가 뛰어나다. 참조 지역성이 좋으면 다량의 데이터를 벌크 연산을 병렬화할 때 캐시에 올리는 시간을 줄일 수 있다.
<br/>

### 종단연산
종단연산 또한 병렬 수행 효율에 큰 영향을 끼친다. `min, max, count, sum`이나 Stream의 reduce 메서드들같은 Reduction이 가장 적합한 종단 연산이고 collect 메서드같은 Mutable reduction이 병렬화에 적합하지 않다.

<br/>
> 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.
병렬화하는 것이 좋아보이더라도 수정 후에 성능지표를 확인하고 확실해졌을 때만 병렬화를 사용하자.
