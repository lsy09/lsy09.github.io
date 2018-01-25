---
layout: post
title:  "Java8 in Action : CHAPTER6"
date:   2017-12-20
desc: "CHAPTER 6. 스트림으로 데이터 수집"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java
---


## CHAPTER 6. 스트림으로 데이터 수집

```text
- Collectors 클래스로 컬렉션을 만들고 사용
- 하나의 값으로 데이터 스트림 리듀스
- 특별한 리듀싱 요약 연산
- 데이터 그룹화와 분할
- 자신만의 커스텀 컬렉터 개발
```

-	자바8의 스트림이란 데이터 집합을 멋지게 처리하는 게으른 반복자
-	스트림의 연산은 filter 또는 map 같은 중간 연산과 count, findFirst, forEach, reduce등의 최종 연산으로 구분
-	중간 연산은 한 스트림을 다른 스트림으로 변환하는 연산으로, 여러 연산을 구분 할수 있음
	-	스트림 파이프라인을 구성, 스트림의 요소를 소비<sup>consume</sup> 하지 않음
-	최종연산은 스트림의 요소를 소비해서 최종 결과 도출(예를 들어 스트림의 가장 큰 값 반환)

	-	스트림 파이프라인을 최적화 하면서 계산 과정을 짧게 생략하기도 함

	`예제 : collect 와 컬렉터로 구현 할수 있는 예제`

	```java
	Map<Currency, List<Transaction>> transactionByCurrencies =
	                                                  new HashMap<>();    <-- 그룹화한 트랜잭션을 저장할 맵을 생성


	for(Transaction transaction : transactions){      <-- 트랜잭션 리스트를 반복
	  Currency currency = transaction.getCurrency();
	  List<Transaction> transactionForCurrency =
	                                        transactionByCurrencies.get(currency);
	  if(transactionForCurrency == null){   <-- 현재 통화를 그룹화하는 맵에 항목이 없으면 항목을 만듬
	    transactionForCurrency = new ArrayList<>();
	    transactionByCurrencies.put(currency, transactionForCurrency);
	  }
	  transactionForCurrency.add(transaction);  <-- 같은 통화를 가진 트랜잭션 리스트애 현재 탐색 중인 트랜잭션을 추가
	}
	```

	`Stream에 toList를 사용하는 대신 더 범용적인 컬렉터 파라미터를 collect 메서드에 전달 함으로써 원하는 연산을 간결하게 구현`

	```java
	Map<Currency, List<Transaction>> transactionByCurrencies =
	  transactions.stream().collect(groupingBy(Transaction::getCurrency));
	```

### 6.1 컬렉터란 무엇인가?

-	함수형 프로그래밍에서는 필요한 컬렉터를 쉽게 추가 가능

> 고급 리듀싱 기능을 수행하는 컬렉터

-	스트림에 collect를 호출하면 스트림의 요소에(컬렉터로 파라미터화된) 리듀싱 연산이 수행
-	collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리
-	가장 많이 사용하는 직곽적인 정적 메서드로 toList를 꼽을 수 있고, toList는 스트림의 모든 요소를 리스트로 수집

	```java
	List<Transaction> transactions =
	                        transactionStrean.collect(Collectors.toList());
	```

> 미리 정의된 컬렉터

-	Collectors에서 제공하는 메서드의 기능은 크게 세가지로 구분
	-	스트림 요소를 하나의 값으로 리듀스하고 요약
	-	요소 그룹화
	-	요소분할

### 6.2 리듀싱과 요약

-	컬렉터(Stream.Collect 메서드의 인수)로 스트림의 항목을 컬렉션으로 재구성

	`counting()이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산`

	```java
	long howManyDishes = menu.stream().collect(Collectors.counting());
	```

	`불필요 한 과정 생략`

	```java
	long howManyDishes = menu.stream().count();
	```

-	counting 컬렉터는 다른 컬렉터와 함께 사용할때 위력을 발휘

> 스트림값에서 최댓값과 최솟값 검색

-	Collectors.maxBy, Collectors.minBy 두 개의 메서드를 이용해서 스트림의 최댓값과 최솟값을 계산
-	두 컬렉터는 스트림의 요소를 비교하는 데 사용할 Comparator를 인수로 받는다

	```java
	Comparator<Dish> dishCaloriesComparator =
	        Comparator.comparingInt(Dish::getCalories);


	Optional<Dish> mostCalorieDish =
	        menu.stream()
	                .collect(maxBy(dishCaloriesComparator));
	```

-	스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용, 이러한 연산을 요약 <sup>summarization</sup> 연산이라고 부름

> 요약 연산

-	Collectors 클래스는 Collectors.summingInt라는 특별한 요약 팩토리 메서드를 제공

	-	summingInt는 객체를 int로 매핑하는 함수를 인수로 받음
	-	summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환
	-	summingInt가 collect 메서드로 전달되면 요약 작업 수행

	```java
	int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
	```

-	Collectors.averagingInt, averagingLong, averagingDouble 등으로 다양한 형식으로 이루어진 숫자 집합의 평균을 계산

	```java
	double avgCalories =
	                menu.stream().collect(averagingInt(Dish::getCalories));
	```

-	int뿐 아니라 long이나 double에 대응하는 summarizingLong, summarizingDouble 메서드와 관련된 LongSummaryStatistics, DoubleSummaryStatistics 클래스도 있음

> 문자열 연결

-	컬렉터에 joining 팩토리 메서드를 이용, 스트림의 각 객체에 toString메서드를 호출 추출한 모든 문자열을 하나의 문자열로 연결해서 반환

	```java
	String shortMenu = menu.stream().map(Dish::getCalories).collect(joining());
	```

-	joining 메서드는 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만듬

-	toString 메서드를 포함 하고 있다면, map으로 문자를 추출하는 과정을 생략 가능

	```java
	String shortMenu = menu.stream().collect(joining());
	```

-	구분 문자열을 넣을수 있도록 오버로드된 joining 팩토리 메서드도 있음

	```java
	String shortMenu = menu.stream().map(Dish::getName).collect(joining(","));
	```

> 범용 리듀싱 요약 연산

-	reducing 팩토리 메서드로도 정의 할 수 있음
-	범용 Collectors.reducing으로도 구현 할수 있음

	```java
	int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
	```

-	reducing은 세 개의 인수를 받음

	1.	리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값(숫자 합계에서는 인수가 없을 때 반환값으로 0이 적합)
	2.	인수는 `요약 연산`에서 정수로 변환할 때 사용한 변환 함수
	3.	같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator

		```java
		Optional<Dish> mostCalorieDish =
		            menu.stream().collect(reducing(
		                (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
		```

### 6.3 그룹화

-	자바8의 함수형을 이용하면 가독성있는 한 줄의 코드로 그룹화를 구현 가능
-	팩토리 메서드 Collectors.groupingBy 이용하여 쉽게 그룹화 가능

	```java
	Map<Dish.Type, List<Dish>> dishesByType =
	                menu.stream().collect(groupingBy(Dish::getType));
	```

-	groupingBy함수를 기준으로 스트림이 그룹화 되므로 이를 <b>분류 함수</b><sup>classification function</sup>라고 함

> 다수준 그룹화

-	Collectors.groupingBy는 일반적인 분류 함수와 컬렉터를 인수로 받음

	```java
	Map<Dish.Type, Map<CaloricLevel, List<Dish>>> groupDishedByTypeAndCaloricLevel() {
	    return menu.stream().collect(
	            groupingBy(Dish::getType,           <-- 첫 번째 수준의 분류 함수
	                    groupingBy((Dish dish) -> {     <-- 두 번째 수준의 분류 함수
	                        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
	                        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
	                        else return CaloricLevel.FAT;
	                    } )
	            )
	    );
	}
	```

-	보통 groupingBy의 연산을 '버킷<sup>bucket</sup>(물건을 담을 수 있는 양동이)' 개념

-	예제의 첫번째 groupingBy는 각 키의 버킷을 만든다. 그리고 준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복, n수준 그룹화를 달성

> 서브그룹으로 데이터 수집

-	다수준 그룹화 예제에서 첫번째 groupingBy로 넘겨주는 컬렉터의 형식은 제한이 없음
-	팩토리 메서드 Collectors.collectingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용

	```java
	Map<Dish.Type, Dish> mostCaloricByType =
	        menu.stream()
	                .collect(groupingBy(Dish::getType,      <-- 분류 함수
	                                collectingAndThen(
	                                        maxBy(comparingInt(Dish::getCalories)),     <-- 감싸인 컬렉터
	                                Optional::get)));       <-- 변환 함수
	```

-	리듀싱 컬렉터는 절대 Optional.empty()를 반환하지 않으므로 안전한 코드

> groupingBy와 함께 사용하는 다른 컬렉터 예제
>
> ```java
> Map<Dish.Type, Set<CaloricLevel>> caloricLevelByType =
> menu.stream().collect(
>       groupingBy(Dish::getType, mapping(dish -> {
>               if(dish.getCalories() <= 400) {
>                   return CaloricLevel.DIET;
>           }else if (dish.getCalories() <= 700){
>               return CaloricLevel.NORMAL;
>           }else {
>               return CaloricLevel.FAT;
>           }
> },
> toSet())));
> ```

### 6.4 분할

-	분할은 <b>분할 함수</b><sup>partitioning function</sup>라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능
-	불린을 반환하므로 맵의 키 형식은 `Boolean`
-	그룹화 맵은 최대(참 아니면 거짓의 값을 갖는) 두 개의 그룹으로 분류

	```java
	Map<Boolean, List<Dish>> partitionedMenu =
	        menu.stream().collect(partitioningBy(Dish::isVegetarian));      <--  분할 함수
	```

	`분리된 참값으로 데이터 추출`

	```java
	List<Dish> vegetarianDishes = partitionedMenu.get(true);
	```

> 분할의 장점

-	참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점
-	컬렉터를 두 번째 인수로 전달할 수 있는 오버로드된 버전의 partitioningBy 메서드도 있음

	```java
	Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType =
	    menu.stream().collect(partitioningBy(Dish::isVegetarian,        <-- 분할 함수
	                  groupingBy(Dish::getType)));        <-- 두 번째 컬렉터
	```

	```java
	Map<Boolean, Dish> mostCaloricPartitionedByVegetarian =
	menu.stream().collect(
	      partitioningBy(Dish::isVegetarian,
	              collectingAndThen(
	                      maxBy(comparingInt(Dish::getCalories)),
	                      Optional::get)));
	```

### 6.5 Collector 인터페이스

-	Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성
-	Collector 인터페이스의 시그니처와 다섯 개의 메서드 정의

	```java
	public interface Collector<T, A, R>{
	    Supplier<A> supplier();
	    BiConsumer<A, T> accumulator();
	    Function<A, R> finisher();
	    BinaryOperator<A> combiner();
	    Set<Characteristics> characteristics();
	}
	```

	-	T는 수집될 스트림 항목의 제네릭 형식
	-	A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
	-	R은 수집 연산 결과 객체의 형식(대게 컬렉션 형식)

		```java
		public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
		```

> supplier 메서드 : 새로운 결과 컨테이너 만들기

-	supplier 메서드는 빈 결과로 이루어진 Supplier를 반환. 즉, supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수
-	ToListCollector 처럼 누적자를 반환하는 컬렉터에서는 빈 누적자가 비어있는 스트림의 수집 과정의 결과가 될수 없음

	```java
	public Supplier<List<T>> supplier() {
	    return () -> new ArrayList<T>();
	}
	```

	`생성자 레퍼런스를 전달 방법`

	```java
	public Supplier<List<T>> supplier() {
	    return ArrayList::new;
	}
	```

> accumulator 메서드 : 결과 컨테이너에 요소 추가하기

-	accumulator 메서드는 리듀신 연산을 수행하는 함수를 반환
-	ToListCollector에서 accumulator가 반환하는 함수는 이미 탐색한 항목을 포함하는 리스트에 현재 항목을 추가하는 연산 수행

	```java
	public BiConsumer<List<T>, T> accumulator(){
	    return List::add;
	}
	```

> finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기

-	finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수 반환
-	누적자 객체가 이미 최종 결과일 때는 변환 과정이 필요 없으므로 finisher 메서드는 항등 함수 반환

	```java
	public Function<List<T>, List<T>> finisher() {
	        return Function.identity();
	    }
	```

> combiner 메서드 : 두 결과 컨테이너 병합

-	리듀싱 연산에서 사용할 함수를 반환
-	combiner는 스트림의 서로 다른 서브파트를 병렬로 처리 할때 누적자가 이 결과를 어떻게 처리 할지 정의

	```java
	public BinaryOperator<List<T>> combiner() {
	    return (list1, list2) -> {
	        list1.addAll(list2);
	        return list1;
	    };
	}
	```

> Characteristics 메서드

-	컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
-	Characteristics는 스트림을 병렬로 리듀스 할 것인지 그리고 병렬로 리듀스 한다면 어떤 최적화를 선택해야 할지 힌트 제공

	> UNORDERED

	```text
	리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않음
	```

	> CONCURRENT

	```text
	다중 스레드에서 accumulator 함수를 동시에 호출 가능, 스트림의 병렬 리듀싱 수행 할 수 있음
	데이터 소스가 정렬되어 있지 않은 상황에서만 별렬 리듀싱 수행 할 수 있음
	```

	> IDENTIT_FINISH

	```text
	리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용 가능
	누적자 A를 결과 R로 안전하게 형변환 할 수 있음
	```

> 컬렉터 구현을 만들지 않고도 커스텀 수집 수행하기

-	IDENTITY_FINISH 수집 연산에서는 Collector 인터페이스를 완전히 해로 구현 하지 않고도 같은 결과를 얻을 수 있음

	```java
	List<Dish> dishes = menuStream.collect(
	                  ArrayList::new,     <-- supplier
	                  List::add,          <-- accumulator
	                  List::addAll);      <-- combiner
	);
	```
	
### 6.6 커스텀 컬렉터를 구현해서 성능 개선하기

> 소수로만 나누기

1.	Collector 클래스 시그니처 정의
2.	리듀싱 연산 구현
3.	병렬 실행할 수 있는 컬렉터 만들기
4.	finisher 메서드와 컬렉터의 characteristics 메서드

> 컬렉터 성능 비교



