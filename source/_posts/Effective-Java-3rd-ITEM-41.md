---
title: Effective Java 3rd ITEM 41
categories:
  - Effective Java 3rd
  - 5. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-19 23:27:04
thumbnail:
---

# ITEM 41. Use marker interfaces to define types

Marker Interface란 아무 메서드를 담고 있지 않고 자신을 구현한 클래스가 특정 속성을 가짐을 표시하는 인터페이스를 의미한다. `Serializable` Interface가 좋은 예로, `Serializable`을 구현한 클래스는 ObjectOutputStream을 통해 쓸 수 있다, 즉 직렬화할 수 있다.

## Marker Interface가 좋은점
1. Marker Interface는 자신을 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있지만 Marker Annotation은 아니다.
2. 적용 대상을 더 정밀하게 지정할 수 있다. 클래스에만 적용하고 싶은 마커가 있다면 Marker Interface를 사용하면 된다.

## Marker Annotation이 좋은점
1. Annotation 시스템의 지원을 받는다. Annotation을 사용하는 프레임워크에서는 Marker Annotation을 사용하는 것이 일관성 유지에 좋을 것이다.

<br/>
## Marker Interface? Marker Annotation?
클래스와 인터페이스 외의 프로그램 요소에는 Marker Annotation을 사용할 수 밖에 없고, 클래스나 인터페이스에 적용하려 한다면 "이 마킹이 된 객체를 매개변수로 받은 메서드를 작성할 일이 있을까?"의 대답이 "그렇다"라면 Marker Interface를 사용하자!"
