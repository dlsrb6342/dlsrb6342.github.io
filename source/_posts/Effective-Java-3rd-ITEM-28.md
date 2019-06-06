---
title: Effective Java 3rd ITEM 28
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-06 20:00:56
thumbnail:
---

# ITEM 28. Prefer lists to arrays

## 배열과 제네릭 타입의 차이
### 공변 / 불공변
배열은 공변(covariant)이고 제네릭은 불공변(invariant)이다. 
공변이란 하위 타입 배열이 상위 타입 배열의 하위 타입이 된다는 것이다. `Sub[]`은 `Super[]`의 하위 타입!
제네릭에서는 `List<Sub>`이 `List<Super>`의 하위 타입이 아니다.

### 실체화
배열은 런타임에도 자신의 원소의 타입을 확인하지만 제네릭 타입은 런타임에는 타입 정보가 소거되어 타입을 알 수 없다. 제네릭의 소거는 제네릭이 없는 버전과의 호환성을 위한 메커니즘이다.

## 배열과 제네릭의 호환
### 제네릭 타입 배열
`new List<E>[], new List<String>[], new E[]`와 같은 제네릭 타입의 배열은 생성할 수 없다.
제네릭 타입의 배열 생성을 허용하면 타입이 안전하지 않다. 
```java
List<String>[] stringLists = new List<String>[1]; //(1)
List<Integer> intList = List.of(42); //(2)
Object[] objects = stringLists; //(3)
objects[0] = intList; //(4)
String s = stringLists[0].get(0); //(5)
```
위 코드는 제네릭 타입 배열 생성을 허용했을 때 발생하는 문제에 대한 코드이다.
(1)에서 `List<String>` 배열을 만든다. 
(2)에서는 42 하나 들어있는 `List<Integer>`를 만들고 
(3)에서 `List<String>[]`을 `Object[]`에 넣는다. 배열은 공변이기 때문에 `List<String>[]`은 `Object[]`의 하위타입이 된다. 그래서 `Object[]`에 할당이 가능하다.
(4)에서 `Object[]`의 원소로 `List<Integer>`를 넣어준다. 이것이 가능한 이유는 제네릭 타입의 소거로 인해 `List<String>[]` -> `List[]`, `List<Intger>` -> `List`가 되기 때문이다.
문제는 (5)에서 발생한다. `List<String>`을 넣기로 했던 `stringLists`에 `List<Integer>`가 들어있어서 `stringLists[0].get(0)`을 했을때 Integer(42)가 나온다. 이를 String에 받으려고 하기 때문에 ClassCastException이 발생한다.

제네릭 컬렉션은 자신의 원소 타입으로 배열을 반환하는게 보통 불가능하고 Object[]로 반환한다. 또 가변인수에 제네릭 타입을 넣어줄 수 없다. 가변인수를 배열로 만들게 되기 때문이다.

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우, 대부분은 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다. 코드가 조금 복잡해지고 성능이 살짝 나빠지지만 그 대신 타입 안전성이 보장되고 상호운용성이 좋아진다.
