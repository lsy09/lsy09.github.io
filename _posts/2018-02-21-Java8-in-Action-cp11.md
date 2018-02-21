---
layout: post
title:  "Java8 in Action : CHAPTER_11"
date:   2018-02-21
desc: "11. CompletableFuture: 조합할 수 있는 비동기 프로그래밍"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---

## CHAPTER 11. CompletableFuture: 조합할 수 있는 비동기 프로그래밍

> Future

- 자바5 부터 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공
- 비동기 계산을 모델링하는 데 Future를 이용할 수 있으며, Future는 계산이 끝났을 때 결과에 접근할 수 있는 레퍼런스를 제공
- 저수준의 스레드에 비해 직관적으로 이해하기 쉽다는 장점 있음, Future를 이용하려면 시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼뒤 
ExecutorService에 제출 해야 함

`자바8 이전`

```java
ExecutorService executor = Executors.newCachedThreadPool();     <-- 스레드 풀에 태스크를 제출하려면 ExecutorService를 만들어야 함
Future<Double> future = executor.submit(new Callable<Double>(){     <-- Callable을 ExecutorService로 제출
   public Double call(){
       return doSomeLongComputation();      <-- 시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행
   } 
});
doSomethingElse();      <-- 비동기 작업을 수행하는 동안 다른작업을 수행
try{
    Double result = future.get(1, TimeUnit.SECONDS);    <-- 비동기 작업의 결과를 가져옴, 결과가 준비되어 있지 않으면 호츨 스레드가 블록, 
} catch (ExecutionException ee){                            하지만 최대 1초까지만 기다림
    // 계산 중 예외 발생
} catch (InterruptedException ie){
    // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te){
    // Future가 완료되기 전에 타임아웃 발생
}
```

> Future 제한

- Stream과 CompletableFuture는 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용
- Future와 CompletableFuture의 관계를 Collection과 Stream의 관계에 비유할 수 있음

> 동기 API와 비동기 API

- 동기 API
    - 메서드를 호출한 다음에 메서드가 계산을 완료할 때 까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행
    - 동기 API를 사용하는 상황을 **블록호출**<sup>blocking call</sup>이라고 함
    
- 비동기 API
    - 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당
    - 비동기 API를 사용하는 상황을 **비블록 호출**<sup>non-blocking call</sup>이라고 함
    - 주로 I/O 시스템 프로그래밍에서 이와 같은 방식으로 동작을 수행
    - 계산 동작을 수행하는 동안 비동기적으로 디스크 접근을 수행, 더 이상 수행할 동작이 없으면 디스크 블록이 메모리로 로딩될 때 까지 기다림
    
> 동기 메서드를 비동기 메서드로 변환

    