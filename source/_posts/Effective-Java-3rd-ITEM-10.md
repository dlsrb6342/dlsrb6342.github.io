---
title: Effective Java 3rd ITEM 10
categories:
  - Effective Java 3rd
  - 3. Methods Common to All Objects
tags:
  - effective_java
  - java
date: 2019-04-14 18:40:16
thumbnail:
---

# ITEM 10. Obey the general contract when overriding equals
equals 규약을 어길 경우, 그 객체를 사용하는 다른 객체들(ex, Collections)에서 어떻게 동작할지 예측할 수 없다.

## equals를 재정의하지 않는 것이 최선인 상황
#### 1. 인스턴스가 본질적으로 고유할 때
값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스. 
Thread가 대표적인 예시로 Object의 equals를 쓰는게 옳다.
<br/>
#### 2. 논리적 동치성(logical equality)를 검사할 일이 없을 때
인스턴스 내부의 논리성이 같은지 검사할 필요가 없을 경우에는 Object의 equals만으로도 충분하다.
<br/>
#### 3. 상위 클래스의 equals로 가능할 때
Set -> AbstractSet / List -> AbstractList / Map -> AbstractList
<br/>
#### 4. 클래스가 private / package-private이고 equals를 호출할 일이 없을 때
<br/>
#### 5. 값이 같은 인스턴스가 하나만 만들어질 때
싱글톤 패턴이나 Enum의 경우와 같이 통제되어있는 상황에서는 equals를 정의할 필요가 없다.
<br/>

## equals를 재정의해야 하는 상황
#### 1. 논리적 동치성을 비교해야 할 때
주로 값 클래스(Value Object)일 경우가 논리적 동치성을 비교해야 할 상황이 많다.
객체가 같은지를 비교하는 것이 아니라 값이 같은지 확인해야 하기 때문이다.
<br/>
#### 2. Map의 key / Set의 원소로 사용하고 싶을 때
equals를 활용해서 원소를 확인하기 때문에 equals에 대한 정의가 필요하다.
```java
/**
 * Implements Map.get and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
<br/>
## 동치관계
#### 1. 반사성(reflexivity)
null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true이다.
<br/>
#### 2. 대칭성(symmetry)
null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)와 y.equals(x)의 값은 같다.
<br/>
#### 3. 추이성(transitivity)
null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y) = true이고 y.equals(z) = true이면 x.equals(z) = true이다.
상위 클래스가 구체 클래스인 상황에서 하위 클래스에서 새로운 필드를 추가하게 되는 경우 추이성을 어길 수도 있다.
상위 클래스가 추상 클래스인 경우에는 이런 일이 발생하지 않는다.
<br/>
#### 4. 일관성(consistency)
null이 아닌 모든 참조 값 x, y에 대해 x, y에 대한 수정이 없다면 x.equals(y) 값은 변하지 않는다.
<br/>
#### 5. NonNull
null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 항상 false이다.
<br/>

## 양질의 equals 구현 방법
#### 1. == 연산자로 자기 자신의 참조인지 확인한다.
성능 최적화용이다.
<br/>
#### 2. instanceof로 올바른 타입인지 확인한다.
instanceof 사용 시, null check가 따로 필요없다.
<br/>
#### 3. 입력을 올바른 타입으로 형변환한다.
<br/>
#### 4. 핵심 필드들이 모두 일치하는지 하나 하나 검사한다.
기본 타입은 == 연산자로, float / double은 Float.compare() / Double.compare()로, 배열은 Arrays.equals를 사용하자.
<br/>

## equals 구현 시 주의사항
#### 1. equals를 재정의할 땐 hashCode도 반드시 재정의하자.
<br/>
#### 2. 너무 복잡하게 해결하려 들지 말자.
<br/>
#### 3. Object 외의 타입을 매개변수로 받는 equals method는 선언하지 말자.
```java
// Bad
public boolean equals(MyClass o) {
    ...
}
```
<br/>

## 참고
https://projectlombok.org/features/EqualsAndHashCode
