---
title: Effective Java 3rd ITEM 14
categories:
  - Effective Java 3rd
  - 3. Methods Common to All Objects
tags:
  - effective_java
  - java
date: 2019-05-03 20:39:03
thumbnail:
---

# ITEM 14. Consider implementing Comparable

Comparable는 단순 동치성 비교와 순서비교를 해주는 Generic interface이다. 단순 동치성 비교를 지원한다는 점에서 Object.equals()와 비슷한 부분이 있다.
한 클래스가 Comparable을 구현했다는 것은 인스턴스들이 자연적인 순서를 가진다는 것을 뜻한다.

### compareTo method 규약
> 1. 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
2. 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.
3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다. -> compareTo의 동치성 결과가 equals의 결과와 같아야 한다.

3번 규약을 지키지 않을 경우, 정렬된 컬렉션에서 의도한 동작과 다르게 동작할 수 있다.
예를 들어 HashSet은 equals를 통해 비교하지만 TreeSet은 compareTo로 비교하기 때문에 같은 Set이지만 똑같은 원소들을 넣어도 다른 동작을 할 수 있다.

만약 핵심 필드가 여러 개일 경우, 그 순서를 정해서 순서대로 비교하자.
```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(this.areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(this.prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(this.lineNum, pn.lineNum);
        }
    }
    return result;
}
```
```java
private static final Comparator<PhoneNumber> COMPARATOR = 
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                  .thenComparingInt(pn -> pn.prefix)
                  .thenComparingInt(pn -> pn.lineNum)

public it compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
핵심 필드를 하나하나 비교해서 0일 경우(같을 경우)에만 다음 필드를 비교하게 한 코드이다. 두 번째 코드는 java8 이후의 Comparator에 구현된 static 생성 메서드를 활용한 코드이다.


순서를 고려해야 하는 클래스라면 꼭 Comparable interface를 구현해서 정렬, 검색, 비교가 오작동하지 않도록 해야한다. 
