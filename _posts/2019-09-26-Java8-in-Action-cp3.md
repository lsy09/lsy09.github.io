---
layout: post
title:  "Java8 in Action : CHAPTER_03"
date:   2019-09-26
desc: "CHAPTER 3. 람다 표현식"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---


## CHAPTER 3. 람다 표현식

### 3.1 람다란 무엇인가?

- 메서드로 전달할 수 있는 익명 함수를 단순화
    - 익명  
    보통의 메서드와 달리 이름이 없으므로 익명이라 표현
    - 함수  
    람다는 메서드처럼 특정 클래스에 종속 되지 않으므로 함수라고 함  
    하지만 메서드처럼 파라미터 리스트, 바디, 반환형식, 가능한 예외 리스트를 포함
    - 전달  
    람다 표현식을 메서드 인수로 전달, 변수로 저장 가능
    - 간결성  
    익명 클래스처럼 많은 자질구레한 코드를 구현할 필요 없음

`기존 코드`
```java
Comparator<Apple> beWeight = new Comparator<Apple>(){
    public int vompare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```

`람다`
```java
Comparator<Apple> beWeight = 
        (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

`자바8에서 지원하는 다섯 가지 람다 표현식 예제`
```
1. (String s) -> s.length()
String 형식의 파라미터 하나를 가지며 int를 반환
람다 표현식에는 return이 함축되어 있으므로 return문을 명시적으로 사용하지 않아도 됨

2. (Apple a) -> a.getWeight() > 150
Apple 형식의 파라미터 하나를 가지며 boolean(사과가 150그램보다 무거운지 결정)반환

3. (int x, int y) -> {
        System.out.println("Result:");
        System.out.println(x+y);
    }
int 형의 파라미터 두 개를 가지며 리턴값이 없음(void리턴).
람다 표현식은 여려 행의 문장을 포함

4. () -> 42
파라미터가 없으며 int를 반환

5. (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
Apple 형식의 파라미터 두 개를 가지며 int(두 사과의 무게 비교 결과)를 반환
```
- 람다의 기본 문법
    - (parameters) -> expression
    - (parameters) -> { statements; }

### 3.2 어디에, 어떻게 람다를 사용할까?

#### 3.2.1 함수형 인터페이스

- Predicate<T> 오직 하나의 추상 메서드만 지정

```java
public interface Predicate<T> {
    boolean test (T t);
}
```

- 자바 API의 함수형 인터페이스로는 Comparator, Runnable 등이 있음

```java
public interface Comparator<T>{     <- java.util.Comparator
    int compare(T o1, T o2);
}

public interface Runnable{      <- java.util.Runnable
    void run();
}

public interface ActionListener extends EventListener {     <- java.awt.event.ActionListener
    void actionPerforemd(ActionListener e);
}

public interface Callable<V>{       <- java.util.concurrent.Callable
    V call();
}

public interface PrivilegedAction<V>{       <- java.security.PrivilegedAction
    V run();
}
```

- 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달 할수 있으므로 <b>전체 표현식을 함수형 인터페이스의 인스턴스로 취급</b>  
함수형 인터페이스 보다는 덜 깔끔하지만 익명 내부 클래스로도 같은 기능을 구현 할수 있음

```java
Runnnable r1 = () -> System.out.prientln("Hello World 1");      <- 람다 사용

Runnnable r2 = new Runnnable(){     <- 익명 클래스 사용
    public void run(){
        System.out.println("Hello World 2"); 
    }
}

public static void process(Runnable r) {
    r.run();
}

process(r1);    <- "Hello World 1" 출력
process(r2);    <- "Hello World 2" 출력
process(() -> System.out.prientln("Hello World 3"));    <- 직접 전달된 람다 표현식으로 "Hello World 3" 출력
```

#### 3.2.2 함수 디스크립터
- 람다 표현식의 시그너처를 서술하는 메서드를 <b>함수 디스크림터<sup>function descriptor</sup></b> 라고 함
- Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로(void 반환) Runnable 인터페이스는인수와 반환값이 없는 시그너처로 생각 할수 있음

- 특수 표기법으로 람다와 함수형 인터페이스 시그너처를 설명
    - () -> void 라는 표기는 파라미터 리스트가 없으며 void를 반환하는 함수를 의미
    
`process 메서드에 직접 람다 표현식을 전달`
```java
public void process(Runnable r){
    r.run();
}

process(() -> System.out.println("This is awesome!!"));
```
- 위 코드를 실행하면 "This is awesome!!"이라고 출력 된다. () -> System.out.println("This is awesome!!"))은 인수가 없으며 void를 반환하는 람다 표현식  
Runnable 인터페이스의 run 메서드 시그너처와 같음

### 3.3 람다 활용 : 실행 어라운드 패턴
```java
public static String processFile()throws IOException{
    try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
        return br.readLine();       <- 실제 필요한 작업을 하는 
    }
}
```
#### 3.3.1 1단계 : 동작 파라미터화를 기억하라
- 한번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하려면 기존의 설정, 정리과정은 재사용하고 processFile 메서드만 다른 동작을 수행하도록 명령하려면 
processFile의 동작을 파라미터화하는것 람다를 이용해 동작을 전달.
- processFile 메서드가 한 번에 두행을 읽도록 코드를 수정, BufferedReader를 인수로 받아서 String을 반환하는 람다가 필요
```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

#### 3.3.2 2단계 : 함수형 인터페이스를 이용해서 동작 전달
- 함수형 인터페이스 자리에 람다를 사용 할수 있음.  
BufferedReader -> String과 IOException을 던질<sup>throw</sup>수 있는 시그너처와 일치하는 함수형 인터페이스를 만듬

```java
@FunctionalInterface
public interface BufferedReaderProcessor{
    String process(BufferedReader b) throws IOException;
}

정의한 인터페이스를 processFile 메서드의 인수로 전달

public static String processFile(BufferedReaderProcessor p) throws IOException{
    ...
};
```

#### 3.3.3 3단계 : 동작 실행!
- 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 **함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리**  
processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출 

```java
public static String processFile(BufferedReaderProcessor p) throws IOException{
    try(BufferedReader br = 
                        new BufferedReader(new FileReader("data.txt"))){
        return p.process(br);   <- BufferedReader 객체 처리
    }   
}
```

#### 3.3.4 4단계 : 람다 전달
`한 행을 처리`
```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
```
`두 행을 처리`
```java
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 3.4 함수형 인터페이스 사용
- 함수형 인터페이스의 추상 메서드 시그너처를 **함수 디스크립터**<sup>function descriptor</sup>라고 함  
다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요  

#### 3.4.1 Predicate
- java.util.function.Predicate<T> 인터페이스는 test라는 추상 메서드를 정의 test는 제네릭 형식 T의 객체를 인수로 받아 불린을 반환
- T 형식의 객체를 사용하는 불린 표현식이 필요한 상황에서 Predicate 인터페이스를 사용할 수 있음

```java
@FunctionalInterface
public interface Predicate<T>{
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> results = new ArrayList<>();
    for(T s : list){
        if(p.test(s)){
            results.add(s);
        }
    }
    return results;
}

predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

#### 3.4.2 Consumer
- java.util.function.Consumer<T> 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의
- Integer 리스트를 인수로 받아서 각 항목에 어떤 동작을 수행 하는 forEach 메서드를 정의 할때 Consumer를 활용할 수 있음

```java
@FunctionalInterface
public interface Consumer<T>{
    void accept(T t);
}

public static <T> void forEach(List<T> list, Consumer<T> c) {
    for(T i : list){
        c.accept(i);
    }
}
forEach(
    Arrays.asList(1, 2, 3, 4, 5), (Integer i) -> System.out.println(i)  <- Consumer의 accept 메서드를 구현하는 람다
    );
```

#### 3.4.3 Function
- java.util.function.Function<T, R> 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 apply라는 추상메서드를 정의  
입력을 출력으로 매핑하는 람다를 정의 할때 Function 인터페이스를 활용

```java
@FunctionalInterface
public interface Function<T, R>{
    R apply(T t);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f){
    List<R> result = new ArrayList<>();
    for(T s : list){
        result.add(f.apply(s));
    }
    return result;
}
//[7,2,6]
List<Integer> l = map(
        Arrays.asList("lambdas", "in", "action"), 
        (String s) -> s.length()    <- Function의 apply 메서드를 구현하는 람다
);
```






