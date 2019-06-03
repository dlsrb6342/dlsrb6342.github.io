---
title: Effective Java 3rd ITEM 25
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-03 22:53:51
thumbnail:
---

# ITEM 25. Limit source files to a single top-level class

소스 파일 하나에 여러 톱레벨 클래스를 작성하지 말아야 한다.
```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

// Utensil.java
class Utensil { static final String NAME = "pan"; }
class Dessert { static final String NAME = "cake"; }

// Dessert.java
class Dessert { static final String NAME = "pie"; }
class Utensil { static final String NAME = "pot"; }
```
Main.java가 Utensil / Dessert를 참조하고 있고 Utensil.java에는 Utensil / Dessert가 선언되어있고 Dessert.java에도 Utensil / Dessert가 선언되어있다.
만약 위 코드처럼 한 소스파일에 Utensil / Dessert가 같이 선언되어있고 우연히 같은 이름의 클래스를 가지고 있다면, 컴파일 순서에 따라 main함수의 결과가 다르게 나온다.
- `javac Main.java Dessert.java`를 실행하면 Utensil class와 Dessert class가 중복 정의되었다고 컴파일 에러가 날 것이다.
- `javac Main.java`나 `javac Main.java Utensil.java`를 실행하면 `pancake`가 출력될 것이다.
- `javac Dessert.java Main.java`를 실행하면 `potpie`가 출력될 것이다.

위처럼 컴파일 순서에 따라 결과가 달라지므로 반드시 바로 잡아야한다. 
해결책은 톱레벨 클래스를 하나하나 소스파일을 분리하면 된다. 굳이 한 파일에 담고 싶다면 정적 멤버 클래스를 사용할 수 있다. 
