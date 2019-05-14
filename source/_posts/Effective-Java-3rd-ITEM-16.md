---
title: Effective Java 3rd ITEM 16
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-05-14 22:43:47
thumbnail:
---

# ITEM 16. In public classes, use accessor methods, not public fields

패키지 바깥에서 접근할 수 있는 클래스라면 접근자(getter, setter)를 제공해서 클래스 내부 구현을 언제든 수정할 수 있는 유연성을 갖게 해야 한다. 
하지만 package-private / private 중첩 클래스라면 필드를 public으로 노출하는 것이 오히려 깔끔할 수 있다.
어차피 클라이언트도 내부 패키지 안에 있기 때문에 수정에 대한 유연성이 유지된다.  

public 클래스의 필드가 불변이라면 public으로 노출할 때의 단점이 줄긴 하지만 완전히 안심할수는 없다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전히 있다.
