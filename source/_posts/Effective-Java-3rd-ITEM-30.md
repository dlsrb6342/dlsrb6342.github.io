---
title: Effective Java 3rd ITEM 30
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-08 11:19:35
thumbnail:
---

# ITEM 30. Favor generic methods

## 제네릭 메서드
```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

위와 같이 제네릭이 아닌 raw type으로 사용 시에는 unchecked 경고가 나온다. 또 클라이언트가 사용 시에 직접 타입 변환을 해야 하기 때문에 런타임 오류가 생길 수 있는 위험이 있다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>();
    result.addAll(s2);
    return result;
}
```
제네릭 메서드로 만들면 경고 문구도 없고 타입 안전하고 쓰기도 쉽다. 
## 제네릭 싱글턴 팩터리
```java
private static final ReverseComparator rcInstance = new ReverseComparator();
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) rcInstance;
}

public static final Set EMPTY_SET = new EmptySet();
public static final <T> Set<T> emptySet() {
    return (EmptySet<T>) EMPTY_SET;
}
```

불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입이 소거되기 때문에 하나의 객체가 여러 타입으로든 매개변수화할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요하다. 위 코드가 그 2가지 예시이고 이러한 패턴을 제네릭 싱글턴 팩터리라고 한다.
* `Collections.reverseOrder()`는 `ReverseComparator`를 매개 변수에 맞게 타입을 바꾸고 반환해주는 함수이다.
* `Collections.emptySet()`은 비어있는 EmptySet 객체를 매개 변수를 바꿔 반환해주는 함수이다.

## 재귀적 타입 한정
자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정하는 것을 재귀적 타입 한정이라고 한다. `Comparable<T>` interface를 예시로 보겠다.

`Comparable<T>`의 T는 비교할 수 있는 원소의 타입을 정의한다. `Comparable`을 구현한 원소의 컬렉션을 입력받는 메서드들은 정렬 / 탐색 / 최솟값 / 최대값을 구하는 식으로 사용되는데 이는 컬렉션의 모든 원소가 상호 기교 가능해야 된다.
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    ...
}
```
위 코드 예시처럼 받을 수 있는 타입 매개변수로 `Comparable`을 구현한 타입만으로 제한하고 사용한다.
<br/>
제네릭 타입과 마찬가지로 클라이언트에서 입력 매개변수화 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 형변환을 해줘야 하는 메서드는 제네릭하게 만들자.
