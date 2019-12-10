---
title: Resilience4j CircuitBreaker
categories:
  - resilience4j
tags:
  - java
  - resilience4j
date: 2019-12-09 23:11:48
thumbnail:
---

# Resilience4j CircuitBreaker

resilience4j의 CircuitBreaker는 3개의 정상적인 state (CLOSED / OPEN / HALF_OPEN) 과 2개의 예외 state (DISABLED / FORCED_OPEN) 으로 구성된 finite state machine으로 구현되어있다.

## Sliding Window
호출의 결과를 Store / Aggregation하기 위해 Sliding Window 를 사용하고 있고 Count Based 와 Time Based, 2 가지 종류가 있다. default로는 Count Based를 사용하고 있다.

### Count Based Sliding Window
> [FixedSizeSlidingWindowMetrics.java](https://github.com/resilience4j/resilience4j/blob/master/resilience4j-core/src/main/java/io/github/resilience4j/core/metrics/FixedSizeSlidingWindowMetrics.java)

```java
// Count Based
CircuitBreakerConfig.custom()
                    .slidingWindowType(SlidingWindowType.COUNT_BASED)
                    .slidingWindowSize(N)
                    .build();
```

Count Based Sliding Window는 N개의 요청을 저장하고 aggregation해서 Circuit의 상태를 판단한다. N개의 Measurements를 가지고 있고 요청이 올때마다 가장 오래된 Measurement를 Total Aggregation에서 빼고 초기화한다.
요청이 있을 때마다 Total Aggregation을 업데이트하기 때문에 Circuit의 Snapshot을 얻는 것은 O(1)의 시간이 걸리고, Circuit을 위해 필요한 메모리는 N개의 Measurements를 가지기 때문에 O(N)이다.

### Time Based Sliding Window
> [SlidingTimeWindowMetrics.java](https://github.com/resilience4j/resilience4j/blob/master/resilience4j-core/src/main/java/io/github/resilience4j/core/metrics/SlidingTimeWindowMetrics.java)

```java
// Time Based
CircuitBreakerConfig.custom()
                    .slidingWindowType(SlidingWindowType.TIME_BASED)
                    .slidingWindowSize(N)
                    .build();
```

Time Based Sliding Window는 N개의 Partial Aggregation Circular Array로 구현되어있다. N초 동안의 요청을 저장하고 aggregation해서 Circuit의 상태를 판단한다. Partial Aggregation은 Bucket이라고 생각하면 되는데, 1초동안에 발생한 요청들에 대해 Aggregation해서 저장하고 있는 bucket이다. 
요청의 결과를 각각 저장하는 것이 아닌 Partial Aggregation과 Total Aggregation에 저장한다. 시간이 지나 필요없어진 Partial Aggregation은 Total Aggregation에서 빼고 초기상태로 reset 된다. 
Time Based Sliding Window도 Total Aggregation을 요청마다 업데이트하기 때문에 Circuit의 Snapshot을 얻는 것은 O(1)의 시간이 걸린다. 필요한 메모리는 N개의 Partial Aggregation과 1개의 Total Aggregation을 가지기 때문에 O(N) 이다.

## Threshold for Open State
resilience4j에서는 CircuitBreakerStateMachine가 OPEN State로 전이되기 위한 threshold를 2가지 제공한다. 요청이 Exception을 던지고 실패한 비율에 대한 Failure Rate Threshold, 지정한 Duration보다 오래 걸린 요청의 비율에 대한 Slow Call Rate Threshold가 있다. 
```java
// Minimum Number of Calls
CircuitBreakerConfig.custom()
                    .minimumNumberOfCalls(10)
                    .build();
```
Failure Rate와 Slow Call Rate 모두 minimum number of calls보다 많은 요청이 있을 때부터 계산하기 시작한다. minimum number of calls보다 적은 요청만 들어왔다면 CircuitBreakerStateMachine은 OPEN 상태로 전이되지 않는다. 
minimumNumberOfCalls를 지정할 때 주의할 점이 있다. 만약 COUNT_BASED Sliding Window를 사용하면서 Window Size를 minimumNumberOfCalls보다 작게 지정했다면 Window Size의 값이 minimumNumberOfCalls 값으로 사용된다.

### Failure Rate Threshold
```java
// Failure Rate Config
CircuitBreakerConfig.custom()
                    .failureRateThreshold(80)
                    .recordExceptions(Exception.class)
                    .ignoreExceptions(IllegalArgumentException.class)
                    .recordException(throwable -> throwable instanceof Exception)
                    .ignoreException(throwable -> throwable instanceof IllegalArgumentException)
                    .build();
```
말그대로 실패한 요청에 대한 threshold이다. 기본값은 50이고 Failure Rate가 전체 요청의 50%를 넘게 되면 CircuitBreakerStateMachine의 State가 CLOSED에서 OPEN으로 전이된다. 
기본으로는 모든 Exception이 Failure로 기록되지만 여러 가지 config 값을 넣어줄 수 있다. 
* recordExceptions : 실패로 기록할 Exception list를 넣어줄 수 있다. 이 list에 속한 Exception은 ignoreExceptions에 포함되지 않는다면 모두 Failure로 기록된다.
* ignoreExceptions : 실패나 성공으로 기록하지 않을 Exception list를 넣어줄 수 있다. 이 list에 속한 Exception은 Success도 아니고 Failure도 아닌 요청이 된다.
* recordException : 실패로 기록할 Exception을 판단하는 `Predicate<Throwable>`을 설정할 수 있다.
* ignoreException : 무시하고 넘어갈 Exception을 판단하는 `Predicate<Throwable>`을 설정할 수 있다.

### Slow Call Rate Threshold
```java
// Slow Call Rate Config
CircuitBreakerConfig.custom()
                    .slowCallDurationThreshold(Duration.ofSeconds(3))
                    .slowCallRateThreshold(80)
                    .build();
```
Slow Call에 대한 threshold 이다. slow call duration threshold를 설정할 수 있고 이 값보다 더 오래 걸린 요청에 대해서는 slow call로 기록하게 된다. 이렇게 기록된 slow call이 전체 요청에서 slow call rate threshold보다 높게 되면 CircuitBreakerStateMachine의 State가 OPEN으로 전이된다.
기본값으로는 slowCallDurationThreshold는 60초, slowCallRateThreshold는 100%로 되어있다. 그렇기 때문에 따로 설정을 넣어주지 않는다면 slow call에 의한 OPEN State 전이는 Disable된 것이나 마찬가지이다.

## CircuitBreaker State
CircuitBreaker의 State는 AtomicReference에 저장된다. 또 State를 update할 때, atomic operation을 사용하고 Sliding Window에 요청을 기록하거나 SnapShot을 읽는 작업들이 모두 synchronized되어있기 때문에 CircuitBreaker는 thread safe하다. 

### Half Open State
```java
// Permitted Calls in Half Open State
CircuitBreakerConfig.custom()
                    .permittedNumberOfCallsInHalfOpenState(80)
                    .build();
```
resilience4j의 CircuitBreaker는 HALF_OPEN state일 때, 다시 CLOSED나 OPEN으로 전이하기 위한 테스트 요청의 수를 정할 수 있다. CLOSED state의 Sliding Window는 TIME_BASED와 COUNT_BASED 2가지 종류를 지원하는 것과는 달리 HALF_OPEN state의 Sliding Window는 COUNT_BASED 만 지원하고 있다. 
permittedNumberOfCallsInHalfOpenState 값으로 HALF_OPEN state의 Sliding Window의 size를 지정할 수 있다. CLOSED state와 마찬가지로 minimumNumberOfCalls 값에 영향을 받으며 이때도 Window Size가 minimumNumberOfCalls보다 작으면 Window Size의 값을 minimumNumberOfCalls 값으로 사용한다.

### Disabled and Forced Open State
DISABLED State는 모든 요청을 허용하는 상태이고 FORCED_OPEN은 모든 요청을 거부하는 상태이다. 이 두 상태 모두 아무런 CircuitBreaker Event를 발생시키지 않고, metrics 또한 기록하지 않는다. 또 DISABLED나 FORCED_OPEN state에서 다른 state로 전이되는 방법은 직접 상태 전이를 trigger하거나 CircuitBreaker를 reset해야 한다.

## Other Configs
이외에도 writableStackTraceEnabled, automaticTransitionFromOpenToHalfOpenEnabled, waitIntervalFunctionInOpenState 등의 설정이 있다. 
* writableStackTraceEnabled : Circuit Open으로 인해 CallNotPermittedException이 발생했을때 Stack Trace를 출력할지 정하는 boolean 값이다.
* waitIntervalFunctionInOpenState : 고정된 wait duration이 아닌 Circuit open 후에 CLOSED State로의 전이를 위해 시도한 횟수로 wait duration을 설정해주는 Function을 넣어줄 수 있다.
* automaticTransitionFromOpenToHalfOpenEnabled : OPEN State에서 HALF_OPEN State로의 전이는 이 설정이 false일 경우, 다음 요청이 들어왔을 때 전이된다. 하지만 이 설정이 true라면 waitDurationInOpenState 시간 이후에 바로 HALF_OPEN State로 전이된다.


## Reference
* https://resilience4j.readme.io/docs/circuitbreaker
* https://github.com/resilience4j/resilience4j
