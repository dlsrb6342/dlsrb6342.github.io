---
title: Effective Java 3rd. ITEM 2.
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-02-24 16:19:37
thumbnail:
---

# ITEM 2. If Constructor has many parameters, Use builder.

Static factory method나 Constructor들은 매개변수가 많을 경우 대응이 어렵다.
## 대안
### 1. 점층적 생성자 패턴(Telescoping Constructor Pattern)
paramter를 하나씩 하나씩 늘려 Constructor를 여러 개 작성할 수 있겠지만 코드를 작성하거나 읽기 어려워진다.
<br/>
### 2. JavaBeans Pattern
NoArgsConstructor로 생성 후, Setter를 활용해 객체 생성.
단점으로는 하나의 객체를 만들때 여러 method를 호출하게 되고 모든 값이 정해지기 전까지는 일관성이 무너진 상태이다.
-> 객체를 Immutable한 상태로 만들기 어렵다. -> Thread unsafe
<br/>
### 3. Builder Pattern
Telescoping Constructor의 안전성과 JavaBeans의 가독성을 가진 Pattern
Builder의 setter method들은 자기 자신을 반환하기 때문에 연쇄적 호출 가능
-> Fluent API / Method Chaining이라 한다.

Builder Pattern은 계층적으로 설계된 클래스와 함께 쓰기도 좋다. 책에서는 예시로 Pizza를 보여줬지만 다른 예시를 만들어보았다.

```java
public abstract class Coffee {
    public static enum Size { TALL, GRANDE, VENTI }
    final Size size;

    Coffee(Builder<?> builder) {
        size = builder.size;
    }

    abstract static class Builder<T extends Builder<T>> {
        Size size;

        public T size(Size size) {
            this.size = size;
            return self();
        }

        abstract Coffee build();

        protected abstract T self();
    }
}
```
공통된 변수 size를 가지는 Coffee라는 Class이다. Coffee의 하위 클래스에는 Cappucino, HazelnutAmericano가 있다.
HazelnutAmericano는 syrup의 양을 필수 매개변수로 받고 Cappucino는 cinnamon을 넣을지 말지를 필수 매개변수로 받는다.
```java
public class HazelnutAmericano extends Coffee {
    private final int syrup;

    private HazelnutAmericano(Builder builder) {
        super(builder);
        syrup = builder.syrup;
    }

    public static class Builder extends Coffee.Builder<Builder> {
        private final int syrup;

        public Builder(int syrup) {
            this.syrup = syrup;
        }

        @Override
        public HazelnutAmericano build() {
            return new HazelnutAmericano(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
}
```
```java
public class Cappuccino extends Coffee {
    private final boolean cinnamon;

    Cappuccino(Builder builder) {
        super(builder);
        cinnamon = builder.cinnamon;
    }

    public static class Builder extends Coffee.Builder<Builder> {
        private final boolean cinnamon;

        public Builder(boolean cinnamon) {
            this.cinnamon = cinnamon;
        }

        @Override
        public Cappuccino build() {
            return new Cappuccino(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
}
```
```java
HazelnutAmericano hazelnutAmericano = new HazelnutAmericano.Builder(1)
        .size(Size.TALL)
        .build();
Cappuccino cappuccino = new Cappuccino.Builder(true)
        .size(Size.VENTI)
        .build();
```

위와 같이 계층적으로 빌더를 사용할 수 있다.
Builder Pattern의 단점으로는 Builder를 생성해야 하는 비용이 든다는 것이 있다.


## 참고
https://projectlombok.org/features/Builder
https://projectlombok.org/features/experimental/SuperBuilder 
