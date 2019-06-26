---
title: Effective Java 3rd ITEM 36
categories:
  - Effective Java 3rd
  - 6. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-16 00:57:29
thumbnail:
---

# ITEM 35. Use EnumSet instead of bit fields

## 비트 필드
```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // Parameter is bitwise OR of zero or more STYLE_constants
    public void applyStyles(int styles) { ... }
}

text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
열거 값들이 단독이 아닌 집합으로 사용될 경우, 예전에는 정수 열거 상수에 2의 거듭 제곱 값을 할당하여 사용했었다.

### 장점
1. 비트별 연산을 사용해 집합 연산이 효율적이다.

### 단점
1. 정수 열거 상수의 모든 단점을 그대로 지닌다.
2. 비트 필드 값이 그대로 출력되면 해석하기 어렵다.
3. 비트 필드의 모든 원소를  순회하기 어렵다. 
4. 최대 몇 비트가 필요한지 예측하여 적절한 타입(int나 long)을 선택해야 한다.

## EnumSet
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) { ... }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
1. 열거 타입 상수의 값으로 구성된 집합을 표현한다. 
2. Set 인터페이스를 완벽히 구현하며 타입 안전하다.
3. 다른 Set 구현체와도 함께 사용할 수 있다.
4. 내부 구현은 비트 벡터로 되어있어서 원소 64개 이하라면 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
5. `removeAll() retainAll()`같은 대량 작업은 효율적으로 처리할 수 있게 구현되어있다.
6. 직접 비트를 다룰 때 겪는 오류들에서 해방된다.
7. 깔끔!

<br/>
EnumSet을 사용하자!
