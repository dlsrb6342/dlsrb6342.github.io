---
title: Effective Java 3rd ITEM 69 70 71
categories:
  - Effective Java 3rd
  - 10. Exceptions
tags:
  - effective_java
  - java
date: 2019-07-14 15:20:49
thumbnail:
---

# ITEM 69. Use exceptions only for exceptional conditions
예외는 이름 그대로 오직 예외 상황에서만 상황해야 한다. 절대로 일상적인 흐름을 제어하기 위해 사용되어서는 안된다. 일부러 예외를 만들고 그 예외를 catch하는 흐름을 만들어서는 안된다.

이런 원칙은 API 설계에도 똑같이 적용할 수 있다. 클라이언트가 정상적인 흐름에서 예외를 사용할 일은 없어야 한다. 특정 상황에서만 호출 가능한 **상태 의존적 메서드**는 **상태 검사 메서드**와 함께 제공되어야 한다.
```java
// 상태 의존적 메서드
iterator.next();

// 상태 검사 메서드
iterator.hasNext();
```
이런 상태 검사 메서드 대신 null같은 특정값 혹은 Optional은 return하는 방법도 있다.

* 특정값? Optional? 상태 검사 메서드?
1. 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부요인으로 상태가 변할 수 있다면 Optional이나 특정값을 사용하자. 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 상태가 변경될 수 있기 때문이다.
2. 성능이 중요한 상황에서 상태 검사 메서드와 상태 의존적 메서드의 작업이 일부 중복된다면 Optional이나 특정값을 사용하자.
3. 1번 2번을 제외한 모든 경우에 상태 검사 메서드가 더 낫다. 가독성도 좋고 버그를 발견하기도 쉽기 때문이다.
<br/>

# ITEM 70. Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
throwable에는 검사 예외, 런타임 예외, 에러 3가지가 있다. 이중에 비검사 throwable에는 런ㅁ타임 예외와 에러가 있고 둘의 동작도 똑같다. 이 둘은 프로그램에서 잡을 필요가 없거나 잡으면 안되는 예외들이다. 또 비검사 throwable들은 모두 RuntimeException의 하위 클래스이다.

### 검사 예외? 런타임 예외? 에러?
* 호출하는 쪽에서 catch하고 처리해야하는 상황이라면 검사 예외를 사용하라.
* 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하라.
* Exception / RuntimeException / Error 모두 상속하지 않은 throwable은 사용하지 말자.
* 검사 예외는 복구할 수 있는 조건일 때 사용하자.

<br/>

# ITEM 71. Avoid unnecessary use of checked exceptions
검사 예외를 과하게 사용하면 쓰기 불편한 API가 될 수 있다. 어떤 메서드가 검사 예외를 던진다면, 그것을 호출하는 클라이언트는 catch 블록을 두고 처리하거나 바깥으로 던져주는 코드를 작성해야 한다. 결국 사용자에게 부담을 주게 된다. 

API를 제대로 사용해도 발생할 수 있는 예외이거나 의미있는 예외가 아니라면 비검사 예외를 사용하는 것이 좋다. 검사 예외 대신 Optional을 return하는 것도 방법이다. 상태 검사 메서드를 추가할 수 있지만 여러 단점이 존재해 적절하지 않을 수 있다.

**검사 예외는 프로그램의 안전성을 높여주지만 사용자에게 부담을 줄 수 있다. 비검사 예외를 사용하거나 Optional로 처리해보자**
