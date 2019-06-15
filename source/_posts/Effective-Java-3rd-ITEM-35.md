---
title: Effective Java 3rd ITEM 35
categories:
  - Effective Java 3rd
  - 5. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-16 00:51:13
thumbnail:
---

# ITEM 34. Use instance fields instead of ordinals

```java
/**
 * Returns the ordinal of this enumeration constant (its position
 * in its enum declaration, where the initial constant is assigned
 * an ordinal of zero).
 *
 * Most programmers will have no use for this method.  It is
 * designed for use by sophisticated enum-based data structures, such
 * as {@link java.util.EnumSet} and {@link java.util.EnumMap}.
 *
 * @return the ordinal of this enumeration constant
 */
public final int ordinal() {
    return ordinal;
}
```
모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal()` 메서드를 제공한다. 하지만 `ordianl()` 메서드의 문서에 적혀있는 것처럼 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적이 아니라면 이 메서드를 사용해서는 안된다.

열거 타입 상수에 연결된 정수 값을 얻고자 한다면 직접 인스턴스 필드에 저장해서 사용하자.

