---
title: Effective Java 3rd ITEM 43
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-20 22:33:40
thumbnail:
---

# ITEM 43. Prefer method references to lambdas

## Method References
Method references는 자바에서 함수 객체를 람다보다 간결하게 만드는 방법이다. 
```java
map.merge(key, 1, (count, incr) -> count + incr);
map.merge(key, 1, Integer::sum);
```
첫번째 줄을 method references를 이용해 두번째 줄로 간결하고 보기 좋게 줄일 수 있다.

```java
service.execute(GoshThisClassNameIsHumongous::action);
service.execute(() -> action());
```
하지만 위 코드처럼 람다가 더 간결할 때도 있고, 어떤 람다에서는 매개변수의 이름이 프로그래머에게 좋은 가이드가 되기도 하기 때문에 길이는 더 길지만 읽기 쉽고 유지보수가 쉬울 수 있다.

## Method References 유형

| 메서드 참조 유형   | 예                     | 같은 기능을 하는 람다                              |
|--------------------|------------------------|----------------------------------------------------|
| 정적               | Integer::parseInt      | str -> Integer.parseInt(str)                       |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now(); t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                           |
| 클래스 생성자      | TreeMap<K,V>::new      | () -> new TreeMap<K,V>()                           |
| 배열 생성자        | int[]::new             | len -> new int[len]                                |

<br/>
***메서드 참조는 람다의 간단명료한 대안이 될 수 있다. 메서드 참조가 더 짧고 명확하다면 메서드 참조를 쓰고, 아니라면 람다를 쓰자.***
