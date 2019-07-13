---
title: Spring Content Logging
categories:
  - spring-mvc
tags:
  - spring
  - java
date: 2019-07-02 21:55:05
thumbnail:
---

# Spring Content Logging

요구사항은 간단하다. Request body와 Response body를 로깅하는 것이다. 생각보다 간단했다.

## Servlet Wrapper
spring boot에서는 기본적으로 [RequestPacade](https://tomcat.apache.org/tomcat-9.0-doc/api/org/apache/catalina/connector/RequestFacade.html) / [ResponsePacade](https://tomcat.apache.org/tomcat-9.0-doc/api/org/apache/catalina/connector/ResponseFacade.html)로 SevletRequest / ServletResponse가 들어온다. 이를 Content Caching Wrapper로 감싸주기만 하면 된다.

spring-web에 이미 [ContentCachingRequestWrapper](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/util/ContentCachingRequestWrapper.java)와 [ContentCachingResponseWrapper](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/util/ContentCachingResponseWrapper.java)가 구현되어있다. 
물론 원하는대로 직접 구현해도 되지만 spring이 예쁘게 구현해두었으니 활용해보자.

### CustomServletWrapperFilter
```java
@Component
public class CustomServletWrappingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {

        ContentCachingRequestWrapper wrappingRequest = new ContentCachingRequestWrapper(request);
        ContentCachingResponseWrapper wrappingResponse = new ContentCachingResponseWrapper(response);

        chain.doFilter(wrappingRequest, wrappingResponse);

        wrappingResponse.copyBodyToResponse();
    }
}
```
Servlet을 wrapping해주는 filter를 구현했다. OncePerRequestFilter를 상속받아 한 reuqest당 한번의 실행만 되도록 보장하였다. spring bean으로 등록해두면 Filter 적용은 끝난다.

가장 중요한 점은 마지막 줄에 있다. `wrappingResponse.copyBodyToResponse()`로 실제 response body에다가 값을 넣어주어야 한다. 이 코드를 안넣어주면 클라이언트가 아무 응답도 받지 못하게 된다.

### LoggingInterceptor
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class LoggingInterceptor extends HandlerInterceptorAdapter {

    private final ObjectMapper objectMapper;

    @Override
    public void afterCompletion(
            HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {

        final ContentCachingRequestWrapper cachingRequest = (ContentCachingRequestWrapper) request;
        final ContentCachingResponseWrapper cachingResponse = (ContentCachingResponseWrapper) response;

        log.info(
                "ReqBody : {} / ResBody : {}",
                objectMapper.readTree(cachingRequest.getContentAsByteArray()),
                objectMapper.readTree(cachingResponse.getContentAsByteArray())
        );

    }
}
```
logging은 interceptor로 구현하였다. 이 코드는 간단하게 구현하느라 validation들을 모두 제외하였다. 직접 구현할 때는 wrapping이 안됐을 경우나 caching된 content가 없을 경우 등, 검증해야 할 부분이 많다. 
caching된 content는 byte array로 들고 있기 때문에 objectMapper를 이용해 JsonNode로 읽어주고 로깅했다. 

## Test

Sample code는 https://github.com/dlsrb6342/blog-sample/tree/master/spring-content-caching 에서 확인할 수 있다.
아래 예시와 같이 request body / response body logging을 할 수 있다. 
```
// request
curl -X POST \
  http://127.0.0.1:8080/test \
  -H 'Content-Type: application/json' \
  -d '{
	"foo": "foo",
	"bar": "bar"
}'

// response
{"result":"test","code":100}

// logging result
2019-07-02 22:44:21.195  INFO 15920 --- [nio-8080-exec-1] c.c.caching.demo.LoggingInterceptor      : ReqBody : {"foo":"foo","bar":"bar"} / ResBody : {"result":"test","code":100}
```

