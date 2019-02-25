---
title: Effective Java 3rd. ITEM 1.
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-02-24 14:52:10
thumbnail:
---

# ITEM 1. Static Factory method not Constructor.
## Pros
### 1. 이름을 가질 수 있다.
이름으로 반활될 객체의 특성을 제대로 설명할 수 있다.
하나의 signaure로는 Constructor는 하나밖에 만들 수 없지만 Static Factory는 이름을 가질 수 있기에 각각의 차이를 나타내는 이름으로 여러 개를 만들 수 있다.
 method signature - method name + method parameter (return type은 포함되지 않는다.)
<br/>
### 2. 호출될 때마다 인스터스를 새로 생성하지는 않아도 된다.
```
Boolean.valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
미리 만들어 놓거나 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
어느 인스턴스를 살아 있게 할지 통제할 수 있다. -> instance-controlled class
  -> 인스턴스 통제를 통해 Sigleton(ITEM 3), Noninstantiable(ITEM 4)를 만들 수 있다.
Flyweight pattern 참고.
<br/>
### 3. return 타입의 하위 타입 객체를 반환할 수 있다.
반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다.
인터페이스 기반 프레임워크! 인터페이스를 구현한 모든 클래스를 공개하는 것이 아닌 인터페이스만을 공개해 API를 작게 유지할 수 있다.

> java.util.Collections
Collection에는 45개의 class가 있지만 Collections의 Static Factory method를 통해 객체를 얻는다. 또 이렇게 얻은 객체를 인터페이스만으로 다루게 된다.

<br/>
### 4. parameter에 따라 다른 클래스의 객체를 반환할 수 있다.
다른 클래스 - return 타입의 하위 타입만 가능.

> EnumSet - EnumSet은 Static Factory method로만 인스턴스를 제공하는데 원소 개수에 따라 RegularEnumSet / JumboEnumSet을 return한다.
하지만 사용자 입장에서는 어떤 것이 return되었든 알 필요없이 EnumSet의 >하위 클래스이면 문제없이 사용할 수 있다.

<br/>
### 5. Static Factory method를 작성하는 시점에 반환할 객체의 클래스가 존재하지 않아도 된다.
Serivce Provider Framework의 근본.
Service Provider Framework는 Service Interface / Provider Registration API / Service Access API, 3가지 핵심 컴포넌트로 이뤄진다. 이외에도 Service Provider Interface가 있다.
클라이언트가 Service Access API를 사용할때 조건을 명시하여 원하는 구현체를 반환받을 수 있다.
서비스의 구현체가 Provider의 역할을 하고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제한다. 이로써 클라이언테와 구현체를 분리해준다.

> JDBC - Connection(Service Interface) / DriverManager.registerDriver(Provider Registration API) / DriverManager.getConnection(Service Access API) / Drivcer(Service Provider Interface)

<br/>
## Cons
### 1. 상속 시에 필요한 public / protected 생성자가 없어 하위 클래스를 만들 수 없다.
이 제약은 상속보다는 Composition(ITEM 18) / Immutable(IMEM 17)을 사용하도록 유도하여 장점일 수 있다.
<br/>
### 2. Static Factory method는 프로그래머가 찾기 어렵다.
API 문서를 잘 쓰고 method 이름도 규약에 맞게 작성하는게 좋다.
