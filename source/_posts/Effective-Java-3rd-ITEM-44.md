---
title: Effective Java 3rd ITEM 44
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-20 22:50:02
thumbnail:
---

# ITEM 44. Favor the use of standard functional interfaces

## Functional Interfaces
람다가 추가되기 전, 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴을 많이 사용했다. 하지만 람다가 추가되면서 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 방식으로 많이 대체되었다. 

`LinkedHashMap`을 보면, removeEldestEntry를 재정의하여 오래된 원소를 제거할 수가 있다. 
```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```
size가 100이 넘어가면 오래된 원소를 제거하도록 재정의하였다. 만약 `LinkedHashMap`을 람다를 이용해 다시 구현한다면 removeEldestEntry를 함수 객체로 받는 정적 팩터리나 생성자를 제공했을 것이다.

## java.util.function 
java.util.function 패키지를 보면 표준 함수형 인터페이스들이 총 43개 구현되어있다. 필요한 용도에 맞는게 있다면 직접 구현하지 말고 표준 함수형 인터페이스를 활용하자.
https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html

## 직접 구현?
전용 함수형 인터페이스를 구현해야 하는지 고민이라면 아래 3가지 중 하나 이상을 만족하는지 확인하고 생각해봐야 한다.
1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.

`Comparator<T>`가 대표적인 예시이다. 이름 자체로 용도를 훌륭히 설명해주고 있고 규약을 가지며 여러 디폴트 메서드를 담고 있다. 
만약 함수형 인터페이스를 직접 구현하기로 했다면 반드시 `@FunctionalInterface` annotation을 사용하자. 이 클래스가 람다를 위해 설계된 것임을 알릴 수 있고 누군가 추상 메서드를 추가하지 못하게 막아주는 역할도 한다.
