---
title: Effective Java 3rd ITEM 50
categories:
  - Effective Java 3rd
  - 8. Methods
tags:
  - effective_java
  - java
date: 2019-06-27 17:58:54
thumbnail:
---

# ITEM 50. Make defensive copies when needed

클라이언트가 클래스의 불변식을 깨뜨릴 수 있기 때문에 방어적으로 프로그래밍해야 한다.

## 가변 필드
클래스가 가변 필드를 가질 경우, 외부에서 내부를 수정하는 일이 발생할 수 있으므로 주의해야 한다.
```java
public final class Period {
    private final Date start;
    private final Date end;

    // 생성자
    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    public Date start() { return start; }

    public Date end() { return end; }
}
```
위 Period 클래스는 불변으로 만들고자 설계되었다. 하지만 Date가 가변이기에 `start, end`의 필드가 외부에서 수정될 수 있는 위험이 있다.

### 생성자
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);
```
이렇게 생성자로 넘겨준 end 값을 외부에서 수정해버리면 Period 내부의 end 값도 수정되어버린다. 따라서 생성자에서 가변 객체를 넘겨 받았을때는 Defensive Copy를 해야 한다.
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
}
```

### 접근 메서드
Period가 제공하는 접근 메서드 `start(), end()`로도 내부 값을 수정할 수가 있다.
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
```

이렇게 내부 가변 객체를 꺼내 수정하는 것이 가능하기 때문에 접근 메서드도 Defensive Copy를 반환해야 한다.
```java
public end() { return new Date(end.getTime()); }
public start() { return new Date(start.getTime()); }
```

<br/>
물론 모든 경우에 Defensive copy를 해야 하는 것은 아니다. 클라이언트가 잘못 수정할 일이 없음을 신뢰한다면 Defensive copy 대신 해당 구성요소 수정 시에 책임이 클라이언트에 있다는 사실을 명시해주면 된다. 
