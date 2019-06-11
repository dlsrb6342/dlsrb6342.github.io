---
title: Effective Java 3rd ITEM 33
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-11 22:26:14
thumbnail:
---

# ITEM 33. Consider typesafe heterogeneous containers

제네릭은 `Set<E> Map<K,V> ThreadLocal<T> AtomicReference<T>` 등의 단일 원소 컨테이너에도 흔히 쓰인다. 이런 상황에서는 매개변수화되는 대상은  컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수는 제한된다. 
> `Map<K,V>`에서는 K와 V 2개로 제한 / `Set<E>`에서는 E 1개로 제한.

하지만 이보다 더 유연한 수단이 필요할 때가 있다. 이럴땐 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 
***=> 타입 안전 이중 컨테이너 패턴***


## 타입 안전 이중 컨테이너

예시로 타입별로 즐겨 찾는 인스턴스를 저장하는 Favorites class를 보자.
```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public <T> T getFavoirte(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
각 타입의 Class 객체를 매개변수화 키로 사용한 것인데 이는 class 리터럴의 타입이 `Class<T>`이기 때문에 사용할 수 있는 것이다.
> 메서드들이 주고 받는 class 리터럴을 타입 토큰이라 한다.  

`favorites` 맵의 값 타입이 Object이기 때문에 `getFavoirte()` 시에 형변환이 필요하다. 따라서 여기서는 Class의 cast 메서드를 이용해 동적 형변환하였다.  

여기서 중요한 점은 Favorites의 instance들은 타입 안전하다는 것이다. `getFavorite(String.class)`를 했을때 String이 아닌 타입의 인스턴스를 반환할 일이 절대 없다.
***=> Favorites class는 타입 안전 이중 컨테이너***
<br/>
### Favorites class의 제약

#### 1. 악의적 클라이언트
```java
f.putFavorite((Class)Integer.class, "Heellllooo")
```
만약 악의적인 클라인트가 Class 객체를 제네릭이 아닌 raw type으로 넘긴다면 타입 안전성이 깨지게 된다. 위 예시에서 보듯이 raw type을 사용하면 Integer.class 타입으로 String 값을 넘겨줄 수 있게 된다.  

이것을 예방하려면 `putFavorite(type, instance)` 메서드에서 넣기전에 instance의 타입을 type과 같은지 확인해주면 된다. Collections의 `checkedSet, checkedList, checkedMap`이 이렇게 타입을 확인해주는 컬렉션 레퍼들이다.
> [Collections.CheckedCollection.class code](https://github.com/openjdk/jdk11u/blob/master/src/java.base/share/classes/java/util/Collections.java#L3038-L3148)

<br/>
#### 2. 실체화 불가 타입
Favorites 클래스에는 실체화 불가 타입을 사용할 수 없다. `List<String>` 같은 제네릭을 사용할 수 없다는 것이다. `List<String>`이나 `List<Integer>` 등등 모두 List.class로 같은 Class 객체를 공유한다. 따라서 실체화 불가 타입은 Favorites 클래스에서 사용할 수 없다.

> 슈퍼 타입 토큰..? spring framework의 ParameterizedTypeReference로 가능하긴 하지만 완벽하진 않다.

<br/>
### 한정적 타입 토큰
Favorites의 키 타입은 비한정적 타입 토큰이지만 타입을 제한하고 싶다면 한정적 타입 토큰으로 바꿀 수 있다. `Class<? extends SuperClass>`로 선언하면 된다.
반환 하는 객체는 `Class<? extends SuperClass>` 타입으로 형변환이 필요한데 이때 Class 클래스에 있는 `asSubClass` 메서드로 쉽게 형변환 해줄 수 있다. 
