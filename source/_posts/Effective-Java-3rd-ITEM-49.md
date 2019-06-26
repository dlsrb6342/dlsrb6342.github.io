---
title: Effective Java 3rd ITEM 49
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-26 19:38:46
thumbnail:
---

# ITEM 49. Check parameters for validity

메서드와 생성자는 대부분 매개변수들이 특정 조건을 만족하기를 바란다. 
따라서 ***메서드 로직이 실행되기 전에, 즉 메서드의 시작 부분에서 매개변수의 조건을 확인한다면 잘못된 값이 넘어왔을 때 깔끔하게 예외를 던질 수 있다.***
> 오류는 가능한 한 빨리 잡아야 한다.

<br/>
### public / protected method
public / protected 메서드는 던지는 예외에 대해 문서화해야 한다. 어떤 이유로 어떤 예외를 던지는지 javadoc의 @throws 태그를 사용해 작성해주어야 한다. 
```java
/*
* ...
* @throws ArithmeticException if m is less than or equal to 0
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("Modulus <= 0: " + m);
    ... // Do the computation
}
```

#### Objects.requireNonNull

java7에 추가된 `Objects.requireNonNull`을 사용하면 null 검사를 수동으로 하지 않아도 된다. 원하는 예외 메시지도 작성할 수 있고 입력 값을 그대로 반환하므로 값을 사용하는 동시에 검사도 가능하다.
```java
this.strategy = Objects.requireNonNull(strategy, "strategy");
```
java9에서 `checkFromIndexSize, checkFromToIndex, checkIndex`도 추가되었는데 예외 메시지 지정은 할 수 없고 리스트와 배열에서만 사용 가능하다.

<br/>
### private method
assert문으로 매개변수 유효성을 검증할 수 있다.
```java
// Private helper function for a recursive sort
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // Do the computation
}
```
#### 특징
1. 실패 시 AssertionError를 던진다.
2. 런타임에 아무런 효과가 없고 성능 저하도 없다. 런타임에 영향을 주려면 실행 시에 -ea / --enableassertions를 넣어주어야 한다.


