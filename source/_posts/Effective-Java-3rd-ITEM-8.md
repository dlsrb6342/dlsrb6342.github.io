---
title: Effective Java 3rd ITEM 8
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-04-07 20:51:32
thumbnail:
---

# ITEM 8. Avoid finalizers and cleaners
## Finalizers and Cleaners
Finalizer는 예측불가능하고 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 
Java 9에서 Finalizer의 대안으로 cleaner를 소개했지만 그또한 여전히 예측할 수 없고 느리다.
### 1. 수행 시점
객체에 접근할 수 없게 된 후, 언제 finalizer나 cleaner가 실행될지 알 수 없다.
이는 GC 알고리즘마다 천차만별이며 환경마다 다르게 동작할 수 있다.
Finalizer의 경우, 어떤 스레드가 수행할지 명시되어 있지 않아 낮은 우선순위로 인해 계속 Queue에 쌓여갈 수 있다. 

<br/>
### 2. 수행 여부
Finalizer나 Cleaner는 수행여부를 보장하지 않아 종료 작업이 전혀 수행되지 못한채 프로그램이 종료될 수 있다. 
예를 들어 DB의 lock 해제를 Finalizer나 Cleaner에게 맡겼다면 점점 시스템이 멈출 것이다ㅣ.

<br/>
### 3. 수행 중 발생한 예외
Finalizer 수행 중 발생한 예외는 무시되며 다른 작업이 남아있더라도 그 순간 종료된다.
마무리가 덜 된 상태의 객체가 남아있을 수 있어 이러한 객체를 사용하게 되면 어떻게 동작할지 예측할 수 없다. 
예외가 발생해도 Stack trace를 출력하지 않고 그대로 진행된다. Cleaner의 경우, 자신의 스레드를 통제하기 때문에 이러한 문제는 발생하지 않는다.

<br/>
### 4. 성능 문제
AutoCloseable 객체를 try-with-resources로 생성하고 수거하기까지는 12ns가 걸렸고
Finalizer를 사용해서 생성 / 수거를 하기까지는 550ns가 걸렸다. Finalizer가 GC의 효율을 떨어뜨리기 때문이다.

<br/>
### 5. 보안 문제
생성자나 직렬화 과정(readObject / readResolve)에서 예외가 발생하면 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행되게 할 수 있다.
이러한 일그러진 객체 생성을 막으려면 생성자에서 예외를 던지는 것으로 충분하지만 finalizer가 있다면 그렇지 않다.
final 클래스로 만들면 하위클래스를 만들 수 없게 되어 안전하지만 final이 아닐 경우, 빈 finalize method를 만들고 final로 선언하면 된다.

<br/>
## AutoCloseable
파일이나 자원을 담고 있는 객체는 finalizer / cleaner가 아닌 AutoCloseable을 구현해주면 된다.
예외가 발생해도 제대로 종료될 수 있도록 try-with-resources를 사용해야 한다.
