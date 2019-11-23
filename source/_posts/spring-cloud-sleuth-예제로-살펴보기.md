---
title: spring-cloud-sleuth 예제로 살펴보기
categories:
  - spring-cloud
  - spring-cloud-sleuth
tags:
  - spring
  - spring-cloud
  - java
date: 2019-07-20 17:36:43
thumbnail:
---
> Spring Cloud Sleuth implements a distributed tracing solution for Spring Cloud.

오늘은 spring-cloud 프로젝트 중 분산형 추적 솔루션인 spring-cloud-sleuth에 대해 알아보려한다.
공식 문서 : https://cloud.spring.io/spring-cloud-sleuth/spring-cloud-sleuth.html

## 용어
spring-cloud-sleuth는 [Dapper](https://ai.google/research/pubs/pub36356)의 용어를 가져와서 사용하고 있다. 중요한 용어 몇가지만 정리하겠다.

### Span
Span은 64 bit unique ID를 가지는 Tracing의 가장 기본 단위이다. 
```java
final class RealSpan extends Span {

  final TraceContext context;
  final PendingSpans pendingSpans;
  final MutableSpan state;
  final Clock clock;
  final FinishedSpanHandler finishedSpanHandler;

  ...
}
```
[openzipkin/brave](https://github.com/openzipkin/brave)에 있는 Span 구현체이다. context는 span이 속해있는 Trace에 대한 정보를 들고 있고 pendingSpans에는 report되기 전의 pending된 span들이 WeakConcurrentMap으로 담겨있다. 
실제로 이 Span에 대한 정보를 담고 있는건 state이다. TraceContext를 제외한 시작시간 / local ip / remote ip / tags 등등 많은 정보를 들고 있다.

### Trace
Span을 설명할때 잠깐 언급된 Trace이다. Trace는 span들을 모아둔 것으로 tree와 비슷한 구조로 되어있다. Trace Id는 가장 처음 trace를 시작시킨 span의 ID와 같다.

### Annotation
Annotation은 어떤 이벤트가 발생하고 진행되는지에 대한 기록을 쓰인다. 하지만 [Brave](https://github.com/openzipkin/brave)를 사용하면서 더이상 request에 대해 이벤트들을 기록할 필요가 없어졌다. 
* cs : Client Sent. client가 request를 보낸 이벤트. span의 시작이다.
* sr : Server Received. server가 request를 받는 이벤트. 
* ss : Server Sent. server가 request 처리를 완료한 이벤트.
* cr " Client Received. client가 response를 받는 이벤트. span의 종료이다.

## Brave
spring-cloud-sleuth 2.0.0 version부터는 tracing instrumentation library인 [Brave](https://github.com/openzipkin/brave)를 사용하고 있다.  sleuth는 더이상 context를 만들거나 관리하지 않고 그 일을 Brave에게 위임하였다. naming이나 tagging conventions도 Brave와 똑같게 변경하였다.


## Example
이제 spring-cloud-sleuth를 실제로 사용해보면서 공부해보겠다. 예시 코드는 https://github.com/dlsrb6342/blog-sample/tree/master/spring-cloud-sleuth-example 에 올라가있다. 예시는 spring-webflux를 사용해서 만들려고 했지만 제대로 동작하질 않아서 힘들었다. 이틀정도 고생하면서 내가 뭘 잘못해서 안되는지 찾아봤다. 결론은 내 잘못은 아닌것으로 보인다. [spring-cloud-sleuth 이슈](https://github.com/spring-cloud/spring-cloud-sleuth/issues/1402)로 등록해두었다. 아직 답변은 못 받았지만 버그인 것 같다. 예시 코드는 일단 webflux와 mvc 두개 모두 만들어두었다.

예시 코드의 구조는 간단하다. 4가지 application(fcaller, fclient, mcaller, mclient)을 만들고 모두 sleuth를 적용시켜두었다. zipkin 서버는 간단하게 docker에 띄우고 실행했다. http://localhost:9411에 접근해보면 zipkin ui를 볼 수 있을 것이다.
```console
> docker run -d -p 9411:9411 openzipkin/zipkin
```

### user -> mcaller -> mclient
mcaller와 mclient는 spring-mvc를 사용해 구현한 application이다. 
```console
> curl http://localhost:8082/mcaller
{"mclient":"client","mcaller":"caller"}
```
위 요청을 보내면 mcaller가 mclient에 요청을 보내고 받은 응답을 user에게 돌려준다. zipkin에도 해당 request가 정확히 tracing되었다.

{% asset_img "mvc.png" "sleuth with mvc" %}

1. mcaller가 user로부터 받은 요청으로 인해 생긴 "Server Start / Server finish" span
2. mcaller가 mclient를 호출하면서 생긴 "Client Start / Client finish" span
3. mclient가 mcaller로부터 받은 요청으로 인해 생긴 "Server Start / Server finish" span

총 3개의 span이 생성되었고 문제없이 잘 동작하였다.

### user -> fcaller -> fclient
fcaller와 fclient는 spring-weblfux로 만들었는데 위에 언급했듯이 정확하게 동작하지 않는다. Tracing정보를 넘겨주고 요청이 완료될 경우 report를 하고 있는 CoreSubscriber의 onComplete 함수가 제대로 호출되지 않고 있다.
자세한 예시와 내용에 대해서는 https://github.com/spring-cloud/spring-cloud-sleuth/issues/1402 여기서 확인하면 된다.

## 정리
spring boot의 auto configuration 덕분에 spring-cloud-sleuth를 dependency에 넣어주고 zipkin uri만 넣어주는 것만으로도 쉽게 적용할 수 있었다. 하지만 spring-webflux에서는 제대로 동작하지 않을뿐만 아니라 성능상의 이슈도 있다고 한다( https://github.com/spring-cloud/spring-cloud-sleuth/issues/1397 ). 성능이 중요한 webflux application을 개발할 때는 spring-cloud-sleuth를 적용하는 것에 대해 조금 고민이 필요할 것 같다. 
