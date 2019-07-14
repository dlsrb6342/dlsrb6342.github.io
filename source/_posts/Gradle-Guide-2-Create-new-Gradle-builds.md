---
title: Gradle Guide 2. Create new Gradle builds
categories:
  - build
  - gradle
tags:
  - gradle
date: 2019-07-14 14:40:56
thumbnail:
---

# Gradle Guide 2. Create new Gradle builds
> 공식 가이드 : https://guides.gradle.org/creating-new-gradle-builds/

이번 가이드에서는 command line에서 gradle project를 생성하고 기본적인 명령어를 배운다.

## gradle init
```console
# kotlin
# gradle init --dsl kotlin

# groovy
> gradle init
Starting a Gradle Daemon (subsequent builds will be faster)

BUILD SUCCESSFUL in 6s
2 actionable tasks: 2 executed

> tree .
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle

2 directories, 6 files
```
* build.gradle : project의 configuration을 담고 있는 build script이다.
* gradle-wrapper.jar : Gradle Wrapper을 들고있는 executable jar 이다.
* gradle-wrapper.properties : Gradle Wrapper의 설정을 가진 파일이다.
* gradlew : UNIX 계열에서 사용할 수 있는 Gradle Wrapper script이다.
* gradlew.bat : Windows에서 사용할 수 있는 Gradle Wrapper script이다.
* settings.gradle : gradle build를 위해 gradle을 설정하는 script이다.

> Gradle Wrapper란 선언된 gradle version으로 gradle을 실행하는 script이다. 필요하다면 실행 전에 gradle 다운로드도 실행한다. gradle project의 빌드에는 Gradle Wrapper를 사용하도록 권장된다.

## Task / Plugin
새로운 Task를 만들고 실행하는 방법에 대한 설명이 있지만 간단해서 생략하겠다. 여러가지 gradle task type을 알고 싶다면 [Documentation](https://docs.gradle.org/4.10.3/dsl/)의 왼쪽 목록 중 Task Type을 보면 된다.

Plugin 적용은 이전에 작성한 [build-scan을 적용하는 글](https://dlsrb6342.github.io/2019/07/13/Gradle-Guide-1-Build-scan/)을 참고하면 된다.

## Explore build
* `./gradlew tasks` : 사용 가능한 모든 task를 보는 명령이다.
* `./gradlew properties`: 프로젝트의 모든 properties를 보는 명령이다.

## Debug build
어떤 task든 실행할 때 `--scan` 옵션을 주면 build scan이 적용되어 웹UI로 task의 과정을 모두 디버깅할 수 있다. build scan을 적용하는 방법은 [이전 글](https://dlsrb6342.github.io/2019/07/13/Gradle-Guide-1-Build-scan/)을 확인하면 알 수 있다.

<br/>
gradle project 생성했을 때의 기초적인 명령들을 알 수 있었다. **`./gradlew tasks`나 `./gradlew properties`는 정말 유용하게 사용할 수 있을 것 같다.**
