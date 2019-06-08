---
title: Effective Java 3rd ITEM 31
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-08 15:47:16
thumbnail:
---

# ITEM 31. Use bounded wildcards to increase API flexibility

매개변수화 타입은 불공변이기 때문에 이보다 유연한 무언가가 필요할 때가 있다.

## producer-extends
```java
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}

// Number Stack에 Integer Iterable를 넣으려고 할 때
Stack<Number> numStack = new Stack<>();
Iterable<Integer> integers = ...;
numStack.pushAll(integers);
```
위 코드는 Stack에 src의 모든 원소를 넣는 코드이다. Integer는 Number의 하위 타입이라 잘 동작할 것 같지만 불공변인 매개변수화 타입때문에 오류가 발생한다.
pushAll의 입력 매개변수 타입을 *E의 Iterable이 아닌 E의 하위타입의 Iterable*일 필요가 있다. 
***=> `Iterable<? extends E>`***
```java
public void pushAll(Iterable<? extends E> src) {
    ...;
}
```

## consumer-super
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

// Number Stack을 Obejct Collection에 넣으려고 할 때
Stack<Number> numStack = new Stack<>();
Collection<Object> objects = ...;
numStack.popAll(objects);
```
위 코드는 Stack에 있는 원소들을 모두 pop하여 Collection에 옮겨담는 코드이다. Number가 Object의 하위타입이기 때문에 잘 동작해야 하지만 불공변으로 인해 오류가 생긴다.
popAll의 입력 매개변수의 타입을 *E의 상위타입의 Collection*으로 해줘야 한다.
***=> `Collection<? super E>`***
```java
public void popAll(Collection<? super E> dst) {
    ...
}
```


## 타입 매개변수와 와일드 카드
```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
위 코드는 원소 swap을 하는 메소드를 2가지 방식으로 선언한 것이다. 이렇게 타입 매개변수와 와일드카드 둘 중 어느 것을 사용해도 괜찮을때가 많다. 이런 상황의 판단 기준은 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하는 것이 좋다. 따라서 public API라면 2번째 방식이 더 간단하고 좋다.
<br/>
```java
// Error List<?>.set()
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```
하지만 2번째 방식의 선언으로 메소드를 구현했을때 오류 메시지가 나올 것이다.
와일드 카드를 사용한 `List<?>`에는 null 이외에 다른 값을 넣을 수 없기 때문이다. 이 때문에 와일드 카드 타입의 실제 타입을 알려주는 private 메서드가 필요하다.
<br/>
```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```
`swapHelper` 메서드는 `List`의 타입 매개변수를 알 수 있기 때문에 swap이 가능해진다.
