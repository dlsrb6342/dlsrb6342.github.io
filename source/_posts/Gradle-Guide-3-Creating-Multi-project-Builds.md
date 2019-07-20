---
title: Gradle Guide 3. Creating Multi-project Builds
categories:
  - build
  - gradle
tags:
  - gradle
date: 2019-07-20 13:02:38
thumbnail:
---

# Gradle Guide 3. Creating Multi-project Builds
> 공식 가이드 : https://guides.gradle.org/creating-multi-project-builds/

이번 가이드에는 gradle로 멀티 모듈의 프로젝트를 만들고 빌드하는 것에 대해 배운다.

## Create a root project
[Gradle Guide 2](https://dlsrb6342.github.io/2019/07/14/Gradle-Guide-2-Create-new-Gradle-builds/)에서 했던 것처럼 `gradle init`으로 새로운 root project를 만들어준다. 자동생성된 많은 파일 중에 `settings.gradle`을 보면 rootProject.name이 잘 들어가 있을 것이다.

```console
> gradle init
Starting a Gradle Daemon (subsequent builds will be faster)

BUILD SUCCESSFUL in 5s
2 actionable tasks: 2 executed

> cat settings.gradle
rootProject.name = 'gradle-3'
```

## Configure from above
내가 멀티 프로젝트의 장점이라 생각하는 점 중에 하나는 공통의 dependency나 설정에 대해 root project에 한번에 쓸 수 있다는 점이다. 
```groovy
// build.gradle

allprojects {
    repositories {
        jcenter()
    }
}

subprojects {
    version = '1.0'
}

```
### allprojects
allprojects는 root를 포함한 모든 sub project들에 configuration을 추가하는 block이다. 위에서는 모든 project에 jcenter repository를 추가하였다.

### subprojects
subprojects는 allprojects block에서 root를 제외한 설정이다. 위에서는 모든 sub project에 version을 1.0으로 설정하였다. 
> 다른 scripts block에 대해서는 [gradle docs](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#N159F0)를 보면 자세히 설명이 되어있다. 


## Add a Groovy library subproject
이제 sub project를 추가해보자. 가이드에서는 groovy library를 추가하는 방법에 대해 설명하고 있어서 groovy를 잘 모르지만 그냥 따라해봤다.

greeting-library란 폴더를 만들고 build.gradle 파일을 작성하였다.
```groovy
// greeting-library/build.gradle
plugins {
    id 'groovy'
}

dependencies {
    compile 'org.codehaus.groovy:groovy:2.4.10'

    testCompile 'org.spockframework:spock-core:1.0-groovy-2.4', {
        exclude module: 'groovy-all'
    }
}
```

이제 root project에 greeting-library를 포함시켜보자. 아까 rootProject.name이 쓰여있던 settings.gradle 파일에 써주면된다.
```groovy
rootProject.name = 'gradle-3'
include 'greeting-library'
```

기본 폴더 구조인 src/main/groovy/greeter, src/main/test/groovy/greeter를 만들어주고 간단한 코드를 작성해준다. 
```groovy
// greeting-library/src/main/groovy/greeter/GreetingFormatter.groovy
package greeter

import groovy.transform.CompileStatic

@CompileStatic
class GreetingFormatter {
    static String greeting(final String name) {
        "Hello, ${name.capitalize()}"
    }
}
```
~~테스트코드는 건너뛰겠다..~~

다 작성한 후에 `./gradlew build`를 쓰면 greeting-library 모듈이 빌드되는 것을 볼 수 있을 것이다. gradle은 sub project의 build task가 있다는것을 자동으로 감지하고 그것을 실행해준다. 이 부분이 gradle multi project의 큰 장점 중 하나라고 한다.
***=> Root project에 있는 task 이름과 같은 task가 sub project에 있다면 Gradle이 각각의 project의 같은 이름의 task를 함께 실행해줄 수 있다.***

## Add a Java application sub-project
이번엔 java application을 추가한다. 똑같이 폴더를 만들고 build.gradle 작성, code 작성을 하면 된다. 물론 root project의 settings.gradle에도 sub project로 추가하여야 한다. 코드는 생략하고 github에 올려두었다.

위의 gradle project와 다른 점은 mainClassName을 설정해주어야 한다는 점이다. Java application 모듈이기 때문에 gradle에게 진입점(main 함수가 있는 class)를 알려주어야 한다.
```groovy
// greeter/build.gradle
mainClassName = 'greeter.Greeter'
```

또 greeter-library에 있는 GreetingFormatter를 사용하기 때문에 greeting-library를 greeter의 dependency로 추가해주어야 한다.
```groovy
// greeter/build.gradle
dependencies {
    compile project(':greeting-library')
}
```
두 sub project가 완성되었으니 이제 build를 해보자. 문제없이 sub project 모두 build가 잘 될것이다.
<br/>

---

<br/>
본문 가이드에서는 Docs / Test 추가있지만 따로 다루진 않겠지만 한번 읽어보면 좋을 것 같다. 이 가이드 관련 코드는 https://github.com/dlsrb6342/gradle-study/tree/guide-3-multi-projects에 올려두었다. gradle guide를 공부하게 된 계기도 multi project를 구성하려다 막혀서 였는데 이 가이드를 통해서 어느정도 알아간 것 같다. scripts block에 대해서도 조금 찾아봐야 할 것 같다.

