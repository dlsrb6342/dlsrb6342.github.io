---
title: Effective Java 3rd ITEM 4
categories:
  - Effective Java 3rd
  - 2. Objects 
tags:
  - effective_java
  - java
date: 2019-03-10 16:34:32
thumbnail:
---

# ITEM 4. Use Private Constructor for Noninstantiability
## Utility Class
> java.lang.Math / java.util.Arrays - 기본 타입값, 관련 메서드를 모아놓은 class
java.util.Collections - 특정 인터페이스를 구현하는 객체를 생성해주는 static method를 모아놓은 class
final class - final class를 상속해서 하위 클래스에 method를 넣는것이 불가능하기 때문에 final class 관련 메서드를 모아놓은 class

위의 3가지의 경우 static method와 static field로만 이루어진 Utility class를 만들게 된다.
Utility class는 인스턴스로 만들기 위해 설계된 것이 아니기 때문에 인스턴스 생성을 막을 필요가 있다.
NoArgsConstructor가 기본 생성자로 생성되기 때문에 사용자 입장에서 구분할 수 없다.

### Abstract Class
추상 클래스로 만들더라도 해당 클래스를 상속하여 인스턴스를 만들면 되기 때문에 인스턴스화를 막을 수 없다.

### Private Constructor
```java
public class Utils {
    private Utils() {
        throw new AssertionError();
    }
}
```
Private Constructor를 추가하여 자동 생성되는 기본 생성자를 안생기게 해준다.
생성자를 호출할 수 없기 때문에 인스턴스화를 막을 수 있고 AssertionError를 던져 내부에서도 사용하지 않도록 해준다.
이 방식은 상속을 불가능하게 하는 효과도 있다. 
