---
title: spring cloud gateway 구조
categories:
  - spring-cloud
  - spring-cloud-gateway
tags:
  - spring
  - spring-cloud
  - java
date: 2019-05-14 23:11:51
thumbnail:
---

https://cloud.spring.io/spring-cloud-gateway/spring-cloud-gateway.html#gateway-how-it-works

spring-cloud-gateway의 공식문서 how it works를 보다가 조금더 자세하게 Diagram을 그려봤다.
{% asset_img "spring-cloud-gateway.png" "spring-cloud-gateway-diagram" %}


### 1. ReactorHttpHandlerAdapter.apply(request, response)
HttpServerRequest와 HttpServerResponse를 HttpWebHandlerAdapter에 ReactorServerRequest / ReactorServerResponse로 NettyBufferFactory와 함께 wrapping해 전달해준다.

### 2. HttpWebHandlerAdapter
ReactorHttpHandlerAdapter로부터 받은 ReactorServerRequest / ReactorServerResponse를 ServerWebExchange로 만들어 DispatcherHandler에게 넘겨준다.

### 3. DispatcherHandler
HandlerMappings - RouterFunction / RequestMapping / **RoutePredicateHandlerMapping**
* HandlerMapping에 WebFlux가 지원하는 RouterFounction과 RequestMapping이 있고 그 이외의 spring-cloud-gateway의 RoutePredicateHandlerMapping이 있다.  

Gateway로 들어오는 request는 RoutePredicateHandlerMapping이 handler를 찾게 된다. 
* RoutePredicateHandlerMapping.getHandler(ServerWebExchange).getHandlerInternal(ServerWebExchange)

### 4. RoutePredicateHandlerMapping
getHandlerInternal()
* lookupRoute() method를 통해 Route마다 설정된 Predicate를 적용해 알맞은 Route instance를 찾는다.
* 여기서 찾은 Route instance를 ServerWebExchange에 attribute로 넣어주고 GlobalFilter를 가지고 있는 FilteringWebHandler를 return한다.
* FilteringWebHandler는 spring-web이 아닌 spring-cloud-gateway에 속한 class를 의미한다.
  * https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/server/handler/FilteringWebHandler.java
  * https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/FilteringWebHandler.java

### 5. DispatcherHandler
invokeHandler()
* RoutePredicateHandlerMapping에서 return 받은 FilteringWebHandler를 지원하는 handlerAdapter를 찾고 handling한다. 
* FilteringWebHandler를 지원하는 handlerAdapter는 SimpleHandlerAdapter이다.

### 6. SimpleHandlerAdapter
SimpleHandlerAdapter.handle()은 항상 Mono.empty()를 return한다.
그러므로 DispatcherHandler는 handlerAdapter로부터 return받아 handleResult를 call하게 되는데 값이 empty이기 때문에 DispatcherHandler.resultHandlers는 아무 일도 하지 않게 된다.

### 7. FilteringWebHandler
handle()
  * Route instance에서 GatewayFilter를 가져와 GlobalFilters와 Order에 따라 chaning해서 수행하게 된다.  
  * For example as GatewayFilter, AddRequestHeaderGatewayFilterFactory, and so on..
  * NettyWriteResponseFilter
    * NettyWriteResponseFilter는proxy한 reponse를 ServerWebExchange.response에 쓰는 역할을 한다.

**And Finally HttpServerResponse will be passed.**
