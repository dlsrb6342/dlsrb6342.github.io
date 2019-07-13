---
title: Effective Java 3rd ITEM 63 64 65
categories:
  - Effective Java 3rd
  - 9. General Programming
tags:
  - effective_java
  - java
date: 2019-07-13 14:40:42
thumbnail:
---

# ITEM 63. Beware the performance of string concatenation
문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.
=> StringBuilder를 사용하자!

<br/>
# ITEM 64. Refer to objects by their interfaces
매개변수 반환값 변수 필드 등 전부 인터페이스 타입으로 선언하는 것이 좋다. 프로그램을 훨씬 유연하게 만들 수 있다.
적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 **덜 구체적인** 클래스를 타입으로 사용하자!

<br/>
# ITEM 65. Prefer interfaces to reflection
리플렉션(java.lang.reflect)를 사용하면 클래스의 생성자 메서드 필드에 해당하는 인스턴스를 꺼낼 수 있고 그에 해당하는 정보를 가져올 수 있다. 클래스의 거의 모든 것을 조작하고 호출할 수 있다.

### 리플렉션의 단점
1. 컴파일타임의 타입 검사의 이점을 누릴 수 없다.
2. 코드가 지저분해지고 장황해진다.
3. 성능이 떨어진다. 

컴파일 타임에 알 수 없는 클래스를 사용해야 한다면 리플렉션을 사용할 수 밖에 없을 것이다. 이 경우에도 객체 생성에만 리플렉션을 사용하고 객체를 사용할 때는 적절한 인터페이스나 상위 클래스로 사용하자.
***=> 리플렉션은 단점을 피하고 이점만 취할 수 있는 아주 제한된 형태로만 사용하자!***

