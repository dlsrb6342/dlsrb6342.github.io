---
title: ThreadLocal vs. InheritableThreadLocal
categories:
  - null
tags:
  - java
date: 2019-07-15 22:13:44
thumbnail:
---

# ThreadLocal vs. InheritableThreadLocal
Thread.java 코드를 보면 아래와 같이 InheritableThreadLocal과 ThreadLocal이 따로 관리되는 것을 볼 수 있다. 둘의 차이가 무엇인지 확인해보자.
```java
public class Thread implements Runnable {
    ...
     
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    ...
}
```

## ThreadLocal
ThreadLocal은 이름에서도 알 수 있듯이 한 Thread의 scope내의 local variable이다. 하나의 Thread 안에서는 같은 값을 공유해서 사용한다. 
```java
// 생성
ThreadLocal<String> threadLocal = new ThreadLocal<>();

// 저장
threadLocal.set("test");

// 사용
threadLocal.get();
```
사용법도 간단하다. 만들고 저장하고 쓰면 된다. 

### 활용
보통 하나의 Request에 대한 정보들을 저장할 때 사용된다. 
* spring-security : 사용자 인증 정보를 ThreadLocal에 담아서 전달해준다. 
* spring-web : RequestContextHolder가 요청의 Context를 ThreadLocal로 들고 있어 어디서든 꺼내쓸 수 있다.
* mdc : Logging에 사용될 attribute를 ThreadLocal로 들고 있다.

ThreadLocal은 spring-web을 사용할때는 문제가 되지 않았다. 하지만 reactive programming(spring-webflux)을 사용하게 되고 하나의 요청이 여러 Thread에서 처리될 수 있게 되면서 ThreadLocal이 문제가 되었다.

## InheritableThreadLocal
이름에서도 알 수 있듯이 **Inheritable**한 ThreadLocal이다. InheritableThreadLocal은 부모 Thread에서 생성된 자식 Thread에 그 값이 전달된다. 사용법은 ThreadLocal과 똑같다. 
자식 Thread에도 그 값이 전달되기 때문에 Thread를 왔다갔다 할수도 있는 reactive programming에서도 문제없이 사용 가능하다.

## 비교 테스트
```java
public class ThreadLocalTests {

	ThreadLocal<String> threadLocal = new ThreadLocal<>();
	InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

	@Test
	public void toAnotherThread() {
		long threadId = Thread.currentThread().getId();
		threadLocal.set("test");
		inheritableThreadLocal.set("test");

		System.out.println(String.format(
				"Thread ID : %s / ThreadLocal : %s / InheritableThreadLocal : %s",
				threadId, threadLocal.get(), inheritableThreadLocal.get()));

		Mono.fromRunnable(() -> {
			long anotherThreadId = Thread.currentThread().getId();
			System.out.println(String.format(
					"Thread ID : %s / ThreadLocal : %s / InheritableThreadLocal : %s",
					anotherThreadId, threadLocal.get(), inheritableThreadLocal.get()));
		}).subscribeOn(Schedulers.elastic()).subscribe();
	}
}
```
ThreadLocal과 InheritableThreadLocal을 비교하는 테스트이다. 부모 Thread에서 ThreadLocal과 InheritableThreadLocal를 모두 set해주고 자식 Thread에서 꺼내서 확인하는 코드이다. 다른 Thread를 실행하는 것은 reactor의 subscribeOn을 사용했다.
```console
Thread ID : 1 / ThreadLocal : test / InheritableThreadLocal : test
Thread ID : 15 / ThreadLocal : null / InheritableThreadLocal : test
```
결과는 이렇게 나온다. 부모 Thread에서는 두 값 모두 저장되어있지만 자식 Thread에서는 InheritableThreadLocal만 남아있다. 

## InheritableThreadLocal in zipkin
InheritableThreadLocal은 spring-cloud-sleuth와 zipkin을 사용하다가 발견했다. zipkin은 다른 Thread로 넘어가도 문제가 없게 하기 위해 현재 span의 정보를 CurrentTraceContext에 InheritableThreadLocal에 담아서 들고 있고 새로운 Scope가 시작될때 바꿔주고 있다. zipkin에서 MDC도 활용하고 있기 때문에 MDC에 넣어주는 부분은 Slf4jScopeDecorator를 참고하면 된다.

```java
// CurrentTraceContext.java
  public static final class Default extends ThreadLocalCurrentTraceContext {
    static final InheritableThreadLocal<TraceContext> INHERITABLE = new InheritableThreadLocal<>();

    public static CurrentTraceContext inheritable() {
      return new Default();
    }

    Default() {
      super(new Builder(), INHERITABLE);
    }
  }

// ThreadLocalCurrentTraceContext.java
  @Override public Scope newScope(@Nullable TraceContext currentSpan) {
    final TraceContext previous = local.get();
    local.set(currentSpan);
    class ThreadLocalScope implements Scope {
      @Override public void close() {
        local.set(previous);
      }
    }
    Scope result = new ThreadLocalScope();
    return decorateScope(currentSpan, result);
  }

```

<br/>
## 결론
Reactive 환경에서는 InheritableThreadLocal을 사용하면 Servlet에서 RequestContext를 관리했던 것처럼 할 수 있을 것 같다. 하지만 reactor에서는 `subscriberContext()`를 제공하기 때문에 이것을 활용하는 것이 더 나을수도 있을 것 같다. 나중에 그 둘을 비교하는 글도 작성할 예정이다. 
또 여러 글들을 보다보니 InheritableThreadLocal은 Servlet에서는 사용하면 좋지 않다는 얘기가 있다. 관련된 내용은 https://stackoverflow.com/questions/14498503/why-should-i-avoid-using-inheritablethreadlocal-in-servlets 이 링크에서 확인하면 된다.

