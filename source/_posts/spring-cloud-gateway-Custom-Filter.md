---
title: spring-cloud-gateway Custom Filter
categories:
  - spring-cloud
  - spring-cloud-gateway
tags:
  - spring
  - spring-cloud
  - java
date: 2019-06-30 23:01:11
thumbnail:
---

# spring-cloud-gateway Custom Filter

spring-cloud-gateway에서 Custom Filter 추가하는 방법에 대해 정리해보았다.

## Add shortcutField
spring-cloud-gateway를 쓰면서 정말 편하다고 생각했던 부분은 yml에도 custom으로 추가한 Filter나 Predicate를 쓸 수 있다는 점이다.
`**GatewayFilterFactory` / `**RoutePredicateFactory`라고 이름을 지어줬다면 postfix를 제거한 이름으로 yml에서 사용할 수 있다. 물론 Bean으로 등록해줘야 한다.

여기서 더 발전하여 custom Factory에서도 shortcut field를 사용할 수 있다.
```yaml
# shortcut field
filters:
  - StripPrefix=2

# without shortcut field
filters:
  - name: StripPrefix
    args:
      parts: 2
```
위 yaml은 기본구현체인 StripPrefixGatewayFilterFactory를 shortcut을 쓴 예시와 안쓴 예시의 설정이다. shortcut field를 쓰면 조금 더 간단하게 설정을 할 수가 있다.
이를 구현하는것은 간단하다.
```java
public class CustomGatewayFilterFactory 
        extends AbstractGatewayFilterFactory<Config> {
    public static final String FOO_KEY = "foo";
    public static final String BAR_KEY = "bar";

    @Override
    public List<String> shortcutFieldOrder() {
        return List.of(FOO_KEY, BAR_KEY);
    }

    public static class Config {
        private String foo;
        private String bar;
    }
}

// Custom=foo_value,bar_value
```
`shortcutFieldOrder()`를 override해주면 된다. 반환값은 Config class의 field의 이름들을 담은 리스트이다. 마지막줄의 주석처럼 yaml 설정 파일에서 foo의 값, bar의 값을 순서대로 넣어주면 된다!
shortcut field를 사용하면 간단해서 좋긴 하지만, 그 값이 무슨 field를 뜻하는지 한눈에 보기 어려운 단점이 있는듯하다. field가 하나이거나 의미가 명확한 경우 사용하면 좋을 것 같다.

## Add Order
spring-cloud-gateway에서는 filter들에게 순서를 줄 수 있다. 명시해주지 않는다면 yaml 설정에 적어준 순서대로 0, 1, 2, ... 의 order 값을 받게 된다. 
```java
// RouteDefinitionRouteLocator.java
ArrayList<GatewayFilter> ordered = new ArrayList<>(filters.size());
for (int i = 0; i < filters.size(); i++) {
	GatewayFilter gatewayFilter = filters.get(i);
	if (gatewayFilter instanceof Ordered) {
		ordered.add(gatewayFilter);
	}
	else {
		ordered.add(new OrderedGatewayFilter(gatewayFilter, i + 1));
	}
}
```

### @Order / Ordered
custom filter에 순서를 넣어주고자 할때 가장 먼저 드는 생각은 `@Order` annotation이나 `Ordered`를 implement하는 것일 것이다.
```java
@Order(100)
public class CustomGatewayFilterFactory { }

public class CustomGatewayFilterFactory implement Ordered { }
```
하지만 이런 방법은 filter에 순서를 넣어주지 못한다. Filter가 아닌 FilterFactory에게 순서를 넣어줬을 뿐이다. **따라서 이를 위해 spring-cloud-gateway가 지원하는 `OrderedGatewayFilter`를 사용해야 한다.**

### OrderedGatewayFilter
```java
public class CustomGatewayFilterFactory 
        extends AbstractGatewayFilterFactory<Config> {
    @Override
    public GatewayFilter apply(Config config) {
        return new OrderedGatewayFilter((exchange, chain) -> {
            // ...
            return chain.filter(exchange);
        }, Ordered.HIGHEST_PRECEDENCE);
    }
}
```
위 예시는 가장 높은 우선순위를 주어 구현한 OrderedGatewayFilter이다. 이 GatewayFilterFacotyr를 사용하면 가장 먼저 이 filter를 타게 된다.
사실 난 처음엔 `OrderedGatewayFilter`의 존재를 몰랐고 왜 GatewayFilterFactory의 Order를 받지 않는지 의문이 생겼었다. 그래서 spring-cloud-gateway에 [issue](https://github.com/spring-cloud/spring-cloud-gateway/issues/1122)를 남겼었다. 올리고나서 OrderedGatewayFilter를 발견했고 코멘트로도 이것을 사용하라는 가이드를 받았다.

## Sample Source Code
https://github.com/dlsrb6342/spring-cloud-gateway-custom-filter
ShortCut field와 OrderedGatewayFilter를 사용한 sample custom filter code이다. 가장 높은 우선순위를 준 filter가 설정에 상관없이 항상 먼저 실행되는 것을 확인할 수 있을 것이다.

