---
title: Effective Java 3rd ITEM 21
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-02 16:13:52
thumbnail:
---

# ITEM 21. Design interfaces for posterity
Java8에 와서 Interface에 Default method를 추가해 인터페이스에 메서드를 추가하는 방법이 생겼지만 이 방법도 안전하진 않다.
Java7 이전의 구현 클래스들은 인터페이스에 메서드가 추가될 일은 영원히 없다라는 가정 하에 작성되었다. 그렇기 때문에 Default method를 선언하면 해당 method를 재정의하지 않은 모든 구현 클래스에서 Default method가 쓰이게 된다. 이 과정에서 모든 구현 클래스에 Default method가 아무 문제없이 적용될거라는 보장이 없다.

## Java Collection
Java8의 Collection에는 Lambda를 활용하기 위한 Default method가 다수 추가되었다.
이중 `removeIf()`는 Predicate를 받아 모든 원소를 test해서 true이면 삭제하는 Collection의 Default method이다. 범용적으로 잘 구현된 코드이지만 apache commons-collections의 SynchronizedCollection의 동기화 문제를 해결해주지 못한다.
SynchronizedCollection은 모든 메서드를 호출할 때 lock을 통해 동기화 관리를 해주고 있지만 `removeIf()`의 Default method를 사용하면 동기화를 해주지 못한다.
```java
/**
 * @since 4.4
*/
@Override
public boolean removeIf(final Predicate<? super E> filter) {
    synchronized (lock) {
        return decorated().removeIf(filter);
    }
}
```
현재 4.3까지 release되어있지만 4.4부터 `removeIf()`를 구현해두었다.
<br/>
## Interface 설계
Default method를 기존 인터페이스에 추가하는 일은 꼭 필요하지 않다면 피해야 한다. Default method는 기존 구현체에 런타임 오류를 일으킬 수도 있고 기존 구현 클래스와 충돌할 수 있기 때문이다.
인터페이스를 설계할 때는 Default method가 생겼더라도 세심한 주의를 기울이고 최소한 3가지는 직접 구현해봐야 결함을 찾아낼 수 있다.
생각할 수 있는 모든 상황에서 불변식을 해치지 않는 Default method를 만들기는 어려운 법이다.
