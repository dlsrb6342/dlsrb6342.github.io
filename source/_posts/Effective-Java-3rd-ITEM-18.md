---
title: Effective Java 3rd ITEM 18
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-01 15:25:45
thumbnail:
---

# ITEM 18. Favor composition over inheritance

## 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
#### 메서드 재정의에 의한 문제
상위 클래스 구현에 따라 하위 클래스의 동작이 달라질 수 있다.
상위 클래스는 릴리즈마다 내부 구현이 달라질 수 있으며, 그 여파로 하위 클래스가 오동작할 수 있다.
만약 컬렉션을 상속하여 원소 추가 시에 검증 로직이 들어간 클래스가 있다면
다음 릴리즈에 컬렉션에 원소 추가 메서드가 새로 만들어지면 검증 로직을 타지 않은 원소가 들어갈 수 있다.

#### 새로운 메서드 추가에 의한 문제
메서드 재정의보다는 안전하지만 문제가 있다.
만약 상의 클래스에 새 메서드가 추가되었는데, 하위 클래스의 새로운 메서드와 시그니처가 같고 반환 타입만 다르다면 컴파일조차 되지 않는다.

<br/>
## 상속이 아닌 컴포지션
- Composition : 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 방법.
- Forwarding : 새 클래스의 인스턴스 메서드들이 기존 클래스의 대응하는 메서드를 호출
- Wrapper Class : 기존 클래스의 인스턴스를 감싸고 있는 새로운 클래스

레퍼 클래스는 거의 단점이 없지만 콜백 프레임워크와는 어울리지 않는다.
내부 객체가 자신을 감싸고 있는 레퍼의 존재를 몰라 레퍼 인스턴스가 아닌 내부 객체를 넘겨주게 된다.

Guava에 모든 컬렉션 인터페이스용 전달 메서드를 구현해둔 클래스가 있다.

<br/>
컴포지션을 써야 할 상황에서 상속을 사용하는건 내부 구현을 불필요하게 노출한다.
예를 들어 Properties는 HashTable을 상속하고 있는데, 
`Properties.getProperty(key)`와 `Properties.get(key)` 두 가지 방법으로 value를 가져올 수 있다.
getProperty()는 Properties의 기본 동작이고 get()은 HashTable로부터 받은 메서드라 결과도 다를 수 있다.

컴포지션 대신 상속을 사용하기로 결정하기 전!
1. 하위 클래스가 정말 상위 클래스인가? 
2. 확장하려는 클래스의 API에 결함이 없는가?
3. 결함이 있다면, 이 결함이 확장 클래스의 API까지 전파돼도 괜찮은가?

