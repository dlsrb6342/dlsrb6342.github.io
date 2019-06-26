---
title: Effective Java 3rd ITEM 34
categories:
  - Effective Java 3rd
  - 6. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-13 21:46:03
thumbnail:
---

# ITEM 34. Use enums instead of int constants

## 정수 열거 패턴
### 1. 타입 안전성
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
정수 열거 패턴은 타입 안전을 보장하지 않는다. 오렌지용 상수와 사과용 상수를 서로 잘못 쓰더라도 컴파일러가 경고해주지 않는다. 오렌지를 건네야 할 메서드에 사과가 들어갈 수도 있고 동등 연산자(==)로 비교해도 컴파일러가 잡아주지 못한다.

### 2. 상수의 이름
자바가 정수 열거 패턴을 위한 별도 namespace를 지원하지 않기 때문에 접두어로 구분해야 한다. 예를 들어 수은(원소) / 수성(행성)은 둘다 영어로 mercury이기 때문에 접두어를 붙여서 ELEMENT_MERCURY / PLANET_MERCURY로 구분해야 한다.

### 3. 깨지기 쉽다
평범한 정수 상수를 나열한 것이라 컴파일하면 그 값이 클라이언트 파일에도 그대로 새겨진다. 만약 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다. 하지 않을 경우, 엉뚱한 동작을 하게 될 것이다.

### 4. 문자열 출력
정수 상수는 문자열로 출력하기 까다롭다. 출력하거나 디버거로 보면 그저 숫자로만 보이기 때문에 tracking에 도움이 되기 어렵다. 또 같은 상수 그룹 내에서 순회를 하거나 갯수를 알 수 있는 방법이 없다.

### 5. 문자열 상수
정수 상수가 출력이 어려운 점을 보완하기 위해 문자열 열거 패턴을 사용하기도 하지만 문자열 열거 패턴이 더 나쁘다. 상수의 의미는 볼 수 있지만 문자열 상수의 이름 대신 문자열 값을 그대로 사용하게 만들수도 있다. 이렇게 문자열 값 그대로 쓴 코드는 오타를 확인할 수도 없어 자연스레 런타임 버그로 이어진다. 문자열 비교에 따른 성능 저하도 있다.

## 열거 타입
자바의 열거 타입은 다른 언어의 (정숫값만 가지는) 열거 타입과 다르게 완전한 형태의 클래스이다. 열거 타입 자체는 클래스이며 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 가지고 있는 것이다. 열거 타입은 외부에서 접근 가능한 생성자를 제공하지 않아 인스턴스가 딱 하나씩만 존재한다. 싱글턴은 상수(원소)가 하나뿐인 열거 타입이라 할 수 있고 열거타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

### 장점
- 타입 안정성을 보장한다.
- 각각의 namespace를 가지고 있어 이름이 같은 상수도 공존 가능하다.
- 필드의 이름일뿐이라 클라이언트로 컴파일되어 각인되지 않는다.
- `toString` 메서드가 출력하기에 적합한 문자열을 내어준다.
- 임의의 메서드나 필드를 추가할 수 있다. 대신 열거 타입은 불변이기 때문에 필드도 final이어야 한다.
- interface 구현이 가능하다.
- 상수 하나를 제거해도 문제가 없다. 클라이언트에서는 다시 컴파일하거나 런타임 중에 에러가 발생할 수 있지만 유용한 메시지를 출력할 것이다.
- 추상 메서드를 선언하고 상수별로 다르게 동작하는 코드를 구현할 수 있다. -> 상수별 메서드 구현
- 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환하는 `valueOf(String)` 메서드가 자동 생성된다. 
  - 만약 `toString()`을 재정의했다면 `fromString(String)`도 정의해주면 좋다.  
  모든 열거 타입에서 사용 가능한 `fromString(String)` code이다.

```java
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
            toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

### 단점
- 열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수의 변수뿐이다.
  - 열거 타입의 생성자가 실행되는 시점에는 정적 필드들이 초기화되기 전이기 때문이다.
  - 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다.
- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다.
- switch 문으로 열거 타입의 상수별 동작을 구현하는 것은 적합하지 않다.
  - switch 문을 사용하면 간결하지만 관리 관점에서 위험하다. 새로운 값이 추가되었을때 잊지 말고 case 문을 추가해주어야 한다.
  - 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.
    - 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 이 방식을 적용하는 게 좋다.
    - 종종 쓰이지만 열거 타입 안에 포함할 만큼 유용하지 않은 경우도 마찬가지다.

### 열거 타입은 언제?
**필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 물론 열거 타입에 정의한 상수 개수가 영원히 고정 불변일 필요는 없다. 상수가 추가돼도 잘 호환되도록 설계되어있다.**
