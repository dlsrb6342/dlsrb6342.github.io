---
title: Effective Java 3rd ITEM 19
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-01 17:15:57
thumbnail:
---

# ITEM 19. Design and document for inheritance or else prohibit it
## 상속을 고려한 설계와 문서화
상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
클래스의 공개된 메서드에서 자신의 또 다른 메서드를 호출할 수도 있다.
이 호출되는 메서드가 재정의 가능 메서드라면 이 사실을 API 설명에 적시해야 한다.
<br/>
문서로 남기는 것 뿐만 아니라 효율적인 하위 클래스 작성을 위해 클래스 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
protected로 노출할 메서드를 결정하는 것은 실제 하위 클래스를 만들어 시험해보는 것이 가장 좋다.
꼭 필요한 protected 멤버를 찾을 수 있고 혹은 필요하지 않지만 protected로 선언된 멤버도 찾을 수 있다.
그러므로 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.
<br/>
상속용 클래스의 생성자는 직접/간접적으로 재정의 가능 메서드를 호출해서는 안된다.
만약 하위 클래스 생성자보다 먼저 재정의 가능 메서드가 호출되면 의도대로 동작하지 않을 수 있다.
<br/>
Cloneable과 Serializable 인터페이스를 구현한 클래스는 상속할 수 없게 하는것이 일반적이다.
이런 클래스를 상속하는 것은 프로그래머에게 큰 부담일 수 있다.
이 클래스를 상속하는 하위 클래스에서 이 인터페이스들을 구현하게 하는 방법도 있다.
위 인터페이스 구현 시, clone / readObject 도 생성자와 비슷한 효과를 낸다.
그러므로 clone / readObject도 생성와 같이 직접/간접적으로 재정의 가능 메서드를 호출해서는 안된다.
또 Serializable 구현한 클래스의 readResolve / writeReplace는 private가 아닌 protected이어야 한다.
private일 경우, 하위 클래스에서 무시되기 때문이다.
<br/>
상속용으로 설계되지 않은 클래스는 상속을 금지해야 한다. (ITEM 18 참고)

