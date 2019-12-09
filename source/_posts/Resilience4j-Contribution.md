---
title: 오픈소스 Contribution의 시작(Resilience4j Contribution)
categories:
  - resilience4j
tags:
  - java
  - resilience4j
date: 2019-12-09 01:15:28
thumbnail:
---

# Resilience4j Contribution

resilience4j를 처음 알게 되고 사용한 지 6~7개월 정도 지났다. 그 당시 새로 진행하고 있던 프로젝트에 적용할 CircuitBreaker 구현체를 찾고 있었고 지난 프로젝트에 사용한 Hystrix가 maintenance 모드에 들어갔기에 resilience4j를 사용했다. Hystrix와의 차이점은 지난 [포스트](https://dlsrb6342.github.io/2019/06/03/Resilience4j%EB%9E%80/#Netflix-Hystrix%EC%99%80-%EB%8B%A4%EB%A5%B8%EC%A0%90)에 작성해두었다. 그리고 7개월이 지난 지금은 resilience4j의 maintainer로 활동 중이다.


## 시작.
resilience4j에 처음으로 한 contribution은 오타 수정이다. resilience4j를 사용해보기 위해 guide나 javadoc을 꼼꼼하게 다 읽어보았고, 그러다 보니 보이는 오타들이 있었다. 이슈를 남기기보다는 간단한 문서 수정이기에 바로 PR을 올렸었다.
* https://github.com/resilience4j/resilience4j/pull/482
* https://github.com/resilience4j/resilience4j/pull/535

작은 수정들이었지만 내가 open source contribution에 흥미를 갖게 해주는 계기가 되었다. 이후에는 resilience4j에 올라오는 이슈들을 하나하나 다 읽어보고 혹시나 내가 기여할 수 있는 부분을 찾아 헤맸다. 이 시기에 진행하고 있던 프로젝트에 resilience4j를 사용하고 있었으므로 사용 중에 불편했던 점이 생기기도 했고 여러 트러블 슈팅 경험도 생겨서 이슈에 코멘트를 달아주고 해결 방법에 대해 대화를 나누는 일들이 많아졌다.

## spring-cloud
진행 중인 프로젝트가 spring-cloud의 library들을 많이 사용했었다. 이전 포스트들을 보면 알겠지만 spring-cloud-gateway / spring-cloud-sleuth / spring-cloud-config 등에 대한 글을 몇 개 작성했었고 spring-cloud-gateway에는 여러 커밋을 하기도 했다. 이 시기에 참 열심히 살았던 것 같다. 퇴근해서는 오픈 소스만 쳐다봤고 contribution 자체가 재밌었다.


{% asset_img "spring-cloud-gateway.png" "spring-cloud-gateway contribution" %}


spring-cloud 이야기를 꺼낸 이유는 resilience4j에 첫 소스 코드에 대한 커밋은 spring-cloud 덕분이었기 때문이다. 이 당시(2019.06)에는 spring-cloud-circuitbreaker가 아직 incubating 중이었기 때문에 resilience4j를 spring-cloud와 사용하기에 불편함이 있었다. 이에 대해 똑같은 생각을 했던 사람이 이슈를 남겼고 코멘트로 트러블 슈팅을 했던 경험을 공유했다.
* https://github.com/resilience4j/resilience4j/issues/546

이에 resilience4j maintainer가 PR을 해줄 수 있냐고 물었고 난 정말 기쁜 마음으로 코드를 짜고 컨트리뷰팅을 했다. 이 이슈에 대한 [PR](https://github.com/resilience4j/resilience4j/pull/550)은 7월 말에 처음 open 하였고 9월초에 머지되었다. 한 달이 넘는 기간, 46개의 코멘트를 주고받으며 리뷰를 받고 머지될 수 있었다. 중간중간 다른 이슈들에 대해서도 PR을 보냈고 여러 PR들이 머지되었다.

## 초대.
여러 가지 이슈도 만들고 PR을 보내가며 커밋을 6개 정도 쌓아 갔을 때쯤, resilience4j maintainer로부터 slack에 초대를 받았다. resilience4j maintaining을 위한 채널이었다.
* https://github.com/resilience4j/resilience4j/pull/606#issuecomment-530386425

resilience4j maintainer로써 활동하게 된 것이다. 이 초대를 받고 resilience4j의 member가 된 것이 9월 중순이었으니 resilience4j에 관심을 갖고 여러 contribution을 한 지 3개월 만에 member가 될 수 있었다.

{% asset_img "contribution.png" "Resilience4j Contribution" %}

## 이후.
resilience4j의 member가 되고 난 이후에는, 사실 commit은 많이 하지 못했다. 약간의 번아웃이 오기도 했고 여러 가지 개인 사정으로 인해 많은 시간을 쏟지 못했다. commit은 못했지만 그 대신 issue에 대해 답변을 준다거나 PR을 리뷰해주는 등 resilience4j가 더 성숙해지고 커질 수 있도록 노력했다. 6월 v0.16.0이던 프로젝트가 이젠 v1.2.0을 달고 릴리즈 되었고 spring-cloud-circuitbreaker에서 다른 CircuitBreaker 구현체들은 제외하고 resilience4j만을 지원하는 등, 정말 많은 성장을 한 것 같다. 처음 resilience4j를 만나고 간단한 [포스트](https://dlsrb6342.github.io/2019/06/03/Resilience4j%EB%9E%80/)를 작성했었는데 이제는 많은 설정들이 달라졌고 추가되었다. 이 부분에 대해서는 이어질 포스트들에서 다뤄볼 예정이고 앞으로의 버전 릴리즈 시에도 변경사항에 대한 포스트를 작성해볼 생각이다.


## Open Source Contributing

나에게는 굉장히 신선한 경험이었다. spring-cloud-gateway에서 받은 PR 리뷰도 그렇고 resilience4j에서 받은 리뷰 모두 내가 생각하지 못했던 부분에 대한 리뷰를 받을 수 있었다. 라이브러리 개발자가 아닌 서비스 개발자로서 개발해왔기 때문에 내가 만든 코드를 사용하는 유저들에 대한 생각을 하지 못했다. 이런 내가 고려하지 못했던, 놓쳤던 부분들에 대한 리뷰들이 나를 더욱 open source contribution에 빠져들게 했고 한층 더 성장시켜주었다. 직장에 다니고 대학 시절과는 다르게 개발이 일이 되면서 흥미나 의욕을 조금씩 잃어가는 느낌이 있었는데 open source contribution을 하면서 대학 시절보다도 더 의욕이 넘치게 된 것 같다.
Open Source Contribution를 처음 시작하려고 하는 분들에게는 쉽게 생각하자고 말씀드리고 싶다. 나도 처음엔 얼굴에 철판을 깔았는지 이슈를 올리고 아무 PR이나 보내고 이슈에 코멘트를 달고 많이 나대고 다녔었다. 그러다 보니 contribution 자체가 어렵지 않게 느껴졌고 편하고 쉽게 생각할 수 있게 되었다. 또 시작하려고 할 때 뭐부터 해야 하지 라고 생각이 들 때는 문서를 꼼꼼하게 읽고 파악하는 것과 조금이라도 직접 사용해보는 것이 중요해 보인다. 그래야 PR도 자신 있게 올릴 수 있고 문제점이 눈에 들어오기도 하는 것 같다. 더 많은 사람들이 오픈 소스를 그저 사용해보는 것만이 아닌 여러 가지 방법으로 기여하면서 나와 비슷한 경험들을 했으면 좋겠다. 
