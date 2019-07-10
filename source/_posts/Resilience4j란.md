---
title: Resilience4j란
categories:
  - resilience4j
tags:
  - java
  - resilience4j
date: 2019-06-03 00:44:41
thumbnail:
---

# Resilience4j

> Resilience4j is a lightweight, easy-to-use fault tolerance library inspired by Netflix Hystrix. 
 
Hystrix는 다른 dependency를 많이 가진 archaius를 사용했지만 Resilience4j는 Vavr를 제외한 다른 library dependancy가 없다.

## Netflix Hystrix와 다른점
* Hystrix는 외부 API 요청을 HystrixCommand로 wrapping해서 사용해야 한다. 하지만 Resilience4j는 functional interface, lambda, method reference를 위한 higher-order functions(decorator)를 제공한다.   
> 어떤 decorator를 사용할지 혹은 사용하지 않을지 선택할 수 있게 된다.

* Hystrix는 기본적으로 10개의 1-second window buckets에 실행 결과를 저장하고, bucket 하나가 끝나면 새로운 bucket을 만들고 기존 bucket은 없앤다.  Resilience4j는 Ring Bit Buffer에 결과를 저장하고 성공한 요청은 0 bit로 실패한 요청은 1 bit로 저장한다. Ring Bit Buffer는 설정으로 조절 가능한 fixed-size를 갖고, long[] array에 bits를 저장한다.  
> time winodw가 지났다고 실행 결과를 제거하지 않기 때문에 frequency가 낮든 높든 별다른 설정없이 CircuitBreaker를 이용할 수 있다.

* Hystrix는 CircuitBreaker가 half-open 상태일때 Circuit을 close할지 결정할 딱 한번의 요청을 수행한다. 하지만 Resilience4j는 요청의 수행 횟수와 threshold 값을 설정할 수 있다.
* RxJava operators를 지원한다.

## resilience4j-circuitbreaker
https://martinfowler.com/bliki/CircuitBreaker.html
### CircuitBreakerRegistry
* CircuitBreaker instance를 만들고 없애는 관리를 할 수 있는 class이다.
* 기본적으로 ConcurrentHashMap을 활용한 in-memory CircuitBreakerRegistry를 제공한다.

### CircuitBreakerConfig
* `failureRateThreshold`
  * CirucitBreaker를 열지 결정하는 failure rate threshold percentage 값. 
  * default는 50
* `waitDurationInOpenState` 
  * CircuitBreaker를 open한 상태를 유지하는 지속 기간을 의미한다. 이 기간 이후에 half-open 상태가 된다. 
  * default는 60seconds
* `ringBufferSizeInHalfOpenState` 
  * CircuitBreaker가 half-open 상태일때 RingBuffer의 사이즈. 
  * default는 10
* `ringBufferSizeInClosedState`
  * CircuitBreaker가 closed 상태일때 RingBuffer의 사이즈.
  * default는 100
* `recordExceptions`
  * failure count를 증가시켜야하는 exception list
* `ignoreExceptions`
  * failure count를 증가시키지 않고 무시하는 exception list
* `recordFailure`
  * 어떠한 경우에 Failure Count를 증가시킬지 Predicate를 정의해 CircuitBreaker에 대한 Exception Handler를 재정의하는 것이다. true를 return할 경우, failure count를 증가시키게 된다.
* `automaticTransitionFromOpenToHalfOpenEnabled`
  * waitDurationInOpenState 기간 이후에 ScheduledExecutorSerivce를 이용해 half-open으로 자동으로 전환해줄지 결정하는 값. 
  * default는 false

## resilience4j-ratelimiter
https://stripe.com/blog/rate-limiters  
CircuitBreaker와 비슷한 API를 가지고 있다. In-memory RateLimiterRegistry를 제공하고 RateLimiterConfig로 설정할 수 있다.
### RateLimiterConfig
* `timeoutDuration` 
  * 요청을 허용해주기 전까지 기다리는 기간을 의미한다. 
  * default는 5초
* `limitForPeriod`
  * 하나의 period(limitRefreshPeriod 값을 사용)마다 허용하는 limit count. 
  * default는 50
* `limitRefreshPeriod` 
  * limit count(limitForPeriod 값을 사용)가 refresh되는 period duration. 1ns 이상이어야 한다.
  * default는 500ns

## resilience4j-bulkhead
https://skife.org/architecture/fault-tolerance/2009/12/31/bulkheads.html
https://github.com/Netflix/Hystrix/wiki/How-it-Works#Isolation

Hystrix와 다른 점은 Hystrix는 각각의 dependecy 마다 Thread pool을 주고 Thread Pool size만큼 execution을 제한하고 있다면 resilience4j-bulkhead는 maxConcurrentCalls을 semaphore의 permits 값으로 주고 semaphore를 이용해 제한한다.  
다른 구현 모듈과 비슷하게 In-memory BulkheadRegistry를 제공한다.
### BulkheadConfig
* `maxConcurrentCalls` 
  * Bulkhead가 허용할 최대 concurrent call의 값.
  * default는 25
* `maxWaitTime` 
  * Thread가 semaphore를 획득할 때까지 기다리는 timeout 값이다. 값이 0이면 semaphore를 획득할 수 없는 상황에는 기다리지 않고 바로 요청을 막게 된다. 
  * default는 0

### Dynamic Reconfiguration
* bulkhead 모듈은 다른 모듈과 다르게 changeConfig() 함수를 제공한다.
* 하지만 이미 현재 semaphore를 얻기 위해 기다리고 있는 Thread에게는 바뀐 maxWaitTime이 적용되지 않는다.

## resilience4j-retry
Retry는 Registry는 따로 없고 `Retry retry = Retry.ofDefaults("id");` 이런 식으로 생성한다.
### RetryConfig
* `maxAttempts`
  * 최대 retry 시도 횟수. 
  * default는 3
* `waitDuration` 
  * 시도 실패 후 기다리는 시간을 의미하고 10ms 이상이어야 한다. 
  * default는 500ms.
* `retryOnResult` 
  * response에 따라 재시도를 할지 말지 결정하는 Predicate.
* `retryOnException` 
  * 어떤 exception이 발생했을때 재시도를 할지 결정하는 Predicate.
* `retryExceptions` 
  * 재시도를 할 Exception 클래스 리스트.
* `ignoreExceptions` 
  * 재시도를 하지 않을 Exception 클래스 리스트.  

retryOnException과 retryExceptions, ignoreExceptions 이 3개를 조합해서 exceptionPredicate를 만들어 retry할 상황을 결정한다.

## resilience4j-cache
resilience4j-cache는 javax.cache.Cache를 wrapping해서 사용한다. javax.cache.Cache instance를 만들고 그 instance를 io.github.resilience4j.cache.Cache로 wrapping한다.
```java
// Create a CacheContext by wrapping a JCache instance.
javax.cache.Cache<String, String> cacheInstance = Caching.getCache("cacheName", String.class, String.class);
Cache<String, String> cacheContext = Cache.of(cacheInstance);

// Decorate your call to BackendService.doSomething()
CheckedFunction1<String, String> cachedFunction = Decorators.ofCheckedSupplier(() -> backendService.doSomething())
    .withCache(cacheContext)
    .decorate();
String value = Try.of(() -> cachedFunction.apply("cacheKey")).get();
```

## resilience4j-timelimiter
TimeLimiter는 future supplier의 time limit을 정하는 API이다.
### TimeLimiterConfig
* `timeoutDuration` 
  * 실행 timeout 값이다. 
  * default는 1초
* `cancelRunningFuture` 
  * timeout 발생 후 future를 취소할지 결정하는 boolean 값. 
  * default는 true
