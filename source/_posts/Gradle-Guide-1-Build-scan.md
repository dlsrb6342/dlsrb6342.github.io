---
title: Gradle Guide 1. Build scan
categories:
  - build
  - gradle
tags:
  - gradle
date: 2019-07-13 18:37:53
thumbnail:
---

{% asset_img "Gradle.png" "Gradle Logo" %}

gradle.. spring을 처음 사용했을때 gradle을 썼었다(뭣도모르고 좋다니 쓴 느낌). 그 이후부터는 항상 maven을 썼고 이제는 maven의 xml이 너무 편해졌다. 그래도 혼자 공부할때나 토이 프로젝트를 할 때는 gradle을 썼다. 하지만 **갓**텔리제이가 처음 initializing 해주는 gradle 파일로도 충분했기에 따로 공부할 생각을 안했다. 

결국 항상 가져다만 쓰던 gradle에 발목이 잡혔다. 멀티모듈 프로젝트를 구성하고 root module의 build.gradle을 어떻게 사용하는지 찾아봤는데 도대체가 답변마다 다 가지각색으로 다르게 사용하고 있다. 이걸 보고나서 '아 이젠 공부하고 써야겠다' 라는 마음이 들어 [gradle 공식문서의 guide](https://gradle.org/guides/#getting-started)들을 공부하기로 했다.

가이드 공부에 사용된 gradle 프로젝트 IntelliJ spring initializr로 생성한 간단한 spring 프로젝트이다. 
https://github.com/dlsrb6342/gradle-study
```shell
> ./gradlew --version

------------------------------------------------------------
Gradle 5.4.1
------------------------------------------------------------

Build time:   2019-04-26 08:14:42 UTC
Revision:     261d171646b36a6a28d5a19a69676cd098a4c19d

Kotlin:       1.3.21
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.13 compiled on July 10 2018
JVM:          12.0.1 (Azul Systems, Inc. 12.0.1+12)
OS:           Mac OS X 10.14.4 x86_64
```

# Gradle Guide 1. [Creating Build Scans](https://guides.gradle.org/creating-build-scans/)
1번부터 처음보는 것이다. 여태까지 정말 막 써 왔구나를 느낀다. 

이 가이드에서는 별다른 수정이나 설정없이 build scan ad-hoc을 적용하는 방법에 대해 배운다.

## Upper gradle 4.3
gradle 4.3 이후부터는 별다른 설정없이 `./gradlew build --scan`을 입력하면 build scan을 사용할 수 있다고 한다. 는 개구라였다 ㅋㅋㅋㅋ 안된다!!! gradle 관련 글들이 너무 가지각색이라 공식 문서로 찾아왔는데 공식문서도 안된다 ㅋㅋㅋㅋㅋㅋ `./gradlew build --help`를 쳐보니 build scan plugin을 넣어야한다고 한다.

```shell
> ./gradlew biuld --help

...
--scan    Creates a build scan. Gradle will emit a warning if the build scan plugin has not been applied. (https://gradle.com/build-scans)
...

```
사실 이 글을 다 쓰고 build-scan plugin 제거하고 다시 `./gradlew build --scan`을 입력해봤는데 plugin없이도 된다.. 뭐가 문제였는지 모르겠다 ㅠㅠ

## build-scan plugin
추가하는 방법은 간단하다. build.gradle에 plugins 안에 넣어주면 된다. 만약 현재 사용중인 plugin이 있다면 build-sacn을 가장 위에 놓아야 한다. 아래 넣어도 똑같이 동작하지만 모든 build 과정을 scan하지 못하게 된다.
```groovy
// build.gradle
plugins {
    id 'com.gradle.build-scan' version '2.1' 
}
```
plugin을 추가하고 나서 `./gradlew build --scan`을 입력하면 잘 동작한다.
```shell
> ./gradlew build --scan

...
Publishing a build scan to scans.gradle.com requires accepting the Gradle Terms of Service defined at https://gradle.com/terms-of-service. Do you accept these terms? [yes, no] yes

Gradle Terms of Service accepted.

Publishing build scan...
https://gradle.com/s/{어쩌구저쩌구}
```
아래 주소에 url에 들어가면 이메일 주소를 입력하게 되고 scan결과를 보여주는 url을 메일로 받을 수 있다. 보여주는 것도 깔끔하고 실행시간, 콜솔 로그 등등 엄청 많은 정보가 들어있어 디버깅하기 좋아보인다.

{% asset_img "email.png" "Insert email" %}
{% asset_img "discover.png" "Check your email" %}
{% asset_img "scan.png" "Scan Result" %}

## License Agreement
build scan의 귀찮은 점은 build가 끝나고 항상 license agreement에 yes라고 타이핑해줘야 한다는 점이다. 이것 또한 항상 동의할 수 있도록 설정할 수 있다.
```groovy
// build.gradle
buildScan {
    termsOfServiceUrl = 'https://gradle.com/terms-of-service'
    termsOfServiceAgree = 'yes'
}
```

<br/>
기존 프로젝트에 별다른 추가 사항은 없지만 따로 관리하기 위해 이번 build-scan 가이드는 https://github.com/dlsrb6342/gradle-study/tree/guide-1-build-scan에 올려두었다.
gradle build가 너무 오래 걸리거나 혹은 오류를 뱉어낸다면 정말 유용한 툴이 될 수 있을 것 같다. **첫 가이드부터 좋은 글인거 같아 다음 가이드들이 기대된다.**
