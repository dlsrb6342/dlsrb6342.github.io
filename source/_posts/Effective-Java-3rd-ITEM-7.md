---
title: Effective Java 3rd ITEM 7.
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-03-23 14:42:04
thumbnail:
---

# ITEM 7. Eliminate obsolete object references

## 직접 메모리를 관리하는 경우
```java
public class Stack {
    private Object[] elements;
    private int size = 0;

    public void push(Object e) {
        // stack이 꽉 찼을 경우, 공간을 2배 증가시킨다.
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
```
직접 메모리를 관리하는 경우에는 GC 입장에서 어떤 객체가 활성영역인지 비활성영역인지 알 길이 없다.
비활성영역 객체가 하나 살아있게 될 경우, 그 객체를 참조하는 모든 객체들 또한 GC에서 처리할 수 없게 되어 메모리 누수가 발생할 수 있다.
따라서 직접 메모리를 관리할 때는 사용하지 않는 객체에 대해서는 null 처리를 해야 한다. 
위의 Stack 예시에서는 직접 elements라는 Object 배열을 관리하고 있다. pop()을 호출했을 때, elements의 마지막에서 객체 하나를 꺼내 반환하고 있다. 
하지만 null 처리를 하지않아 elements 배열에 아직 들어있는 상태이므로 마지막에 있던 원소가 더 이상 사용하지 않는 상태인지 GC 입장에서 판단할 수가 없다. 
```java
Object result = elements[--size];
elements[size] = null;
return result;
```
따라서 위 코드처럼 마지막 element를 꺼내고 null 처리해야 메모리누수를 방지할 수 있을 것으로 보인다.
<br/>

## 캐시
객체 참조를 캐시에 넣어두고 잊는 경우 메모리 누수의 위험이 있다. 
외부에서 키에 대한 참조가 살아있는 동안에만 캐시가 필요하다면 WeakHashMap을 사용하도록 권장한다. 
쓰지 않은 캐시 엔트리에 대한 정리는 Background Theard로 정리하거나 LinkedHashMap.removeEldestEntry 같은 새 엔트리를 추가할 때 정리하는 방식이 있다.
<br/>

## Listner or Callback
Callback을 등록만 하고 명확히 해지하지 않는다면 계속 쌓일 것이다.
이 경우에도 Callback을 weak reference로 저장하면 해결할 수 있다. (WeakHashMap)
<br/>

## 참고
GC Reference https://gogorchg.tistory.com/entry/Java-WeakReference-SoftReferernce-StrongReference
WeakHashMap https://anyuneed.tistory.com/entry/JAVA-WeakHashMap%EC%9D%98-%EC%82%AC%EC%9A%A9
