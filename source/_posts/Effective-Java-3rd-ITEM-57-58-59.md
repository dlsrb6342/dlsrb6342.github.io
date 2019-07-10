---
title: Effective Java 3rd ITEM 57 58 59
categories:
  - Effective Java 3rd
  - 9. General Programming
tags:
  - effective_java
  - java
date: 2019-07-10 22:10:59
thumbnail:
---

# ITEM 57. Minimize the scope of local variables

지역변수의 범위를 최소화하면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.
* 지역 변수가 가장 처음 쓰일 때 선언하자. 
* 거의 모든 지역변수는 선언과 동시에 초기화하자. 
* 메서드를 작게 유지하고 한 가지 기능에 집중하자.
<br/>

# ITEM 58. Prefer for-each loops to traditional for loops

Iterator를 사용한 for문이나 인덱스 변수로인한 for문은 코드를 지저분하게 하고 오류가 생길 가능성이 높아진다. 또 컬렉션 / 배열에 따라 코드 형태도 달라지는 문제점이 있다.

for-each문을 사용하면 이 모든 문제를 해결할 수 있다. 또한 for-each문은 사람이 손으로 최적화한 것과 같은 성능을 낸다.
<br/>

# ITEM 59. Know and use the libraries

## 표준 라이브러리 사용의 이점
* 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다. 
* 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다.
* 따로 노력하지 않아도 성능이 지속해서 개선된다.
* 기능이 점점 많아진다.
* 우리가 작성한 코드가 많은 사람에게 낯익은 코드가 된다.

java.lang / java.util / java.io 는 잘 알아두자! 특히 java.util.stream / java.util.Collections / java.util.concurrent는 능숙하게 다룰 수 있게 하자.
