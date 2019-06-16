---
title: Effective Java 3rd ITEM 38
categories:
  - Effective Java 3rd
  - 5. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-16 13:09:32
thumbnail:
---

# ITEM 38. Emulate extensible enums with interfaces

열거 타입은 확장할 수가 없다. 대부분의 상황에서 열거 타입을 확장하는건 좋은 생각이 아니지만 연산 코드와 같은 상황에서는 필요하다.

## Operation
임의의 인터페이스를 추가하고 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다면 확장의 효과를 낼 수 있다. 
```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    ...
}
```
Operation 인터페이스를 만들고 BasicOperation이 그를 구현했다. Operation 인터페이스를 구현한 다른 열거 타입을 만든다면 확장의 효과를 낼 수 있다. 
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) { return Math.pow(x, y); }
    },
    REMAINDER("%") {
        public double apply(double x, double y) { return x % y; }
    };

    ...
}
```
추가한 열거타입 ExtendedOperation이다. 새로 추가되었지만 기존에 Operation 인터페이스를 사용하도록 작성되어있는 곳이면 어디든 사용 가능하다.

한가지 사소한 문제가 있는데 바로 열거 타입끼리 구현을 상속할 수는 없다는 점이다. 공유되는 부분이 적은 경우는 괜찮지만 많다면 도우미 클래스나 정적 메서드로 분리해서 코드 중복을 피할 수 있을 것이다.

## 참고
java.nio.file.LinkOption
