---
title: Effective Java 3rd ITEM 29
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-06 21:44:48
thumbnail:
---

# ITEM 29. Favor generic types

```java
public class Stack {
    private Object[] elements;

    public Stack() {
        elements = new Object[16];
    }

    public Object pop() {
        ...
    }
}
```
위 Stack class의 Object배열은 제네릭 타입으로 쓰는게 좋다. 지금 상태로는 스택에서 pop한 객체를 원하는 타입으로의 형변환이 필요해 런타임 오류가 생길 위험이 있다.

```java
public class Stack<E> {
    private E[] elements;

    public Stack() {
        elements = new E[16];
    }

    public E pop() {
        ...
    }
}
```
이렇게 제네릭 E를 써서 수정할 수 있다. 하지만 제네릭 타입은 배열로 만들 수 없기 때문에 `elements = new E[16];`에서 오류가 난다. 이것을 해결하는 방법은 2가지가 있다.

### 1. 비검사 형변환!
```java
public Stack() {
    elements = (E[]) new Object[16];
}
```
제네릭 타입 배열을 만들 수 없으니 Object 배열로 만들고 비검사 형변환을 해주는 것이다.
`elements` 필드에는 항상 E 타입이 들어간다는 보장이 있기 때문에 이렇게 비검사 형변환을 해주고 `@SuppressWarnings`를 달아주자.

### 2. Object[]로!
```java
public class Stack<E> {
    private Object[] elements;
    ...
}
```
`elements` 필드를 Object[]로 쓰는 것이다. 이렇게 하면 `pop()` method에서 `incompatible types` 경고가 발생한다. E 타입을 반환해야 하는데 Object가 나오기 때문이다. 따라서 `pop()`할때 Object를 E 타입으로 형변환을 해줘야 한다. 하지만 이번엔 `unchecked cast` 경고가 나온다. 하지만 E 타입인 것을 보장할 수 있기 때문에 경고를 없애주기 위해 `@SuppressWarnings`을 넣어주자.

두 방법 모두 나름의 지지를 얻고 있다. 첫 번째는 배열을 `E[]`로 선언하기 때문에 가독성이 좋고 코드도 짧지만 힙오염이 발생할 수 있다. 두 번째 방법은 힙오염은 발생하지 않지만 배열에서 원소를 꺼낼때마다 형변환이 필요하다. 
<br/>
위 예시인 Stack과 같은 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약이 없어서 `Stack<int[]>, Stack<Object>, Stack<List<String>>`과 같이 사용할 수 있다. 단 `int, double` 등 기본 타입은 못 쓰니 대신 Boxing 타입을 사용하자.
해당 클래스를 사용하는 입장에서 직접 형변환해야 하는 타입보다 제네릭 타입으로 쓰면 더 안전하고 쓰기 편하다. 
형변환 없이 사용할 수 있도록 제네릭으로 설계하자.
