---
title: Effective Java 3rd ITEM 9
categories:
  - Effective Java 3rd
  - 2. Objects
tags:
  - effective_java
  - java
date: 2019-04-07 21:40:58
thumbnail:
---

# ITEM 9. Prefer try-with-resources to try-finally
## try-with-resources
java library에는 close method를 직접 호출해 닫아줘야하는 자원들이 많다.
예를 들어, InputStream, OutputStream, Connection 등이 있다. 
모두 close method를 호출해주어야 하지만 놓치기 쉬워 보통 finalizer를 안전망으로 활용하고 있다.
java 7 전에는 try-finally를 통해 자원이 제대로 닫힘을 보장하였다. 하지만 사용하는 자원이 여러개라면 try문이 겹쳐서 작성되게 된다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
알아보기 힘들고 코드가 지저분해지게 된다. 
또한 이 경우에는 finally문에서 발생한 Exception이 try문에서 발생한 Exception을 집어삼키게 된다.
Stack trace에서는 try문 내부의 Exception에 관한 정보는 남지 않아 디버깅을 어렵게 한다.

위와 같은 예제에 try-with-resouces 구문을 적용하게 되면 반드시 닫아줘야하는 자원들을 쉽게 관리할 수 있다. 
try-with-resouces를 사용하기 위해서는 해당 자원이 AutoCloseable interface가 구현되어 있어야 한다.
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
위 try-finally 예제를 try-with-resources로 구현한 예제이다. 깔끔하고 디버깅에도 훨씬 좋다. 
try-with-resouces에도 catch 구문은 똑같이 사용할 수 있다.
