---
layout: post
title:  "Java8 in Action : CHAPTER9"
date:   2018-02-17
desc: "CHAPTER 9. 디폴트 메서드"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---

## CHAPTER 9. 디폴트 메서드

- 자바8 API에도 List 인터페이스에 sort 같은 메서드를 추가했으므로 문제가 발생 할수 있음

- 기본 구현을 포함하는 인터페이스를 정의하는 두가지 방법 제공

    - 인터페이스 내부에 **정적 메서드**<sup>static method</sup>를 사용하는 것
    - 인터페이스의 기본 구현을 제공할 수 있도록 **디폴트 메서드**<sup>default method</sup>라는 기능을 사용하는 것

- 메서드 구현을 포함하는 인터페이스를 정의 할수 있음, 결과적으로 기존 인터페이스를 구현하는 클래스는 자동으로 인터페이스에 추가된 
새로운 메서드의 디폴트 메서드를 상속 받게 됨

> List 인터페이스의 sort

- List 인터페이스의 sort 메서드는 자바8에서 새로 추가된 메서드

```java
default void sort(Compatator<? super E> c){
    Collections.sort(this.c);
}
```

- default 키워드는 해당 메서드가 디폴트 메서드임을 가르키며, sort 메서드는 Collections.sort 메서드를 호출

> Collection 인터페이스의 stream

```java
default Stream<E> stream(){
    return StreamSupport.stream(spliterator(), false);
}
```

- stream 메서드는 내부적으로 StreamSupport.stream이라는 메서드를 호출해서 스트림을 반환
- stream 메서드의 내부에서는 Collection 인터페이스의 다른 디폴트 메서드 spliterator도 호출

> 바이너리 호환성, 소스 호환성, 동작 호환성

- 바이너리 호환성 : 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황을 **바이너리 호환성**이라고 함
(바이너리 실행에는 인증<sup>verification</sup>, 준비<sup>preparation</sup>, 해석<sup>resolution</sup>등의 과정이 포함)
- 소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음을 의미
- 동작 호환성 : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행한다는 의미


> 디폴트 메서드란?

- 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 **디폴트 메서드**<sup>default method</sup>를 제공
- default 라는 키워드로 시작하며 다른 클래스에 선언된 메서드처럼 메서드 바디를 포함

```java
public interface Sized{
    int size();
    default boolean isEmpty(){      <--디폴트 메서드
        return size() == 0;
    }
}
```
- Sized 인터페이스를 구현하는 모든 클래스는 isEmpty의 구현도 상속 받음 즉, 인터페이스에 디폴트 메서드를 추가하면 소스 호환성이 유지 됨
- Predicate, Function, Comparator 등 많은 함수형 인터페이스도 Predicate.and 또는 Function.andThen 같은 다양한 디폴트 메서드를 포함

> 추상 클래스 와 자바8 의 인터페이스

1. 클래스는 하나의 추상 클래스만 상속, 인터페이스 여러개 구현
2. 추상클래스는 인스턴스 변수(필드)로 공통 상태를 가짐, 하지만 인터페이스는 인스턴스 변수를 가질 수 없음

> 선택형 메서드

- Iterator는 hasNext와 next뿐 아니라 remove메서드도 정의
- remove 기능은 잘 사용하지 않으므로 자바8 이전에는 remove 기능을 무시
- 결과적으로 Iterator 를 구현하는 많은 클래스에서는 remove에 빈 구현을 제공

```java
interface Iterator<T> {
    boolean hasNext();
    T next();
    default void remove(){
        throw new UnsupportedOperationException();
    }
}
```

- 위와 같이 자바8 의 Iterator 인터페이스에서 remove메서드를 기본 구현의 제공되므로 Iterator 인터페이스를 구현하는 클래스는 빈 remove 메서드를 
구현 할 필요가 없어졌고, 붎필요한 코드를 줄일 수 있음
 
> 동작 다중 상속

- 디폴트 메서드를 이용하면 기존에는 불가능 했던 **동작 다중 상속** 기능도 구현
- 자바에서 클래스는 한 개의 다른 클래스만 상속, 인터페이스는 여러 개 구현

```java
public class ArrayList<E> extends AbstractList<E>             <-- 한 개의 클래스를 상속 받음
         implements List<E>, RandomAccess, Cloneable,
            Serializable, Iterable<E>, Collection<E>{         <-- 여섯 개의 인터페이스를 구현
    
}
```

> 다중 상속 형식

- ArrayList는 한 개의 클래스를 상속 받고, 여섯 개의 인터페이스를 구현
- ArrayList는 AbstractList, List, RandomAccess, Cloneable, Serializable, Iterable, Collection 의 **서브형식**<sup>subtype</sup>
이 되며, 디폴트 메서드를 사용하지 않아도 다중 상속을 활용 할수 있음
- 인터페이스가 구현을 포함, 클래스는 여러 인터페이스에서 동작(구현 코드) 상속

> 해결 규칙

- 다른 클래스나 인터페이스로 부터 같은 시그너처를 갖는 메서드를 상속받을 때는 세 가지 규칙을 따라야 함

1. 클래스가 항상 이김, 클래스나 슈퍼클래스에서 정의한 메서드가 디폹트 메서드 보다 우선권을 가짐
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이김, 상속관계를 갖는 인터페이스에서 같은 시그너처를 갖는 메서드를 정의할 때는 서브인터페이스가 이김.
즉, B가 A를 상속 받는다면 B가 A를 이김
3. 여전히 디폴트 메서드의 우선순의가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드 하고 호출 해야함










 