---
title: Effective Java 3rd ITEM 53
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-30 17:18:05
thumbnail:
---

# ITEM 54. Use varargs judiciously

가변인수 메서드는 인수를 0개 이상 받을 수 있는 메서드이다. 인수들을 배열에 저장하여 메서드에 전달해준다. 

## 가변인수가 1개 이상 필요할때
가변인수가 1개 이상이어야 할 때는 인수로 가변인수만 받게 설계하는 것은 좋지 않다.
```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```
위 코드는 1개 이상의 가변인수를 받아 최솟값을 반환하는 메서드이다. 이 메서드는 0개의 인수를 받았을때 런타임 예외가 발생하는 점과 명시적인 유효성 검사를 해야하는 등 여러 문제점이 있다.

이럴 경우에는 첫 번째 인수로 평범한 매겨변수를 받고, 두 번째 인수에 가변인수를 받으면 해결된다.
```java
static int min(int firstArg, int... args) {
    int min = firstArg;
    for (int arg : args)
        if (arg < min)
            min = arg;
    return min;
}
```

## 성능에 민감한 상황

만약 성능에 민감한 상황에서 가변인수를 사용해야 한다면 배열을 새로 하나 할당하고 초기화하는 가변인수는 걸림돌이 될 수 있다.
이럴 경우에는 인수가 0개인 것부터 4개인 것까지 만들어주면 그나마 성능을 올릴 수 있다.
```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```
