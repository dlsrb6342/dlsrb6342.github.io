---
title: Effective Java 3rd ITEM 5
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-03-10 17:08:28
thumbnail:
---

# ITEM 5. Prefer dependency injection to hardwiring resources
많은 클래스가 하나 이상의 의존성(자원)을 가진다. 책에서는 맞춤법 검사를 예로 들고 있다.
SpellChecker는 Dictionary라는 의존성(자원)을 필요로 한다.

## 잘못 구현한 예
### 1. Utility Class
```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() { };
   
    // static methods...
}
```

### 2. Singleton
```java
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker() { };
    public static SpellChecker INSTANCE = new SpellChecker();
   
    // static methods...
}
```

위의 두 방식 모두 dictionary field가 고정적이다. 
다양한 사전을 적용할 필요가 있을때 사용하기 어렵다.
이렇게 사용하는 자원(사전)에 따라 동작이 달라진다면 Utility Class나 Singleton은 적합하지 않다.
<br/>

## 의존성 주입
```java
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }

    // methods...
}
```
인스턴스를 생성할 때 필요한 자원(사전)을 넘겨주는 방식이다.
의존 객체 주입 방식은 자원이 몇개이든 관계가 어떻든 잘 동작한다. 
