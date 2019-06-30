---
title: Effective Java 3rd ITEM 55
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-30 17:50:40
thumbnail:
---

# ITEM 55. Return optionals judiciously

## Optional
java8이전에 값을 반환할 수 없을때는 null을 반환하거나 예외를 던져야 했다. 하지만 java8부터 `Optional<T>`이 추가되었고 값이 없을때는 Optional에 빈 결과를 담아 반환하면 된다.

Optional은 사용자에게 값이 없을 수도 있음을 명확히 알려주는 취지다. 
- orElse
- orElseThrow
- orElseGet
- isPresent
- ...

Optional에는 위와같이 다양한 메서드들이 있으니 값이 없을때를 대비하여 설계하자.

***Optional은 결과가 없을 수 있으며, 클라이언트가 이 상황을 따로 처리해야 할때 사용하자. 어느정도 성능 저하가 있을 수 있으므로 성능이 민감한 메서드는 null을 반환하자.***
<br/>

## Optional with Container type
Optional은 컨테이너 타입과는 사용하면 안된다. 빈 `Optional<List<T>>`를 반환하기 보다는 빈 `List<T>`를 반환하는 것이 클라이언트 입장에서 Optional 처리를 안해도 되기 때문에 더 좋다.

## Optional with Boxing type
박싱된 기본 타입을 담은 Optional은 사용하지 말자. 기본 타입을 박싱하고 다시 Optional로 감싸면 더욱 무겁기 때문이다.  `Optional<T>`가 아닌 `OptionalInt / OptionalLong / OptionalDouble`을 사용하자. 

## Optional in Collection
Optional을 컬렉션의 키 / 값 / 원소로 사용하는 것은 적절치 않다. 쓸데없이 복잡해 혼란을 주고 오류 가능성만 키울 뿐이다.
