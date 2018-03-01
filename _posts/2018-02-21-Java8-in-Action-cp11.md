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

- 자바8에서 새로 제공하는 CompletableFuture 클래스(Future 인터페이스를 구현한 클래)
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

- 자바5 부터 비동기 계산의 결과를 표현할 수 있는 java.util.concurrent.Future 인터페이스를 제공(즉, 호출자 스레드가 블록되지 않고 다른 작업을 실행)

```java
public Future<Double> getPrice(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();      <-- 계산 결과를 포함할 CompletableFuture를 생성
    new Thread( () -> {
                try {
                    double price = calculatePrice(product);     <-- 다른 스레드에서 비동기적으로 계산을 수행
                    futurePrice.complete(price);        <-- 오랜 시간이 걸리는 계산이 완료되면 Future에 값을 설정
                } catch (Exception ex) {
                    futurePrice.completeExceptionally(ex);
                }
    }).start();
    return futurePrice;     <-- 계산 결과가 완료되길 기다리지 않고 Future를 반환
}
```

- Future가 값을 가지고 있다면 Future에 포함된 값을 ㅇ릭거나 값이 계산될 때까지 블록

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product");     <-- 상점에 제품가격 정보 요청
long invocationTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Invocation returned after " + invocationTime 
                                                + " msecs");
// 제품의 가격을 계산하는 동안
// 다른 상점 검색 등 다른 작업 수행
doSomethingElse();
try {
    double price = futurePrice.get();                   <-- 가격정보가 있으면 Future에서 가격 정보를 읽고,
    System.out.printf("Price is %.2f%n", price);            가격정보가 없으면 가격 정보를 받을 때까지 블록
} catch (ExecutionException | InterruptedException e) {
    throw new RuntimeException(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Price returned after " + retrievalTime + " msecs");
```
    
> 에러 처리 방법

- 예외가 발생하면 해당 스레드에만 영향을 줌 즉, 에러가 발생해도 가격 계산을 계속 진행되며 일의 순서가 꼬임
- 클라이언트는 get 메서드가 반환될 때까지 영원히 기다리게 될 수도 있음
- 블록문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋음
- 에러 발생원인을 알수 있는 방법으로는 completeExceptionally 메서드를 이요해서 CompletableFuture 내부 발생한 예외를 클라이언트로 전달 해야함

```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread( () -> {
                try {
                    double price = calculatePrice(product);
                    futurePrice.complete(price);    <-- 계산이 정상적으로 종료되면 Future에 가격 정보를 저장한 채로 Future를 종료
                } catch (Exception ex) {
                    futurePrice.completeExceptionally(ex); <-- 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종
                }
    }).start();
    return futurePrice;
}
```

> 팩토리 메서드 supplyAsync로 CompletableFuture 만들기

- 팩토리 메서드 supplyAsync를 사용하여 간단하게 한행으로 구현 가능

```java
public Future<Double> getPrice(String product) {
    return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

> 병렬 스트림으로 요청 병렬화 

- 병렬 스트림을 이용해서 순차 계산을 병렬로 처리해서 성능을 개선

```java
public List<String> findPrices(String product) {
    return shops.parallelStream()   <-- 병렬 스트림으로 데이터를 병렬로 요청
            .map(shop -> String.format("%s price is %.2f", shop.getName(). shop.getPrice(product)))
            .collect(toList());
}
``` 

> CompletableFuture로 비동기 호충 구현

```java
public List<String> findPricesFuture(String product) {
    List<CompletableFuture<String>> priceFutures =
            shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(     <-- CompletableFuture로 각각의 가격을 비동기적으로 계
                    () -> shop.getName() + " price is "
                    + shop.getPrice(product), executor))
            .collect(Collectors.toList());

    List<String> prices = priceFutures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
    
    return priceFutures.stream()
            .map(CompletableFuture::join)   <-- 모든 비동기 동작이 끝나길 기다림
            .collect(Collectors.toList());
}
```

