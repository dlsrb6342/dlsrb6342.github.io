---
title: Gradle Multi Project with Multiple version of plugin
categories:
  - gradle
tags:
  - gradle
date: 2021-01-30 15:58:19
thumbnail:
---

gradle에서 spring boot dependencies 버전은 `org.springframework.boot` plugin으로 관리된다. 그런데 plugin의 경우 rootProject에서 버전을 명시해버리면 하위 프로젝트에서 버전을 변경해서 적용할 수 없다. 이 점 때문에 gradle에서 한가지 plugin을 다양한 버전으로 사용하는 것은 조금 불편하다..
```
.                             # 전체 프로젝트는 2.4.1
├── build.gradle
├── bar
│   └── build.gradle
├── foo
│   └── build.gradle
└── streams                   # streams 이하 모듈은 2.2.7.RELEASE
    ├── build.gradle
    ├── stream-baz
    │   └── build.gradle
    └── stream-qux
        └── build.gradle
``` 

이번 새 프로젝트는 특정 모듈에서만 spring boot 버전이 다르게 사용해야 하는 요구사항이 있었다. 위 프로젝트 구조에서 `streams` 아래 모듈에서만 2.2.7.RELEASE 버전의 spring boot를 사용해야 했다(나중에 글을 쓸 예정이지만 spring-cloud-dataflow를 사용하면서 어쩔 수 없는 상황이었다 ㅠㅠ). 대충 요구사항을 정리해보자면..

1. spring-boot 버전은 `streams`에서는 2.2.7.RELEASE를, 나머지 모듈에서는 2.4.1을 사용한다.
2. Library/Plugin의 버전 관리는 하나의 파일에서 혹은 정리된 형태로 보기 쉽게 한다.  

이 요구사항을 충족시키기 위해 몇가지 방법을 찾고 적용해봤다.

## 1. Plugin Version in classpath
첫 번째 방법은 classpath로 plugin version을 선언해두는 것이다. 
```groovy
// libraries.gradle
ext.version = [
        springBoot: '2.4.1',
        ...
]

ext.libraries = [
        classpath: [
                springBootPlugin: "org.springframework.boot:spring-boot-gradle-plugin:${ext.version.springBoot}",
        ],
        dependency: [
                ....
        ]
]

ext.streamsVersion = [
        springBoot: '2.2.7.RELEASE',
        ...
]

ext.streamsLibraries = [
        classpath: [
                springBootPlugin: "org.springframework.boot:spring-boot-gradle-plugin:${ext.streamsVersion.springBoot}"
        ],
        dependency: [
                ...
        ]
]
```
```groovy
//rootProject build.gradle
subprojects {
    apply from: "${rootDir}/libraries.gradle"

    buildscript {
        dependencies {
            if (project.displayName.contains("streams")) {
                classpath "${project.ext.streamsLibraries.classpath.springBootPlugin}"
            } else {
                classpath "${project.ext.libraries.classpath.springBootPlugin}"
            }
        }
    }
}

// sub module builld.gradle
apply plugin: 'org.springframework.boot'
```

이렇게 선언해놓으면 sub module들에서 plugin을 적용하기만 해도 원하는 버전으로 적용시킬 수 있다. 
#### 장점 
* `libraries.gradle`에서 모든 버전 및 라이브러리 관리할 수 있다.
* sub module에서는 버전에 대해 신경쓰지 않고 모든 것을 root project가 관리할 수 있다.

#### 단점
* plugin 적용 방법이 새로운 형식(`plugins { id 'ID' }`)이 아닌 오래된 형식(`apply plugin 'ID'`)을 사용해야 한다. 

#### 결론
오래된 형식의 plugin 적용 방법을 사용하고 싶지 않아서 이 방법은 버려졌다. 물론 `subproject`, `allproject`에서는 `plugins`는 사용할 수 없고 `apply plugin`을 사용해야 한다. (https://stackoverflow.com/a/32353244/7018283)

## 2. pluginManagement 
> https://docs.gradle.org/current/userguide/plugins.html#sec:plugin_management

gradle에서 plugin 버전을 관리할 수 있는 방법을 공식적으로 지원한다. 그런데 이 방법은 무조건 `setting.gradle` 파일의 가장 처음에 작성되거나 [init scripts](https://docs.gradle.org/current/userguide/init_scripts.html#init_scripts) 파일에 작성되어야 한다. 

#### 결론
하지만 이 방법으로는 내가 원하는 모듈 별로 서로 다른 버전의 plugin을 적용할 수는 없는 것 같다(할 수 있지만 방법을 못 찾은것일수도..). 만약 전체 프로젝트에서 version을 한가지만 사용하고 있다면 공식적으로 지원하는 방법이니 이걸 택했을 것 같다.


## 3. `gradle.properties`에 plugin 버전 선언

이 방법은 plugin 버전을 `libraries.gradle`이 아닌 `gradle.properties`에 선언하는 방법이다. `plugins` 블록에서 버전을 변수로써 작성하려면 `libraries.gradle`에 선언한것은 동작하지 않고 `gradle.properties`에 선언되어 있어야 동작하기 때문이다. 

```properties
// gradle.properties
springBootVersion=2.4.1
springBootVersionForStreams=2.2.7.RELEASE
```
```groovy
// foo/build.gradle
plugins {
    id 'org.springframework.boot' version "${springBootVersion}"
}

// streams/build.gradle
plugins {
    id 'org.springframework.boot' version "${springBootVersionForStreams}"
}
```

#### 장점
* `plugins` 블록을 사용하여 plugin을 적용할 수 있다. 
* 버전 관리를 `gradle.properties`에서 할 수 있다. 

#### 단점
* 각 모듈에서 plugin을 적용할 때 항상 version을 명시해주어야 한다. 
* dependencies 관리는 `libraries.gradle`, plugin 관리는 `gradle.properties`로 나뉜다. 사실 이 점은 `gradle.properties`에 모두 맡겨도 된다.

#### 결론
`plugins` 블록을 사용할 수 있다는 점과 버전 관리를 하나의 파일에서 할 수 있다는 점을 이유로 이 방법을 택했다. 
<br>

---

<br>
항상 maven만 사용해오다가 gradle을 큰 프로젝트에 사용해본 것은 처음이었는데 maven보다 자유도가 높은 느낌이었다. 그렇다보니 유명한 큰 오픈소스 프로젝트를 봐도 제각기 사용하는 방식이 달랐다. 이런 점이 gradle의 장점이기도 하고 단점이기도 한 것 같다.


### Reference
* https://medium.com/@jsuch2362/gradle-dependency-%EB%B6%84%EB%A6%AC%ED%95%98%EA%B8%B0-eb0c7c794b9c
* https://gradle.org/
