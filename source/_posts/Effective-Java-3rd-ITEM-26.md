---
title: Effective Java 3rd ITEM 26
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-03 23:10:55
thumbnail:
---

# Generics 관련 용어 설명
Generic은 Java5부터 사용할 수 있게 되었다. 4장에서 계속 쓰일 용어들을 정리한 것이다.

|         한글 용어        |        영문 용어        |               예시                 |
|:------------------------:|:-----------------------:|:----------------------------------:|
| 매개변수화 타입          | Parameterized Type      | `List<String>`                     |
| 실제 타입 매개변수       | Actual Type Parameter   | `String`                           |
| 제네릭 타입              | Generic Type            | `List<E>`                          |
| 정규 타입 매개변수       | Formal Type Parameter   | `E`                                |
| 비한정적 와일드카드 타입 | Unbounded Wildcard Type | `List<?>`                          |
| 로 타입                  | Raw Type                | `List`                             |
| 한정적 타입 매개변수     | Bounded Type Parameter  | `<E extends Number>`               |
| 재귀적 타입 한정         | Recursive Type Bound    | `<T extends Comparable<T>>`        |
| 한정적 와일드카드 타입   | Bounded Wildcare Type   | `List<? extends Number>`           |
| 제네릭 메서드            | Generic Method          | `static <E> List<E> asList(E[] a)` |
| 타입 토큰                | Type Token              | `String.class`                     |

# ITEM 26. Don’t use raw types
클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 *제네릭 클래스* 혹은 *제네릭 인터페이스*라 한다.
***=> 제네릭 타입***

## Raw Type
Raw Type이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다. 제네릭을 사용하기 전 코드와 호환될 수 있도록 만든 것이다.
```java
// Stamp를 위한 Collection
private final Collection stamps = ...;
```
위 코드의 stamps는 Stamp 원소만 받고자 만든 것이지만 다른 원소가 들어가도 컴파일되고 실행된다.

```java
private final Collection<Stamp> stamps = ...;
```
제네릭을 활용하면 주석이 아닌 타입에 Stamp를 선언할 수 있다. 컴파일 타임에 다른 원소를 넣으려고 시도하는 것을 오류로 감지할 수 있다.
<br/>

*Raw Type을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다. 절대로 써서는 안된다.* 그저 하위 버전 호환성을 위해 존재할 뿐이다.
`List`와 같은 Raw Type은 사용하면 안되지만 `List<Object>`처럼 모든 타입을 허용하는 매개변수화 타입은 사용 가능하다. Raw Type과의 차이는 `List`를 매개변수로 받는 메서드에 `List<String>`은 받을 수 있지만 `List<Object>`를 받는 메서드에는 `List<String>`을 받을 수 없다. 이유는 제네릭의 하위 타입 규칙 때문이다. `List<String>`은 `List<Object>`의 하위 타입이 아니기 때문이다.
<br/> 

제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을때는 Unbounded wildcard type(`List<?>`)을 사용하면 된다.
Unbounded wildcard type에는 null을 제외한 어떤 원소도 넣을 수 없다. 이러한 제약이 불편하다면 Generic method / Bounded wildcard type을 사용하면 된다.
<br/>

Raw type을 써야하는 경우도 2가지 있다.
1. class 리터럴에는 Raw type을 사용해야 한다. 예를 들어 `List.class`를 사용하고 `List<String>.class`는 허용되지 않는다.
2. instanceof 연산자에서는 Raw type을 사용하는 것이 깔끔하다. Runtime에는 제네릭 타입에 대한 정보가 지워지므로 instanceof 연산자는 Unbounded wildcard type 이외에는 적용할 수 없다.
그리고 Raw Type이든 Unbounded wildcare type이든 똑같이 동작한다. 
