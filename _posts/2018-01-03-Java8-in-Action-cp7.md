---
layout: post
title:  "Java8 in Action : CHAPTER_07"
date:   2018-01-03
desc: "CHAPTER 7. 병렬 데이터 처리와 성능"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---

## CHAPTER 7. 병렬 데이터 처리와 성능
> [CHAPTER7.sampleTestCode](https://github.com/lsy09/Review/blob/master/Java8InAction/source/src/main/java/java8inaction/chap7/Parallel.java)

-	자바7 등장 이전 데이터 컬렉션을 병렬 처리하기 어려움이 있었음
-	스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 설명
-	스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀수 있음
-	여러 청크를 병렬로 처리하기 전에 병렬 스트림이 요소를 여러 청크로 분할하는 방법을 설명
-	커스텀 Spliterator를 직접 구현, 분할 과정을 우리가 원하는 방식으로 제어하는 방법 설명

> 병렬 스트림

-	컬렉션에 parallelStream 을 호출하면 **병렬 스트림**<sup>parallel stream</sup>이 생성됨
-	병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이며, 따라서 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있음

```java
public static long sequentialSum(long n){   
        return Stream.iterate(1L, i -> i+1)     <-- 무한 자연수 스트림 생성
                     .limit(n)                  <-- n개 이하로 제한    
                     .reduce(0L, Long::sum);    <-- 모든 숫자를 더하는 스트림 리듀싱 연산
    }
```

`전통적 자바에서 반복문 구현`

```java
public static long iterativeSum(long n) {
    long result = 0;
    for (long i = 0; i <= n; i++) {
        result += i;
    }
    return result;
}
```

> 순차 스트림을 병렬 스트림으로 변환하기

-	순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산(숫자 합계 계산)이 병렬로 처리

```java
public static long parallelSum(long n){
    return Stream.iterate(1L, i -> i+1)
                 .limit(n)
                 .parallel()
                 .reduce(0L, Long::sum);
}
```

-	순차 스트림에 parallel을 호출해도 스트림 자제에는 아무 변화도 일어나지 않음
-	내부적으로는 parallel을 호출하면 이후 연산이 병렬로 수행해야 함을 의미하는 불린 플래그가 설정
-	반대로 sequential로 병렬 스트림을 순차 스트림으로 바꿀수 있음

```java
Stream.parallel()
      .filter(...)
      .sequential()
      .map(...)
      .parallel()
      .reduce();
```

-	parallel 과 sequential 두 메서드 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미침

> 스트림 성능 측정

-	n개의 숫자를 더하는 함수의 성능 측정

```java
public static <T, R> long measurePerf(Function<T, R> f, T input) {
    long fastest = Long.MAX_VALUE;
    for (int i = 0; i < 10; i++) {
        long start = System.nanoTime();
        R result = f.apply(input);
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Result: " + result);
        if (duration < fastest) fastest = duration;
    }
    return fastest;
}
```

> > -	iterate가 박싱된 객체를 생성하므로 이를 다시 언박싱하는 과정이 필요
> > -	iterate는 병렬로 실행될 수 있도록 독립적인 청크로 분할하기가 어려움

-	마법 같은 parallel 메서드를 호출했을 때 내부적으로 어떤 일이 일어나는지 꼭 이해 해야함

> 더 특화된 메서드 사용

-	LongStream.rangeClosed라는 메서드는 iterate에 비해 다음과 같은 장점을 제공
	-	LongStream.rangeClosed는 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라짐
	-	LongStream.rangeClosed는 쉽게 청크로 분할할 수 있는 숫자 범위를 생산 함 ex: 1-20 범위의 숫자를 각각 1-5, 6-10, 11-15, 16-20 범위의 숫자로 분할 할 수 있음

`언박싱과 관련한 오버헤드가 얼마나될까? 순차 스트림을 처리하는 시간을 측정`

```java
public static long rangedSum(long n){
    return LongStream.rangeClosed(1, n)
                     .reduce(0L, Long::sum);
}
```

-	기존의 iterate 팩토리 메서드로 생성한 순차 버전에 비해 위 코드의 숫자 스트림 처리 속도가 더 빠름
-	상황에 따라서 어떤 알고리즘을 병렬화 하는 것보다 적절한 자료구조를 선택하는 것이 더 중요하다는 사실을 단적으로 보여줌

`새로운 버전에 병렬 스트림을 적용`

```java
public static long parallelRangedSum(long n){
    return LongStream.rangeClosed(1, n)
                     .parallel()
                     .reduce(0L, Long::sum);
}
```

-	병렬화를 이용하려면 스트림을 재귀적으로 분할해야 하고 각 서브스트림을 서로 다른 스레드의 리듀싱 연산을 할당, 이들 결과를 하나의 값으로 합쳐야 함

> 병렬 스트림의 올바른 사용법

-	병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어남

`n까지의 자연수를 더하면서 공유된 누적자를 바꾸는 프로그램을 구현`

```java
public static long sideEffectSum(long n){
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public static class Accumulator {
    public long total = 0;
    public void add(long value){ total += value; }
}
```

-	위 코드는 본질적으로 순차 실행할 수 있도록 구현되어 있으므로 병렬로 실행하면 참사가 일어남
-	total을 접근할 때마다(다수의 스레드에서 동시에 데이터에 접근하는) 데이터 레이스 문제가 일어남
-	동기화로 문제를 해결하다보면 결국 병렬화라는 특성이 사라짐

`스트림을 병렬로 만들어서 어떤 문제가 일어나는지 확인`

```java
public static long sideEffectParallelSum(long n){
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;

}
```

-	병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다는 사실을 확인

> 병렬 스트림 효과적으로 사용하기

-	어떤 상황에서 병렬 스트림을 사용할 것인지 약간의 수량적 힌트를 정하는것이 도움이 될때가 있음

        -	확신이 서지 않는다면 직접 측정하라. 순차 스트림을 병렬 스트림으로 쉽게 바꿀수 있다. 하지만 무조건 병렬 스트림으로 바꾸는 것이 능사는 아니다.
        -	박싱을 주의하라. 자동 박싱과 언박싱은 성능을 크게 저하시킬수 있는 요소다 자바8은 박싱동작을 피할 수 있도록 기본형 특화 스트림 (IntStream, LongStream, DoubleStream)을 제공한다.
        -	순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다. 특히 limit나 findFirst처럼 요소의 순서에 의존하는 연산은 예를들어 findAny는 요소의 순서와 상관없이 연산하므로 findFirst보다 성능이 좋다.정렬된 스트림에 unordered를 호출하면 비정렬 된 스트림을 얻을수 있다.
        -	스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.
        -	소량의 데이터에서는 병령 스트림이 도움이 되지 않는다.
        -	스트림을 구성하는 자료구조가 적절한지 확인하라.
        -	스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
        -	최종 연산의 병합과정(예를들면 Collector의 combiner 메서드) 비용을 살펴보라.

> 스트림 소스와 분해성

| 소스            | 분해성 |
|-----------------|--------|
| ArrayList       | 훌륭함 |
| LinkedList      | 나쁨   |
| IntStream.range | 훌륭함 |
| Stream.iterate  | 나쁨   |
| HashSet         | 좋음   |
| TreeSet         | 좋음   |

> 포크/조인 프레임워크

-	병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계
-	서브태스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현

> Recursive Task 활용

-	스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 함
-	여기서 R은 병렬화된 태스크가 생성하는결과 형식 또는 결과가 없을 때 RecursiveAction 형식
-	RecursiveTask를 정의 하려면 추상 메서드 compute를 구현해야 함

```java
protected abstract R compute();
```

-	compute 메서드는 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의
-	대부분의 compute 메서드 구현은 다음과 같은 의사코드 형식을 유지

```java
if(태스크가 충분히 작거나 더 이상 분할할 수 없으면){
    순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브태스크의 결과를 합침
}
```

-	위의 알고리즘은 분할 후 정복<sup>divide-and-conquer</sup> 알고리즘의 병렬화 버전
-	일반적으로 애플리케이션에서는 둘 이상의 ForkJoinPool을 사용하지 않는다. 즉, 소프트웨어의 필요한 곳에서 언제든 가져다 쓸 수 있도록 ForkJoinPool을 한번만 인스턴스화해서 정적 필드에 싱글턴으로 저장
-	ForkJoinPool을 만들면서 인수가 없는 디폴트 생성자를 이용했는데, 이는 JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있을을 의미
-	Runtime.availableProcessors의 반환값으로 풀에 사용할 스레드 수를 결정
-	availableProcessors, 즉 '사용할 수 있는 프로세서'라는 이름과는 달리 실제 프로세서 외에 하이퍼스레딩과 관련된 가상프로세서도 개수에 포함됨

> ForkJoinSumCalculator 실행

-	ForkJoinSumCalculator를 ForkJoinPool로 전달하면 풀의 스레드가 ForkJoinSumCalculator의 compute 메서드를 실행하면서 작업을 수행
-	태스크의 크기가 크다고 판단되면 숫자 배열을 반으로 분할해서 두개의 새로운 ForkJoinSumCalculator로 할당되며, 다시 ForkPool이 새로 생성된 ForkJoinSumCalculator 를 실행
-   위의 과정이 재귀적으로 반복되면서 주어진 조건(예제에서는 덧셈을 수행할 항목이 만 개 이하여야 함)을 만족할때까지 태스크 분할을 반복
- 각 서브태스크는 순차적으로 처리되며 포킹 프로세스로 만들어진 이진트리의 태스크를 루트에서 역순으로 방문
- 각 서브태스크의 부분 결과를 합쳐서 태스크의 최종 결과를 계산

> Spliterator

- 자바8에서는 Spliterator라는 새로운 인터페이스를 제공
- Spliterator는 '분할할 수 있는 반복자<sup>splitable iterator</sup>'라는 의미 
- Iterator처럼 Spliterator는 소느의 요소 탐색 기능을 제공한다는 점은 같지만 Spliterator는 병렬 작업에 특화
- 자바8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliterator 구현을 제공

```java
public interface Spliterator<T>{
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

> Spliterator 특성

- Spliterator는 characteristics라는 추상 메서드도 정의함
- characteristics 메서드는 Spliterator 자체의 특성 집합을 포함하는 int를 반환
- Spliterator를 이용하는 프로그램은 이들 특성을 참고해서 Spliterator를 더 잘 제어하고 최적화 할수 있음

|특성     | 의미
|--------|---------
|ORDERED    | 리스트처럼 요소에 정해진 순서가 있으므로 Spliterator는 요소를 탐색하고 분할할 때 이 순서에 유의 해야 함
|DISTINCT   | x,y 두 요소를 방문했을 때 x.equals(y)는 항상 false를 반환
|SORTED     | 탐색된 요소는 미리 정의된 정렬 순서를 따름
|SIZED      | 크기가 알려진 소스(예를 들면 Set)로 Spliterator를 생성했으므로 estimatedSize()는 정확한 값을 반환
|NONNULL    | 탐색하는 모든 요소는 null이 아님
|IMMUTABLE  | 이 Spliterator의 소스는 불변 즉, 요소를 담색하는 동안 요소를 추가하거나, 삭제하거나, 고칠 수 없음
|CONCURRENT | 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있음
|SUBSIZED   | 이 Spliterator 그리고 분할되는 모든 Spliterator는 SIZED 특성을 가짐


