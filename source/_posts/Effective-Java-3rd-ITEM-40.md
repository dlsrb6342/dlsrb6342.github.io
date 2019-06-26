---
title: Effective Java 3rd ITEM 40
categories:
  - Effective Java 3rd
  - 6. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-19 23:11:25
thumbnail:
---

# ITEM 40. Consistently use the Override annotation

```java
public class Bigram {
    
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

}
```
Bigram 클래스는 `equals()` 메서드를 Overriding하려 했지만 매개변수를 Object가 아닌 Bigram으로 선언해서 Overriding이 아닌 Overloading해버렸다. 만약 재정의한 메서드에 `@Override` annotation을 넣어줬더라면 컴파일러가 오류를 내줬겠지만 생략했기 때문에 올바르게 수정하기 어렵다.
<br/>
그러므로 ***상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` annotation을 달자!***
추상 클래스나 인터페이스를 생략해도 괜찮지만 모든 재정의 메서드에는 `@Override` annotation을 달아주는 것이 좋다. 
