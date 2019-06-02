---
title: Effective Java 3rd ITEM 22
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-02 16:41:07
thumbnail:
---

# ITEM 22. Use interfaces only to define types 

클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다. 인터페이스는 이 용도로만 활용해야 한다.

## 상수 인터페이스
상수 인터페이스는 인터페이스를 잘못 사용한 예이다.
클래스의 내부 구현에 해당하는 상수를 인터페이스에 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위다.

상수를 공개할 목적이라면 특정 클래스에 관련된 상수이면 그 클래스에, 열거타입으로 가능하다면 열거타입으로, 아니라면 유틸리티 클래스로 공개하는 것이 좋다.
