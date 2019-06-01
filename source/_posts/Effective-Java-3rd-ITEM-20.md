---
title: Effective Java 3rd ITEM 20
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-01 18:46:49
thumbnail:
---

# ITEM 20. Prefer interfaces to abstract classes
자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스이다.
이 둘의 가장 큰 차이는 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 자바는 단일 상속만 지원하기 때문에, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안게 되는 셈이다.
반면 인터페이스는 어떤 클래스를 상속했든 같은 타입으로 취급된다.


## 인터페이스의 장점
- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
새로운 인터페이스가 요구하는 메서드를 추가하고 implement만 선언하면 된다.
Comparable, Iterable, AutoCloseable 인터페이스가 추가됐을 때 표준 라이브러리의 기존 클래스가 이 인터페이스들을 구현한 채 새로 릴리즈됐다. 
반면 추상클래스를 새로 끼워넣는 일은 클래스 계층구조상 어려운 일이다.
- 인터페이스는 믹스인 정의에 안성맞춤이다.
믹스인(mixin)이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언할 수 있다.
- 기능을 향상시키는 안전하고 강력한 수단이다.
- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
계층을 엄격히 구분하기 어려운 개념일 때 인터페이스가 유용하다. 예를 들어 Singer / SongWriter가 있다.
```java
public interface Singer { AudioClip sing(Song s); }

public interface SongWriter { Song compose(int chartPosition); }

public interface SingerSongWriter {
    AudioClip sing(Song s);
    Song compose(int chartPosition);
}
```
이러한 구조를 클래스로 만들려면 가능한 조합이 2^n개가 된다.

<br/>
## 추상 골격 구현 클래스
추상 골격 구현 클래스를 제공하여 추상 클래스의 장점도 취할 수 있다.
인터페이스로는 디폴트 메서드와 타입을 정의하고 골격 구현 클래스에서 나머지 메서드를 구현한다.
이렇게 하면 골격 구현 클래스를 상속하는 것만으로 인터페이스 구현의 대부분이 완료된다. AbstractCollection들이 좋은 예이다.
인터페이스의 기반 메서드들은 골격 구현 클래스의 추상 메서드가 되고 기반 메서들을 사용해 직접 구현할 수 있는 메서드는 모두 디폴트 메서드를 제공한다.
단 equals / hashCode는 인터페이스의 디폴트 메서드로 제공하면 안되고 골격 구현 클래스에서 구현해야 한다. 
