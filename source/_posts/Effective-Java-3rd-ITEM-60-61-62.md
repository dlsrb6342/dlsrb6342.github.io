---
title: Effective Java 3rd ITEM 60 61 62
categories:
  - Effective Java 3rd
  - 9. General Programming
tags:
  - effective_java
  - java
date: 2019-07-10 22:44:50
thumbnail:
---

# ITEM 60. Avoid float and double if exact answers are required
float와 double은 부동소수점 연산에 쓰이고 과학과 공학 계산용으로 설계되었다. 따라서 정확한 결과가 필요할 때는 적절치 않다. 따라서 올바른 값을 얻기 위해서는 BigDecimal이나 int 혹은 long을 사용해야 한다. 
### BigDecimal
BigDecimal은 정확한 값을 얻을 수 있지만 기본 타입보다 쓰기 불편하고 느리다. 그 대안으로 int 혹은 long을 쓸 수 있다. 

### int 혹은 long
int 혹은 long은 정확한 값을 얻을 수 있고 기본 타입이라 편하고 속도도 좋다. 하지만 값의 크기가 제한되고 소수점을 직접 관리해야 한다. 
<br/>

# ITEM 61. Prefer primitive types to boxed primitives

### 기본 타입과 박싱된 기본 타입의 차이
* 기본 타입은 값만 / 박싱된 기본 타입은 값과 식별성을 갖는다.
* 기본 타입의 값은 언제나 유효 / 박싱된 기본 타입은 null을 가질 수 있다.
* 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

### 주의할점
* 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다.
* 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 오토언박싱이 일어나 NPE가 발생할 수 있다.
* 박싱과 언박싱이 반복해서 일어나면 성능이 굉장히 느려진다.

<br/>

# ITEM 62. Avoid strings where other types are more appropriate

* 문자열은 다른 값 타입(boolean / int / long ...)을 대신하기에 적합하지 않다. 
* 문자열은 열거 타입을 대신하기에 적합하지 않다.
* 문자열은 혼합 타입을 대신하기에 적합하지 않다. 
* 문자열은 권한을 표현하기에 적합하지 않다.
