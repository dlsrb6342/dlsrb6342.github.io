---
title: Effective Java 3rd. ITEM 3.
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-03-01 12:34:57
thumbnail:
---

# ITEM 3. Singleton with Private Constructor or Enum
함수와 같은 stateless 객체나 설상 유일해야 하는 컴포넌트를 보통 Singleton으로 만든다.
그러나 Singleton으로 만들면 mock으로 대체하기가 어려워져 테스트하기가 어려워질 수 있다.
<br/>
## Singleton 생성 방식
### 1. Private Constructor
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
}
```
static final field인 INSTANCE를 초기화할때 private constructor가 사용된다. 
private constructor 밖에 없기 때문에 다른 인스턴스를 만들 수 없다.
이 방식의 장점은 해당 클래스가 Singleton이라는 것을 명확히 API에 드러난다는 점이다.
<br/>
### 2. Static Factory Method
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }
}
```
위의 1. Private Constructor와 다른 점은 static final field인 INSTANCE를 private로 지정했다는 점이다.
그리고 static method인 getInstance()를 통해 항상 같은 인스턴스를 return하게 된다.
이 방식의 장점은 

1. API는 바꾸지 않은채 Singleton이 아니게 변경할 수 있다는 점이다. Static Factory method의 구현만 바꾸면 가능하다.
2. Static Factory method를 Generic Singleton Factory로 바꿀 수 있다(ITEM 30).
3. method reference를 Supplier로 사용할 수 있다. Elvis::getInstance를 Supplier<Elvis>로 사용(ITEM 43, ITEM 44).
<br/>

### 3. Enum
```java 
public enum Elvis {
    INSTANCE;
}
```
1번 방식과 비슷하지만 더 간결하다.
**대부분의 상황에서는 Enum 방식이 가장 좋은 방법이다.** 하지만 Singleton으로 만드려는 클래스가 Enum이 아닌 다른 클래스를 상속받아야 한다면 사용할 수 없다.
<br/>
## Serialization
위 3. Enum 방식의 경우는 추가 코드없이 직렬화가 가능하다.
하지만 1, 2 방식은 직렬화에 사용된다면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다. 
이를 해결하기 위해 readResolve method를 추가해주어야 한다.
```java
private Object readResolve() {
    return INSTANCE;
}
```

## 참고
https://stackoverflow.com/questions/1168348/java-serialization-readobject-vs-readresolve

