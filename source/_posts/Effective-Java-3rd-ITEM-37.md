---
title: Effective Java 3rd ITEM 37
categories:
  - Effective Java 3rd
  - 6. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-16 12:37:48
thumbnail:
---

# ITEM 37. Use EnumMap instead of ordinal indexing

## ordinal 인덱싱
```java
Set<Plant>[] plantsByLifeCycle = ...;
plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
```
이런 식으로 배열이나 리스트의 인덱스로 `ordinal()` 메서드를 사용한 코들들이 있다. 
이런 코드의 문제점은 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다는 점이다. 정수는 타입 안전하지 않기 때문에 잘못된 값을 사용할 수도 있다.
잘못된 값을 사용해도 아무 문제가 없는듯이 동작하거나 `ArrayIndexOutOfBoundsException`을 던질 것이다.

## EnumMap
```java
EnumMap<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = ...;
plantsByLifeCycle.get(p.lifeCycle).add(p);
```
열거타입을 키로 사용한 EnumMap이다. 더 짧고 명료하고 안전하고 내부에서는 배열을 사용하기 때문에 성능도 비등하다.
```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
{
    /**
     * The <tt>Class</tt> object for the enum type of all the keys of this map.
     *
     * @serial
     */
    private final Class<K> keyType;

    /**
     * All of the values comprising K.  (Cached for performance.)
     */
    private transient K[] keyUniverse;

    /**
     * Array representation of this map.  The ith element is the value
     * to which universe[i] is currently mapped, or null if it isn't
     * mapped to anything, or NULL if it's mapped to null.
     */
    private transient Object[] vals;

    ...
}
```
***=> Map의 타입안전성과 배열의 성능을 모두 얻어낸 것. 그러니 EnumMap을 사용하자!***

