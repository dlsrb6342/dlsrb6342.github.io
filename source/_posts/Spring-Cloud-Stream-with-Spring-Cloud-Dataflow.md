---
title: Spring Cloud Stream with Spring Cloud Dataflow
categories:
  - spring-cloud-stream
  - spring-cloud-dataflow
tags:
  - spring
  - spring-cloud
  - java
date: 2021-02-07 16:25:32
thumbnail:
---

최근 회사에서 spring cloud dataflow 상에서 spring cloud stream을 사용하기로 결정해 여러가지 삽질을 한 것을 정리해보려 한다. 굉장히 다양한 삽질을 했으나 몇가지만 정리해보려고 한다.


## Functional Style
spring cloud stream app은 Functional Style, Annotation Based Style 2가지 방식으로 정의할 수 있다.  
**Annotation Based Style로 작성하였을때는 정말 아무 문제없이 처리되었지만.. Functional Style로 바꾸니 바로 동작하지 않았다.**

* https://cloud.spring.io/spring-cloud-stream/reference/html/spring-cloud-stream.html#spring_cloud_function
* https://cloud.spring.io/spring-cloud-stream/reference/html/spring-cloud-stream.html#_annotation_based_support_legacy 

```java
// Annotation Based Style
@SpringBootApplication
@EnableBinding(Processor.class)
public class SampleStreamApplication {
    @StreamListener(Sink.INPUT)
    @SendTo(Source.OUTPUT)
    public String toUpperCase(String str) {
        return str.toUpperCase();
    }
}

// Functional Style
@SpringBootApplication
public class SampleStreamApplication {
    @Bean
    public Function<String, String> toUpperCase() {
        return str -> str.toUpperCase();
    }
}
```

### 원인
원인부터 말하면 Annotation Based Style로 stream app을 작성했을때는 binding의 기본 이름이 `input`, `output`으로 정해지지만 Functional Style은 정의한 Function의 이름에 따라 달라진다.  

이 부분에 대해 부가 설명을 하자면..
* Annotation Based Style에 쓰이는 `@StreamListener`와 `@SendTo` annotation에 들어가는 값에 따라 binding의 이름이 정해지는데 보통 `Sink.INPUT`, `Source.OUTPUT`을 사용하기 때문에 기본 이름이라 칭했다.
* Functional Style 는 동작하지 않았다라고 한 것은 사실 spring-cloud-dataflow에 올렸을 경우에 동작하지 않는다는 것을 의미한다. spring-cloud-dataflow의 경우, stream app의 input, output binding을 `input`, `output`이라는 값으로 정해져있다고 가정하고 app을 구동하기 때문에 이런 경우가 발생하는 것으로 파악했다.


### 해결책
해결책은 간단했다. `application.yml`에 binding의 이름을 정해주면 된다. 위 예시 코드에서는 Function의 이름이 `toUpperCase`이므로 input binding은 `toUpperCase-in-0`, output binding은 `toUpperCase-out-0`으로 생성되었을 것이다. 이걸 `input`, `output`으로 바꿔주기만 하면 된다. 나중에 본 것이지만 [spring-cloud-dataflow-samples](https://github.com/spring-cloud/spring-cloud-dataflow-samples)에 예시가 잘 되어있었다.. ㅠㅠ
```yaml
// application.yml
spring.cloud.stream.function.bindings:
  toUpperCase-in-0: input
  toUpperCase-out-0: output
```

## Header Encoding
Stream Sample App 중에 header enricher라는 App이 있다. 입력한 SpEL에 따라 kafka record header에 값을 추가해주는 App인데 이 App을 통해 필요한 값을 헤더에 넣어주곤 했는데 stream definition을 따라 잘 전달되다가 **어느 앱만 지나면 String으로 잘 전달되던 헤더 값이 `byte[]`로 표현됐다.**

### 원인
Dependencies Version의 문제였다. 내가 사용 중인 Kubernetes Cluster는 1.15.X 버전이기에 spring cloud dataflow는 2.5.3을 설치해서 사용했다. 또 spring cloud dataflow 2.5.3을 사용했기 때문에 spring cloud는 Hoxton.SR4를 사용했다.

* https://dataflow.spring.io/docs/installation/kubernetes/compatibility/
* https://github.com/spring-cloud/spring-cloud-dataflow/releases/tag/v2.5.3.RELEASE

이렇게 해서 버전을 다 맞춰 문제가 없을 줄 알았으나.. spring cloud dataflow 2.5.3을 설치해서 import 해둔 Sample App들의 버전이 문제였다.  
내가 만든 Stream App들은 spring cloud Hoxton.SR4에 포함된 spring cloud stream 3.0.4.RELEASE를 사용했지만 미리 import된 Sample App들은 2.1.X 버전을 사용하고 있었다.

여기서 버전 차이로 인해 `BinderHeaderMapper`의 구현에 차이가 있었다. 그래서 하위 버전앱에서 추가된 헤더를 상위버전앱에서 제대로 읽지 못하는 이슈가 있는 것 같다.


### 해결책
해결책으로는 3.0.X Version의 Sample App을 빌드하는 것과 하위 버전 앱을 원하는 버전으로 직접 빌드하는 것, 두가지가 있다. 

Stream Sample App들은 2.1.X와 3.0.X가 다른 레포지토리에 관리되고 있다. 이걸 찾는 것도 힘들었다 ㅠㅠ
* 3.0.X : https://github.com/spring-cloud/stream-applications
* 2.1.X : https://github.com/spring-cloud-stream-app-starters

첫 번째 방법인 3.0.X Version을 사용하려면 [Release Tag](https://github.com/spring-cloud/stream-applications/releases)에서 spring-cloud-datflow에 Bulk Import할 수 있는 url / file을 얻을 수 있다. 간단히 Import해서 사용하기만 하면 된다.

두 번째 방법인 직접 하위 버전 앱을 빌드하는 방법은 아래와 같이 dependencies를 추가하고 Configuration만 import해주면 된다.
```
// 예시로 aggregator app을 빌드해보겠다.
// build.gradle
dependencies {
    implementation 'org.springframework.cloud.stream.app:spring-cloud-starter-stream-processor-aggregator:2.1.5.RELEASE'
}

// AggregatorApplication.java
@SpringBootApplication
@Import(AggregatorProcessorConfiguration.class)
public class AggregatorApplication {
    public static void main(String[] args) {
        SpringApplication.run(AggregatorApplication.class, args);
    }
}
```


## Headers with Reactor
> https://docs.spring.io/spring-cloud-stream/docs/3.1.1/reference/html/spring-cloud-stream.html#_reactive_functions_support

Stream Application은 Reactor를 사용해서 Reactive Programming model로 작성할 수 있다.  
이 예시로 위에 말한 sample app 중에 httpclient(http-request)를 살펴보면 2.1.X의 httpclient는 내부에서 RestTemplate bean을 생성해서 요청을 보내고 있고 http-request-processor는 WebClient를 사용하고 있다.  
* 2.1.X : https://github.com/spring-cloud-stream-app-starters/httpclient
* 3.0.X : https://github.com/spring-cloud/stream-applications/tree/master/applications/processor/http-request-processor

**그런데 reactor를 사용해 Function을 작성했을때 Header가 다음 App으로 전달되지 않았다.**

### 원인
아쉽게도 이 문제는 아직 원인을 파악하지 못했다. 내부의 Message build하는 로직을 많이 들여다봤는데도 Reactor로 되어있다보니 Debugging이 쉽지가 않았다.  
아마 Message build 과정에서 헤더는 복사되지 않고 build하고 있는 것 같다. 원인을 아시는 분은 제보 부탁드립니다... ㅋㅋ

### 해결책
원인을 못 찾았기 때문에 해결책이 깔끔하지는 않다. Message Build가 문제되는 것 같으니 직접 Message를 build했다. 이렇게 하면 다음 Stream으로 문제없이 헤더가 잘 전달된다.
```java
@SpringBootApplication
public class SampleStreamApplication {
    @Bean
    public Function<Flux<Message<String>>, Flux<Message<String>>> reactiveToUpperCase() {
        return messageFlux -> messageFlux
                .map(message -> {
                    String payload = message.getPayload();
                    return MessageBuilder.createMessage(payload.toUpperCase(), message.getHeaders());
                });
    }
}
```


## Stream App Test
Stream App은 Kafka 혹은 RabbitMQ를 필요로 한다. **물론 테스트를 위해 TestContainer 혹은 Embedded Kafka 같은 방법을 선택할 수 있지만 여러 제한이 있어 사용하지 못했다. 그래서.. 테스트할 방법을 찾아야만 했다.** 이번 삽질은 딱히 원인/해결책은 없고 Stream App을 테스트하는 방법을 소개하려고 한다. 

Stream Sample Apps에서 사용하고 있는 테스트 방법을 사용했다. Kafka / RabbitMQ가 아닌 spring-cloud-stream의 test 구현체인 TestBinder를 사용하는 방법이다.
* https://github.com/spring-cloud/stream-applications/blob/1e6a2e1f44f12619d4522e97a9f5e5869461cd8f/applications/stream-applications-core/pom.xml#L83-L89

maven에서는 위 링크와 같이 넣어주면 되지만 gradle은 아래와 같이 하면 된다. gradle을 많이 사용해보지 않는 나로서는 이것도 꽤나 찾기 힘들었다 ㅠㅠ
```groovy
dependencies {
    testImplementation group: 'org.springframework.cloud', name: 'spring-cloud-stream', classifier: 'test-binder'
}
```

test-binder를 사용한 `toUpperCase` App에 대한 간단한 테스트 코드는 아래와 같다.
```java
class SampleStreamApplicationTest {
    @Test
    void test_toUpperCase() {
        try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
                TestChannelBinderConfiguration.getCompleteConfiguration(SampleStreamApplication.class))
                .web(WebApplicationType.NONE)
                .run()) {

            InputDestination processorInput = context.getBean(InputDestination.class);
            OutputDestination processorOutput = context.getBean(OutputDestination.class);

            String testStr = "test";
            final Message<Map<String, String>> message = MessageBuilder.withPayload(testStr)
                                                     .build();

            processorInput.send(message);
            Message<byte[]> output = processorOutput.receive();

            assertEquals(testStr.toUpperCase, new String(output.getPayload()));
        }
    }
}
```
<br>

---

<br>
spring cloud dataflow가 Task / Stream / Batch 를 관리하고 실행할 수 있다는 점에서 굉장히 좋아보여 사용하기로 했는데 생각보다 Reference가 부족해 조금 어려움을 겪었다. 전체적인 개념을 이해하는데에 시간이 오래 걸리긴 했지만 공식문서가 굉장히 잘 되어있어 도움이 됐다. 영어로 되어있더라도 건너뛰지 않아야겠다.  
앞으로 spring cloud dataflow / spring cloud stream을 더 많이 사용할테니 삽질은 더 많아질 예정이다..



