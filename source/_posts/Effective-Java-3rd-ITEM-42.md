---
title: Effective Java 3rd ITEM 42
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-19 23:50:15
thumbnail:
---

# ITEM 42. Prefer lambdas to anonymous classes

## 익명 클래스
```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
})
```
문자열 리스트를 길이순으로 정렬하는 코드이다. 이 코드에서 Comparator 인터페이스를 익명클래스로 구현했다.  하지만 익명 클래스는 코드가 너무 길어 자바는 함수형 프로그래밍에 적합하지 않았다.

## 람다
Java8에 들어오면서 추상 메서드를 하나만 가지고 있는 인터페이스를 함수형 인터페이스라 불리고 람다식을 사용해 만들 수 있게 되었다. 
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
Collections.sort(words, comparingInt(String::length));
words.sort(comparingInt(String::length));
```
위 익명클래스를 람다로 바꾼 코드이다. 간결하고 명확하다. s1, s2에 대해서는 컴파일러가 타입을 추론해주기 때문에 프로그래머가 굳이 알 필요가 없고 명시해줄 필요도 없다. 타입을 명시해야 코드가 명확해질 때만 제외하고는 람다의 모든 매개변수 타입은 생략하자.
<br/>
### 주의할점
람다는 이름이 없고 문서화도 할 수 없다. 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 3줄 이상이면 람다를 쓰지 말아야 한다. 
람다는 함수형 인터페이스에서만 가능하고 추상 클래스나 추상 메서드가 여러개인 인터페이스를 구현할때는 익명 클래스를 사용해야 한다.
또 람다는 자기 자신을 참조할 수가 없다. 람다 내에서의 this 키워드는 바깥 인스턴스를 가리킨다. 만약 자신을 참조해야 한다면 익명 클래스를 써야 한다.
람다는 직렬화 형태가 구현별로 다를 수 있기 때문에 직렬화하는 일은 삼가야 한다. 직렬화가 필요한 함수 객체가 있다면 private static 중첩 클래스를 사용하자.
