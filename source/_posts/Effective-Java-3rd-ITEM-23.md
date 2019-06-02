---
title: Effective Java 3rd ITEM 23
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-02 17:29:13
thumbnail:
---

# ITEM 23. Prefer class hierarchies to tagged classes

## 태그 달린 클래스
태그 달린 클래스란 내부에 enum type을 가지고 enum 값에 따라 인스턴스가 의미하는 바가 달라지는 클래스를 이야기한다.
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    final Shape shape;

    // For rectangle
    double length; double width;

    // For circle
    double radius
    
    double area() {
        switch(shape) {
            ...
        }
    }
}
```
이러한 클래스는 태그마다 동작이 달라지는 method에는 switch 문이 필요하고 각각이 필요한 필드들을 함께 가지고 있어야 한다. 심지어 관련없는 필드들은 초기화를 하지 않아 final 선언도 불가하다. 
만약 switch문에서 한가지 태그를 빠뜨렸다면 컴파일 타임이 아닌 런타임에 오류가 생길 것이다.
태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율 적이다.

## 클래스 계층구조
계층 구조의 root가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 추상 메서드로 빼주면 된다. 위 Figure에서는 area() 함수가 그에 해당한다.
```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); }
} 

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override double area() { return length * width; }
}
```
계층구조로 작성했을 때는 새로운 타입을 추가하더라도 컴파일 타임에서 추상 메서드 구현 여부를 확인해주기 때문에 런타임 오류가 발생할 일도 없다.
