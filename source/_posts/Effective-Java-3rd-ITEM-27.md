---
title: Effective Java 3rd ITEM 27
categories:
  - Effective Java 3rd
  - 5. Generics
tags:
  - effective_java
  - java
date: 2019-06-06 19:21:27
thumbnail:
---

# ITEM 27. Eliminate unchecked warnings

제네릭을 사용하다보면 비검사 경고가 많이 생기게 된다. 보통 컴파일러가 무엇이 잘못되었는지 친절하게 설명해주니 그에 따라 수정하자.
최대한 모든 비검사 경고를 제거하다보면 코드의 타입 안정성이 보장된다. 

## Suppress Warnings
만약 경고를 제거할 수는 없지만 타입이 안전하다고 확신한다면 `@SuppressWarnings("unchecked")`를 달아 경고를 숨길 수 있다.
`@SuppressWarnings`는 지역변수에도 달 수 있고 클래스 전체에도 달 수 있다. `@SuppressWarnings`을 달더라도 최대한 좁은 범위에 적용하자.
이 어노테이션은 선언에만 달 수 있기 때문에 return 문에는 못 써서 따로 변수 선언이 필요할 수 있다.

`@SuppressWarnings`을 사용할 때는 사용한 이유에 대해 항상 주석을 남겨 다른 사람에게 혼란을 주지 않아야 한다.

