---
title: Effective Java 3rd ITEM 11
categories:
  - Effective Java 3rd
  - 3. Methods Common to All Objects
tags:
  - effective_java
  - java
date: 2019-04-30 17:37:44
thumbnail:
---

# ITEM 11. Always override hashCode when you override equals
equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

## hashCode 일반 규약
> 1. equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode method는 항상 같은 값을 반환해야 한다.
2. equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다를 필요는 없다. 하지만, 다른 값을 반환해야 HashTable의 성능이 좋아진다.

## hashCode 구현
이상적인 해시 함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.
#### 전형적인 hashCode
```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```
전형적인 hashCode method이다. 31은 홀수이면서 소수이기도 하고 `31 * i = (i << 5) - i` 와 같이 시프트연산과 뺄셈으로 최적화할 수 있어 사용하고 있다. 하지만 사용하는 이유는 명확하지는 않고 전통적으로 쓰고 있다.

#### Objects를 활용한 hashCode
```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```
Objects 클래스에서 임의의 객체들을 받아 hash를 생성해주는 method가 있지만 성능이 느리다.

#### 캐싱을 이용한 hashCode
```java
private int hashCode;

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```
클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 계산하지 않고 캐싱하는 방식을 고려해야 한다.

## hashCode 주의사항
성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
hashCode가 반환하는 값의 생성 규칙을 자세히 공표하지 말아야 한다. 추후 계산 방식을 바꿀 경우를 대비해야 한다.
