---
title: jib-gradle-plugin으로 spring-boot application을 Docker에 배포하기
categories:
  - build
  - gradle
  - spring-boot
tags:
  - gradle
  - java
  - spring
date: 2019-07-26 16:39:46
thumbnail:
---

> Build container images for your Java applications.
https://github.com/GoogleContainerTools/jib

spring boot application을 Docker에 배포해보기 위해 검색했을 때 가장 먼저 뜨는 링크는 spring.io에 실려있는 [게시글](https://spring.io/guides/gs/spring-boot-docker/)이다. 이 게시글에서는 [com.palantir.docker plugin](https://github.com/palantir/gradle-docker)을 사용하고 Dockerfile을 작성하여 image build를 해주고 있다. 물론 착착 잘 되는 사람도 있었겠지만 내가 하는 환경에서는 잘 되지 않았다. Mac에서는 docker command의 PATH를 찾지 못하고 계속 오류를 내고 있었다. (참고 : https://github.com/palantir/gradle-docker/issues/162)
여러 시도를 하다가 포기하고 다른 plugin을 찾았는데 그것이 Google에서 만든 jib-gradle-plugin이다. 사용해본 결과, 정말 편하게 별다른 설정없이 docker에 application을 올릴 수 있었다. [jib github repo](https://github.com/GoogleContainerTools/jib)에 들어가보면 maven plugin도 있으니 maven을 사용하신다면 한번 확인해보기 바란다.


## jib-gradle-plugin

### Image build
```groovy
plugins {
  id 'com.google.cloud.tools.jib' version '1.4.0'
}
```
적용방법은 간단하다. build.gradle에 plugin으로 넣어주면 된다. 아무런 설정을 넣어주지 않는다면 image의 이름은 module 이름과 version 태그로 빌드된다. 물론 이름과 태깅 설정도 간단하다.
```groovy
bootJar {
    baseName = 'service-1'
    version =  '0.1.0'
}

jib {
    to {
      image = "${bootJar.baseName}:${bootJar.version}"
      tags = ['latest']
    }
}
```
난 bootJar를 통해 만들어지는 jar 파일과 이름을 같게 하기 위해 위와 같이 설정했다. 위 설정으로 이미지를 빌드하면 service-1:0.1.0과 service01:latest 두 개의 태그가 만들어진다. 

### Push to Docker hub
Docker hub나 원하는 docker registry에 Push하는 것도 간단하다. jib github repo를 확인해보면 다양한 cloud service에 push하는 방법들이 나와있다. 난 Docker hub로 Push하도록 설정해보았다.
```groovy
ext.dockerId = 'dlsrb6342'

jib {
    to {
        image = "${dockerId}/${bootJar.baseName}:${bootJar.version}"
        username = "USER_ID"
        password = "PASSWORD"
        tags = ['latest']
    }
} 
```
위의 이미지 및 태그 설정했던 부분에 docker hub ID를 넣어주고 username / password 값을 추가해주면 된다. 하지만 이 방법은 password가 다 노출될 수 있기 때문에 [docker-credential-helper](https://github.com/docker/docker-credential-helpers)를 사용하기를 권장한다. docker-credential-helper를 사용하면 username과 password를 직접 적어두지 않아도 된다. 나는 OSX를 사용 중이기에 osxkeychain을 설치했다.
```sh
brew install go
export GOPATH=/usr/local/Cellar/go/1.12.7
go get github.com/docker/docker-credential-helpers
cd $GOPATH/docker/docker-credentials-helpers
make osxkeychain
export PATH="${PATH:+${PATH}:}/usr/local/Cellar/go/1.12.7/src/github.com/docker/docker-credential-helpers/bin"
```
go lang도 설치되어있지 않아 다 설치해주었다. 다 설치해주고 나서 `docker-credential-osxkeychain list`를 입력해보면 잘 설정되었는지 볼 수 있을 것이다. 

여기서 하나 정리해야 하는것이 jib-gradle-plugin이 가지고있는 명령들이다. 
* jib : Build your container image
* jibBuildTar : You can build and save your image to disk as a tarball
* jibDockerBuild : Jib can also build your image directly to a Docker daemon

jibBuildTar는 이름 그래도 image를 tar 파일로 만들어주는 명령이다. jib과 jibDockerBuild는 하는 일은 거의 똑같다. 한 가지 다른 점은 jib은 docker hub와 같은 원격 registry에도 image를 push해준다는 점이다. jibDockerBuild를 사용하면 자신의 환경에 설치되어있는 docker command를 사용하기 때문에 local에만 image가 올라가게 된다.

우린 remote registry에 올릴 것이기 때문에 jib 명령을 사용하면 된다.


### Example
코드는 https://github.com/dlsrb6342/blog-sample/tree/master/gradle-docker-example에 올려두었다. service1, service2 2개의 모듈이 있고 이를 jib-gradle-plugin으로 빌드하고 docker-compose를 사용해 두 모듈을 함께 실행해보는 예제이다.

위에 얘기한 설정들은 각 모듈의 build.gradle에서 확인할 수 있고 root project의 build.gradle을 보면 편의를 위해 직접 task를 추가해었다.
```groovy
task dockerDownAll(type:Exec) {
    commandLine 'docker-compose', 'down'
}

task dockerRunAll(type:Exec) {
    dependsOn dockerDownAll
    dependsOn subprojects.jibDockerBuild
    commandLine 'docker-compose', 'up', '-d'
}
```
* dockerRunAll : subprojects를 모두 이미지로 빌드하고 docker-compose를 사용해 실행시키는 명령
* dockerDownAll : 모든 모듈을 중지시키는 명령

```console
> ./gradlew dockerRunAll
...
> Task :dockerRunAll
Creating network "gradle-docker-example_default" with the default driver
Creating service-1 ... 
Creating service-2 ... 
Creating service-2 ... done
BUILD SUCCESSFUL in 30s

> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
3053876b03ee        service-2:latest    "java -cp /app/resou…"   10 seconds ago      Up 7 seconds        0.0.0.0:8082->8082/tcp   service-2
983cb559414e        service-1:latest    "java -cp /app/resou…"   10 seconds ago      Up 7 seconds        0.0.0.0:8081->8081/tcp   service-1

> curl http://127.0.0.1:8081/toService2
{"service2":"success","service1":"toService2-passed"}
```
dockerRunAll 명령을 실행하게 되면 service1, service2 모듈이 실행되는 것을 볼 수 있다. 

Docker hub에 push하는 방법은 각 모듈에서 jib 명령을 실행해주면 된다. 
```console
> ./gradlew :service1:jib
...
Built and pushed image as dlsrb6342/service-1:0.1.0, dlsrb6342/service-1
Executing tasks:
[==============================] 100.0% complete


BUILD SUCCESSFUL in 17s
3 actionable tasks: 2 executed, 1 up-to-date
```

{% asset_img "dockerhub.png" "docker hub" %}
push된 이미지는 Docker hub 페이지에서 확인할 수 있다.


## 정리
com.palantir.docker plugin을 사용하면서 힘들었던 점이 많았다. Dockerfile을 작성하는 문법도 정확히 몰라서 계속 찾아봐야 했고 심지어 다 작성한 후에는 무슨 이유인지 모르게 docker command의 PATH를 찾지 못하는 이슈도 있었다. 이런 일이 있어서인지 jib-gradle-plugin은 정말 편하다는 느낌을 많이 받았다. java application을 docker image로 빌드할 때는 고민없이 이 plugin을 사용할 것 같다.
