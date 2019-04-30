---
title: Effective Java 3rd ITEM 12
categories:
  - Effective Java 3rd
  - 3. Methods Common to All Objects
tags:
  - effective_java
  - java
date: 2019-04-30 18:26:31
thumbnail:
---

# ITEM 12. Always override toString

## toSting 규약
> 1. 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환하라.
2. 모든 하위 클래스에서 이 메서드를 재정의하라.
3. equals와 hashCode만큼 중요하진 않지만 잘 구현하면 사용하기 좋고 디버깅이 쉽다.
4. 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
5. 포맷을 명시하든 아니든 의도는 명확히 밝혀야 한다.
6. toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
