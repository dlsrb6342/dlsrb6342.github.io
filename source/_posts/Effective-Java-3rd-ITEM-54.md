---
title: Effective Java 3rd ITEM 54
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-30 17:32:38
thumbnail:
---

# ITEM 54. Return empty collections or arrays, not nulls

```java
private final List<Cheese> cheesesInStock = ...;

/**
* @return a list containing all of the cheeses in the
shop,
* or null if no cheeses are available for purchase.
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}

List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && ... ) {
    ...
}
```
`getCheeses()` 메서드가 null을 반환하기 때문에 그 메서드를 사용하는 클라이언트는 항상 null check를 해주어야 한다. 이때문에 코드도 더 복잡해지고 만약 check하지 않았다면 오류가 발생할 수도 있다.
***=> null 대신 empty Collection을 반환하자***

Empty Collection을 반환하는 것이 성능을 떨어뜨릴 수 있다고 얘기하지만 사실이 아니다.
1. 이정도의 성능 차이는 신경 쓸 수준이 못 된다.
2. 새로 빈 컬렉션을 할당하지 않고도 반환할 수 있다.

`new ArrayList<>(cheesesInStock); / Collections.emptyList();` 와 같이 사용한다면 성능 저하는 없을 것이다. 
<br/>
배열의 경우도 마찬가지이다. null보다는 길이 0짜리 배열을 반환하자. 
```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```
매번 `new Cheese[0]`하는 것이 성능을 떨어뜨릴 것 같다면 길이가 0인 배열을 미리 선언해두고 사용하자.
