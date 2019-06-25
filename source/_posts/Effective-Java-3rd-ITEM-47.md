---
title: Effective Java 3rd ITEM 47
categories:
  - Effective Java 3rd
  - 7. Lambdas and Streams
tags:
  - effective_java
  - java
date: 2019-06-25 20:58:26
thumbnail:
---

# ITEM 47. Prefer Collection to Stream as a return type

Stream 인터페이스는 Iterable 인터페이스를 확장하지 않았기 때문에 for-each 문에서 사용할 수 없다. 
모든 API에서 Collection을 리턴한다면 Stream으로 짜려는 클라이언트가 불편하게 된다.
***=> 스트림에서만 사용한다면 Stream***
***=> 반복문 사용이 필요하다면 Iterable***
***=> 둘다 지원해야 한다면 Collection을 재정의!***

## 어댑터 사용
```java
// stream -> iterable
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

// iterable -> stream
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```
한가지 return type으로 정해져 있다면 클라이언트는 위 두가지 어댑터를 사용해야 한다.

## Collection return
컬렉션은 Iterable의 하위 타입이며 stream 메서드도 제공하기 때문에 두 요구사항을 모두 충족할 수 있다.
반환할 시퀀스가 덩치가 크지 않다면 ArrayList나 HashSet같은 표준 컬력센 구현체를 쓰는게 최선이다.
덩치가 크다면 직접 전용 컬렉션을 구현해 반환하자. 속도면에서도 어댑터 사용보다 전용 컬렉션을 사용하는것이 더 빠르다.
전용 컬렉션을 구현해서 사용한 예시이다. 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.
```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);

        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big" + s);

        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>=1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```
