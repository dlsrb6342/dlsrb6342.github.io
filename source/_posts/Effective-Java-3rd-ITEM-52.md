---
title: Effective Java 3rd ITEM 52
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-30 16:43:58
thumbnail:
---

# ITEM 52. Use overloading judiciously

## Overloading으로 인한 혼란
```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

위 코드는 `"Set", "List", "Unknown Collection"`을 출력하길 기대하고 작성된 코드지만 결과는 모두 `"Unknown Collection"`을 출력한다. 그 이유는 ***다중 정의된 세 classify 중 어느 메서들를 호출할지가 컴파일 타임에 정해지기 때문이다.*** 컴파일 타임에 for문의 `c`는 모두 Collection 타입이다. 따라서 모두 `"Unknown Collection"`을 출력하는 것이다.

Overriding 메서드는 동적으로(런타임에) 선택되고  Overloading한 메서드는 정적으로(컴파일 타임에) 선택된다. Overloading한 메서드는 객체의 런타임 타입은 전혀 중요하지 않고 컴파일 타임의 타입에 의해 선택된다. 

위 CollectionClassifier처럼 다중정의가 혼동을 일으키는 상황은 피해야 한다. 안전하고 보수적으로 하려면 ***매개변수 수가 같은 다중정의는 만들지 말고 가변인수를 사용하는 메서드에서는 다중정의를 아예 사용하지 말아야 한다.***

## Overloading의 대안
1. 메서드 이름을 다르게 지어주자.
ObjectOutputStream 클래스를 보면 다중정의를 하는 대신 writeBoolean / writeInt / writeLong 같이 이름으로 나누어주었다. readBoolean / readInt / readLong 과 짝을 맞추기도 좋다.

2. 정적 팩터리
생성자의 경우 이름을 다르게 할 수 없기 때문에 여러 생성자가 있다면 다중정의가 된다. 이럴 경우엔 정적 팩터리 메서드를 사용하면 이름을 구분할 수 있다.

