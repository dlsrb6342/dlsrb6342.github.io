---
title: Resilience4j RateLimiter
categories:
  - resilience4j
tags:
  - java
  - resilience4j
date: 2019-12-12 00:43:22
thumbnail:
---

# Resilience4j RateLimiter

RateLimiter 패턴이란 일정 시간동안의 요청의 수를 제한하는 것이다. 자세한 내용은 생략한다. [Token Bucket wiki](https://en.wikipedia.org/wiki/Token_bucket)와 [Java token-bucket 구현체](https://github.com/bbeck/token-bucket)를 보면 도움이 될 것이다.

## AtomicRateLimiter
resilience4j의 RateLimiter는 AtomicRateLimiter와 SemaphoreRateLimiter 2가지로 구현되어있는데 default로 AtomicRateLimiter를 사용한다. AtomicRateLimiter는 RateLimiter가 생성된 시점부터의 모든 nanosecond를 cycle로 나눈다. 

### Cycle
Cycle은 RateLimiterConfig.limitRefreshPeriod 값만큼의 Duration을 가진다. RateLimiter는 각 Cycle이 시작할때마다 active permission의 수를 RateLimiterConfig.limitForPeriod 값으로 설정해준다. 
호출하는 입장에서 바라봤을 때는 매 Cycle마다 active permission을 바꿔주는 것으로 보이겠지만 resilience4j의 AtomicRateLimiter는 RateLimiter를 사용하지 않는동안에는 refresh를 skip하는 성능 최적화가 되어있다.

### State
AtomicRateLimiter는 내부에서 State 값을 AtomicReference로 관리하고 있다. State는 immutable한 객체이며 아래와 같은 값들을 갖는다.
* activeCycle : 가장 최근 호출에서 사용된 cycle number.
* activePermissions : 현재 사용할 수 있는 permission의 수. 만약 reserve된 호출이 있다면 음수 값을 가질 수 있다.
* nanosToWait : 최근 호출이 permission을 얻기 위해 기다려야 할 nanoseconds.

## RateLimiterConfig
```java
RateLimiterConfig.custom()
                 .timeoutDuration(Duration.ofSeconds(1))
                 .limitRefreshPeriod(Duration.ofSeconds(2))
                 .limitForPeriod(100)
                 .writableStackTraceEnabled(true)
                 .build();
```

RateLimiterConfig에는 timeoutDuration, limitRefreshPeriod, limitForPeriod, writableStackTraceEnabled이 있다.
* timeoutDuration : permission을 얻기 위해 thread가 기다릴 시간.
* limitRefreshPeriod : limit가 refresh되는 period. 
* limitForPeriod : period마다 허용되는 permission의 수.
* writableStackTraceEnabled : 호출이 permission을 얻지 못해 CallNotPermittedException이 발생했을때 Stack Trace를 출력할지 정하는 boolean 값.

위의 예시에서는 2초동안 100개의 요청만 허용하면서 만약 permission이 부족할 경우, 1초 간은 기다리게 되는 설정이다. 또 CallNotPermittedException이 발생하면 Stack Trace까지 출력하게 된다.

## Weight in RateLimiter
```java
RateLimiter rateLimiter = RateLimiter.ofDefaults("test");
rateLimiter.executeRunnable(3, () -> {
    ...
});
```

현재 공식 문서에는 추가되어있지는 않지만 1.2.0 버전부터는 RateLimiter의 permission을 얻을 때, weight를 넣어서 호출할 수 있다. 위 예시에서 실행하고자 하는 Runnable Lambda 표현식은 permission 3개를 필요로 하게 된다. 

## Reference
* https://resilience4j.readme.io/docs/ratelimiter
* https://github.com/resilience4j/resilience4j
* https://en.wikipedia.org/wiki/Token_bucket
