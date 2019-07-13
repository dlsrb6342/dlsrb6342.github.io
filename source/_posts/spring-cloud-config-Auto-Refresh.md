---
title: spring-cloud-config Auto Refresh
categories:
  - spring-cloud
  - spring-cloud-config
tags:
  - spring
  - spring-cloud
  - java
date: 2019-06-28 22:11:26
thumbnail:
---

spring-cloud-gateway를 사용하면서 Dynamic Properties를 사용하고 싶어 spring-cloud-config도 같이 사용하기로 했다. 
> https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html

## Refresh 이슈
설정된 PropertySource를 refresh 해주려면 Actuator API를 하나하나 다 요청해주어야 한다. 여러 머신에 여러 인스턴스가 떠있는 상황이었기에 하나하나 다 찔러주는건 무리가 있었다.
### 1. spring-cloud-bus
이런 상황을 위해 [spring-cloud-bus](https://cloud.spring.io/spring-cloud-bus/spring-cloud-bus.html)가 있다. mq를 써서 각 인스턴스마다 refresh 이벤트를 전달해준다.
> https://www.baeldung.com/spring-cloud-bus

***하지만 spring-cloud-bus까지 도입하자니 운영해야할 것들이 너무 많이 늘어나게 된다.***

### 2. ConfigClientWatch
이전 프로젝트에서 Dynamic Properties를 위해 [CentralDogma](https://line.github.io/centraldogma/)를 사용한 적이 있다. CentralDogma에는 Watcher 구현체가 있어서 spring-cloud-config-client에도 있지 않을까?하는 생각에 코드를 뒤져봤다. 
[ConfigClientWatch](https://github.com/spring-cloud/spring-cloud-config/blob/master/spring-cloud-config-client/src/main/java/org/springframework/cloud/config/client/ConfigClientWatch.java)가 있었다! `spring.cloud.config.watch.enabled` 값을 true로 설정해주면 된다.

***하지만...***
```java
// https://github.com/spring-cloud/spring-cloud-config/blob/f24f02395153761236b102786a15450a48fe0b10/spring-cloud-config-client/src/main/java/org/springframework/cloud/config/client/ConfigClientWatch.java#L65-L72
String newState = this.environment.getProperty("config.client.state");
String oldState = ConfigClientStateHolder.getState();

// only refresh if state has changed
if (stateChanged(oldState, newState)) {
	ConfigClientStateHolder.setState(newState);
	this.refresher.refresh();
}
```
내부 로직을 보면 ConfigClientWatch는 `config.client.state` 값을 보고 변경되었는지 체크한다. 하지만 이 state 값은 [Vault](https://www.vaultproject.io/)를 Backend로 사용했을 때만 존재한다.

***따라서 git을 Backend로 사용하고 있는 난 사용할 수 없었다.***

### 3. Version?

Git을 Backend로 썼을 때는 방법이 없을까 하고 열심히 코드를 뒤져봤다.
그러다 `config.client.state` 값은 config client가 config server로부터 받은 결과에서 꺼내서 직접 property에 put하고 있다는것을 발견했다.
[`Environment`](https://github.com/spring-cloud/spring-cloud-config/blob/master/spring-cloud-config-client/src/main/java/org/springframework/cloud/config/environment/Environment.java) 클래스가 config server로부터 받은 결과이고 이 안에 state와 version이 있다. 
그 결과를 받은 config client는 아래 코드와 같이 직접 property를 넣어주고 있다.
```java
// https://github.com/spring-cloud/spring-cloud-config/blob/79cd1100d1399cd880ea5c89577de1ed8f80396b/spring-cloud-config-client/src/main/java/org/springframework/cloud/config/client/ConfigServicePropertySourceLocator.java#L118-L119
putValue(map, "config.client.state", result.getState());
putValue(map, "config.client.version", result.getVersion());
```

state뿐만 아니라 version을 넣어주고 있었고 테스트를 해보니 Git을 Backend로 사용했을때 config server가 이 값을 git HEAD의 checksum으로 전달해주고 있었다. 

따라서 `config.client.state` 값이 아닌 `config.client.version`을 확인한다면 똑같이 refresh의 효과를 볼 수 있을거라 생각이 들었다.

### 4. ConfigGitClientWatch
```java
public class ConfigGitClientWatch implements Closeable, EnvironmentAware {

    private final AtomicBoolean running = new AtomicBoolean(false);
    private final AtomicReference<String> version = new AtomicReference<>();
    private final ContextRefresher refresher;
    private final ConfigServicePropertySourceLocator locator;

    private Environment environment;

    public ConfigGitClientWatch(
            ContextRefresher refresher, ConfigServicePropertySourceLocator locator) {
        this.refresher = refresher;
        this.locator = locator;
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

    @PostConstruct
    public void start() {
        running.compareAndSet(false, true);
    }

    @Scheduled(
            initialDelayString = "${spring.cloud.config.watch.git.initialDelay:180000}",
            fixedDelayString = "${spring.cloud.config.watch.git.delay:500}"
    )
    public void watchConfigServer() {
        if (running.get()) {
            String newVersion = fetchNewVersion();
            String oldVersion = version.get();

            if (versionChanged(oldVersion, newVersion)) {
                version.set(newVersion);
                refresher.refresh();
            }
        }
    }

    private String fetchNewVersion() {
        CompositePropertySource propertySource = (CompositePropertySource) locator.locate(environment);
        return (String) propertySource.getProperty("config.client.version");
    }

    private static boolean versionChanged(String oldVersion, String newVersion) {
        return !hasText(oldVersion) && hasText(newVersion)
               || hasText(oldVersion) && !oldVersion.equals(newVersion);
    }

    @Override
    public void close() {
        running.compareAndSet(true, false);
    }
}

```
직접 구현한 Git Backend spring-cloud-config-client Watch이다. commit을 할때마다 refresh가 잘 되는걸 확인했다. `Scheduled` annotation을 사용했기 때문에 Configuration이나 Application 클래스에 `EnableScheduling` annotation을 달아주어야 한다. 

### 5. Issue 등록 / PR 까지
Default Backend를 git으로 사용하고 있는데 Git Watch가 없는게 이상하기도 하고 ConfigClientWatch에 대해 Vault만을 위한 것이라고도 명시하지 않아 Issue를 등록했다.
> https://github.com/spring-cloud/spring-cloud-config/issues/1378

이슈 등록 후에 별다른 피드백이 없길래 프로젝트를 위해 만들었던 ConfigGitClientWatch를 수정하여 PR까지 올렸다.
> https://github.com/spring-cloud/spring-cloud-config/pull/1390

하지만 이게 필요한지 아직 확실하지 않다는 코멘트와 함께 PR은 closed되었고 이슈에는 `waiting for votes` 라벨이 달렸다. 
혹시 Git Watch가 필요하다 생각되신 분들은 이슈에 ThumbsUp 버튼 한번씩 클릭해주시면 감사하겠습니다 ㅎㅎ

## Source Code
https://github.com/dlsrb6342/blog-sample/tree/master/spring-cloud-config-auto-refresh
위 레포에 올려두었다. gradle 멀티모듈로 server/client를 만들고 config server는 같은 레포를 바라보고 있지만 config client가 label을 config로 지정하여 config branch를 볼 수 있게 하였다.
