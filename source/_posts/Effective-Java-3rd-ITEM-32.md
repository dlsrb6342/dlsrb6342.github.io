---
title: Effective Java 3rd ITEM 32
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-09 19:05:36
thumbnail:
---

# ITEM 32. Combine generics and varargs judiciously

가변인수 메서드는 제네릭과 자바5에서 같이 추가되었다. 하지만 서로 잘 어울리지는 못한다.

## 가변인수
가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다. 가변인수를 담기위한 배열을 자동으로 생성하는데 그 배열을 클라이언트에 노출하는 문제가 있다. 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생한다.
```java
staticf void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList;               // 힙 오염 발생
    String s = stringLists[0].get(0);   // ClassCastException
}
```
이 메서드에서는 마지막 줄에 컴파일러가 자동 생성한 보이지 않는 형변환이 있다. 이렇게 매개변수화 타입의 변수가 타입이 다른 객체를 참조하여 힙 오염이 발생하면 컴파일러가 자동생성한 형변환에서 ClassCastException이 발생할 수 있다. 
***=> 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 타입 안정성이 깨진다.***
<br/>
```java
@SafeVarargs
@SuppressWarnings("varargs")
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```
제네릭 배열을 직접 생성하는 건 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유는 위 코드처럼 실무에 매우 유용하게 사용할 수 있기 때문이다.

## SafeVarargs annotation
자바7에서 `@SafeVarargs` 어노테이션이 추가되면서 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. `@SafeVarargs`는 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다.
* 메서드가 안전한지 확신할 수 있는 근거
  1. 메서드가 이 배열에 아무것도 저장하지 않는다.
  2. 그 배열(혹은 복제본)의 참조가 밖으로 노출되지 않는다.

***varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 해야 한다.***

```java
static <T> T[] toArray(T... args) {
    return args;
}
```
위 메서드처럼 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있다.
`@SafeVarargs`는 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 달아야 한다. 이 말은 안전하지 않는 가변인수 메서드는 절대 작성해서는 안된다는 뜻이다.
`@SafeVarargs`는 재정의할 수 없는 메서드에만 달아야 한다. java8에서는 static 메서드나 final 메서드에만 달 수 있었고 java9부터는 private 메서드에도 허용되었다.
## with List
varargs 매개변수를 List로 변경하는 방법도 있다. 매개변수 부분만 List로 변경해주면 된다. 호출하는 클라이언트에서는 `List.of()`를 사용하면 쉽게 임의의 개수를 인수로 넘겨줄 수 있다.
이 방식은 직접 `@SafeVarargs`를 달지 않아도 되며 안전한지를 판단할 필요도 없다. 다만 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느릴 수 있다는 점이다. 하지만 결과 코드는 제네릭만 사용하므로 타입 안전하다.

