---
title: Effective Java 3rd ITEM 6
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-03-11 21:42:43
thumbnail:
---

# ITEM 6. Avoid creating unnecessary objects.
## String
```java
String s1 = new String("bikini");
String s2 = "bikini";
```
위의 s1 / s2 모두 같은 결과를 가져오지만 
s1의 경우, 호출될 때마다 String 인스턴스 객체를 계속 생성하게 된다. 
s2의 경우에는, 똑같은 문자열 리터럴을 사용하는 경우 객체 재사용이 보장된다.
<br/>

## Static Factory Method
```java
Boolean b1 = new Boolean("true");
Boolean b2 = Boolean.valueOf("true");
```

위 String의 경우와 비슷한 예제이다. b1은 항상 새로운 객체를 생성하게 되고
b2는 static factory method를 사용하기 때문에 같은 객체를 사용하게 된다.
b1에서 사용한 생성자는 java 9에서 deprecated되었다.

## 생성 비용이 높은 객체
String을 정규표현식으로 확인하는 방법은 String.matches method가 있다.
하지만 이 method는 내부적으로 호출될 때마다 Pattern 인스턴스를 새로 생성하게 된다.
Pattern은 인스턴스 생성 비용이 높기 때문에 Pattern 인스턴스를 직접 생성해두고 재사용하는 것이 좋다.
```java
public class RegexExample {
    private static final Pattern PATTERN = Pattern.compile("...");
    
    public static boolean isMatch(String s) {
        return PATTERN.matcher(s).matches();
    }
}
```

## Adaptor(?)
Adaptor는 뒷단 객체만 관리하면 되므로 뒷단 객체 하나당 Adaptor 하나를 가지게 된다.
예를 들어, Map 인터페이스의 keySet method는 Map의 모든 key를 Set에 담아 return한다. 여기서 Map이 뒷단 객체, key가 담긴 Set이 Adaptor를 의미한다.
keySet이 호출될때마다 매번 같은 Set 인스턴스를 return한다. 모두가 똑같은 Map 인터페이스를 대변하기 때문에 매번 같은 인스턴스를 return하는 것이다.
물론 keySet이 계속 다른 객체를 반환해도 되지만, 그럴 필요도 이득도 없다.
<br/>
{% asset_img "adaptor.png" "Adaptor Pattern" %}
Adaptor는 Gamma95 design pattern과 관련된 내용이다. 정확히 다 이해하지 못했다.
<br/>

## Auto Boxing
```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) sum += i;

    return sum;
}
```
위의 과정에서 sum 변수가 Long(Wrapper Class)로 선언되었기 때문에 sum += i;가 호출될 때마다 i를 Long으로 Auto Boxing하게 된다. 
**Wrapper Class가 아닌 primitive type을 사용하여 의도치 않은 Auto Boxing을 주의해야 한다.**
<br/>

## 참고
ITEM 60의 방어적 복사와 비교.

