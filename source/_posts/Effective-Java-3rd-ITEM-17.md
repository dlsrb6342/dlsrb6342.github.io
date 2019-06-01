---
title: Effective Java 3rd ITEM 17
categories:
  - Effective Java 3rd
  - 4. Classes and Interfaces
tags:
  - effective_java
  - java
date: 2019-06-01 14:32:00
thumbnail:
---

# ITEM 17. Minimize mutability

불변 클래스란 간단히 말해 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다.
String, Boxing Class, BigInteger, BigDecimal이 자바 라이브러리에 있는 불변 클래스이다.

### 클래스를 불변으로 만드는 5가지 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다. 클래스를 final로 선언하는 방법이 있다.
- 모든 필드를 final로 선언한다. 
- 모든 필드를 private를 선언한다.
- 자신 외에는 내부의 가변 객체에 접근할 수 없도록 한다. 생성자 / 접근자 / readObject 모두 방어적 복사를 수행하라.

<br/>
### 불변 클래스의 장점
#### 불변 객체는 단순하다.
불변 객체는 생성 시점의 상태를 파괴될 때까지 그대로 간직한다. 
불변 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않아도 영원히 불변으로 남는다. 
하지만 가변 객체는 언제 변할지 몰라 복잡한 상태에 놓일 수 있다.
#### 불변 객체는 근본적으로 Thread Safe하여 따로 동기화할 필요가 없다.
여러 Thread에서 동시에 사용해도 훼손되지 않는다. 
불변 클래스는 클래스를 Thread Safe하게 만드는 가장 쉬운 방법이다.
#### 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
예를 들어 BigInteger는 부호와 크기를 사용하는데 negate()는 크기는 그대로 사용하고 부호만 바꿔 새로운 BigInteger를 생성한다.
그 결과 새로 만든 BigInteger는 원본 인스턴스가 가리키는 내부 배열을 그대로 가진다.
```java
public class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum;
    final int[] mag;

    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }
}
```
#### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
값이 바뀌지 않는 구성요소들로만 이뤄진 객체라면 그 구조가 복잡하더라도 불변을 유지하기 수월하다.
#### 불변 객체는 그 자체로 실패 원자성을 제공한다.
실패 원자성이란 '메서드에서 예외가 발생한 후에도 그 객체는 여전히 호출 전과 같은 유효한 상태여야 한다'를 뜻한다.
불변 객체는 상태가 절대 변하지 않으니 불일치 상태에 빠질 가능성이 없다.
<br/>
### 불변클래스의 단점
값이 다르면 반드시 독립된 객체로 만들어야 한다.
값의 가짓수가 많다면 처리하는데 큰 비용이 든다.
예를 들어, 백만 비트의 BigInteger의 비트 하나를 바꾸려 한다면 하나의 비트를 바꾸기 위해 백만 비트짜리 인스턴스를 새로 만들게 된다.
이러한 연산에서 발생하는 문제에 대처하는 방법은 가변 동반 클래스를 제공하는 것이다.
예를 들어 BigInteger는 모듈러 지수같은 다단계 연산 속도를 높이기 위해 MutableBigInteger를 package-private으로 두고 있다.
String에서는 StringBuilder와 StringBuffer가 있다.
<br/>
### 클래스를 상속하지 못하게 만드는 방법.
- final Class로 선언.
- 모든 생서자를 private or package-private로 선언하고 정적 팩터리를 제공.

정적 팩터리로 구현했을 경우, 가변 동반 클래스같은 다수의 구현 클래스를 만들어 활용할 수 있고 
객체 캐싱과 같은 기능 추가에도 유연하기 때문에 이 방식이 최선일 때가 많다.
<br/>
클랙스는 꼭 필요한 경우가 아니라면 불변이어야 한다. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다. 
생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 
확실한 이유가 없다면 그 어떤 초기화 메서드도 public으로 제공해서는 안된다. 복잡성만 커지고 성능 이점은 거의 없다.
