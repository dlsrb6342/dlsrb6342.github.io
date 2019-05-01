---
title: Effective Java 3rd ITEM 13
categories:
  - Effective Java 3rd
  - 3. Methods Common to All Objects
tags:
  - effective_java
  - java
date: 2019-05-01 11:58:40
thumbnail:
---

# ITEM 13. Override clone judiciously
## Cloneable
Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.
하지만 clone method는 Cloneable이 아닌 Object에 선언되어있고 그마저도 protected이다.
그래서 Cloneable을 implement 한다고 해서 clone method를 호출할 수 있는것이 아니다.
<br/>
Cloneable은 Object의 protected method인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스만 clone method를 수행할 수 있다.

강제성이 없다는 점만 제외하면 clone method는 Constructor Chaining과 비슷한 부분이 있다. 즉 clone method가 super.clone을 호출하지 않고 생성자를 호출해 얻은 인스턴스를 반환해도 문제될 것은 없지만 **해당 클래스를 상속받은 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 생성**된다. 따라서 하위 클래스에서 super.clone을 호출할 가능성이 있다면 제대로된 clone 구현이 필요하다.

하위 클래스가 없는 final class의 경우, clone method 재정의에서 안전하다. final class의 clone method에서 super.clone을 호출하지 않는다면 Cloneable을 구현할 이유도 없다.

#### 가변 객체를 가진 클래스
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
}
```
위 Stack 클래스는 elements라는 가변 객체를 참조하고 있다. 이 클래스의 clone method가 단순히 super.clone을 호출한다면 clone으로 생성된 객체의 elements와 기존 객체의 elements는 같은 배열을 참조하게 된다.
따라서 clone으로 생성된 객체의 elements를 변경하면 기존 객체도 영향을 받게 된다.
**clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**
<br/>
```java
@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = this.elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```
위 구현이 Stack의 제대로된 clone method이다. elements.clone()을 통해 elements 내부 정보 또한 복사해주어야 한다. CloneNotSupportedException은 발생할 일이 없지만 Checked Exception이기 때문에 사용하기 편하라고 handling 해주는게 좋다.

만약 elements field가 final이었다면 위 방식은 동작하지 않는다. final field에 새로운 값을 정의할 수 없기 때문이다. 이 상황에서 우린 Cloneable을 구현하기 위해 기존 구조를 바꿔야하는 문제점을 볼 수 있다.

clone을 정확히 재정의하려면 그 객체의 내부 깊숙하게 숨어있는 모든 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야한다. 내부 복사는 주로 재귀적으로 clone을 호출해 구현하지만 너무 길면 StackOverflow를 발생시킬 수 있다.
<br/>
결론은 복제를 하고 싶다면 아래와 같이 복제 생성자(변환 생성자)나 복제 팩터리(변환 팩터리)를 사용하자. 
Collection 구현체들은 대부분 복제 생성자를 가지고 있다. 
복제 생성자나 복제 팩터리로 Cloneable/clone을 충분히 대치할 수 있다. 
```java
public Yum(Yum yum) { ... };
public static Yum newInstance(Yum yum) { ... };
```
