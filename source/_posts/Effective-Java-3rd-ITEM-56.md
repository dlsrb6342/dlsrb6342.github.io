---
title: Effective Java 3rd ITEM 56
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-30 18:08:17
thumbnail:
---

# ITEM 56. Write doc comments for all exposed API elements

## javadoc

1. 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
  1. 비공개 클래스에도 유지보수를 위해 주석을 달자.
2. 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
  1. how가 아닌 what을 기술하자.
  2. 전제조건 / 사후조건 모두 나열.
  3. 부작용도 문서화하자.
  4. @param / @return / @throws 활용
  5. 재정의한 메서드는 @implSpec으로 어떻게 동작하는지 명시하자.
3. 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.
4. 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
5. 에너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.
6. 패키지 설명은 package-info.java에 / 모듈 설명은 module-info.java에 작성하면 된다.
7. 클래스의 스레드 안전성과 직렬화 형태도 기술해야 한다.

