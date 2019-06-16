---
title: Effective Java 3rd ITEM 39
categories:
  - Effective Java 3rd
  - 5. Enums and Annotations
tags:
  - effective_java
  - java
date: 2019-06-16 14:53:46
thumbnail:
---

# ITEM 39. Prefer annotations to naming patterns

## Naming Patterns
전통적으로 무언가 특별히 다뤄야할 프로그램 요소에는 구분되는 naming patterns을 사용해왔다. 예를 들어 JUnit3에서는 메서드 이름을 test로 시작하게 했다.

### 단점
1. 오타가 있으면 무시하고 지나친다.
2. 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다. JUnit의 경우, 메서드만 가능한데 클래스에 Test를 붙이고 그 안에 메서드들이 실행되길 기대할 수도 있다.
3. 프로그램 요소를 매개변수로 전달할 방법이 없다.

## Annotations
### meta-annotations
annotation 선언에 다는 annotation을 meta-annotation이라고 한다. 
#### Retention
```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```
Retention은 위 RetentionPolicy 값을 들고 있다. 해당 annotation을 언제까지 유지해야 하는지를 뜻한다.

#### Target
```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```
Target은 위 ElementType을 들고 있고 annotation이 어디에 사용가능한지를 뜻한다.

#### Repeatable
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    /**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
    Class<? extends Annotation> value();
}

```
Repeatable은 하나의 프로그램 요소에 같은 Annotation을 여러개 달 수 있음을 뜻한다. Repeatable의 value로는 반복 가능하게 만든 annotation을 반환하는 '컨테이너 annotation'을 넣어줘야 한다. 컨테이너 annotation에는 반복된 annotation의 배열을 return하는 value 메서드를 정의해야 한다.

### Annotation 처리 도구
보통 Annotation은 클래스나 메서드에 직접적인 영향을 주지는 않고 그저 처리 도구에게 추가 정보를 제공하는 역할만 한다. Annotation 처리 도구는 원하는 annotation이 달려있는 프로그램 요소를 특별하게 처리해주는 역할이다.

<br/>
Annotation으로 할 수 있는 일은 naming pattern으로 처리할 이유는 없다. Annotation을 사용하도록 하자!
