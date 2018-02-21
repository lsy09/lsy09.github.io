---
layout: post
title:  "Java8 in Action : CHAPTER4"
date:   2017-12-11
desc: "CHAPTER 4. 스트림 소개"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---


## CHAPTER 4. 스트림 소개

-	컬렉션은 자바에서 가장 많이 사용 하는 기능 중 하나
-	거의 모든 자바 애플리케이션은 컬렉션은 **만들고 처리하는 과정**을 포함
-	대부분의 자바 애플리케이션에서는 컬렉션을 많이 사용하지만 완벽한 컬렉션 관련 연산을 지원 하려면 한참 멀었다

### 4.1 스트림이란 무엇인가?

-	스트림은 자바 API에 새로 추가된 기능
-	스트림을 이용하면 선언형(즉, 데이터를 처리하는 임시 구현 코드 대신 질의로 표현)으로 컬렉션 데이터를 처리
-	멀티스레드 코드를 구현하지 않아도 데이터를 **투명하게** 병렬로 처리

	-	예제) 저칼로리의 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬

	`자바 7`

	```java
	List<Dish> lowCaloricDishes = new ArrayList<>();
	for(Dish d: menu){
	    if(d.getCalories() < 400){          <--- 누적자로 요소 필터링
	        lowCaloricDishes.add(d);
	    }
	}


	Collection.sort(lowCaloricDishes, new Comparatior<Dish>(){      <--- 익명 클래스로 요리 정렬
	    public int compare(Dish d1, Dish d2){
	        return Integer.compare(d1.getCalories(), d2.getCalories());    
	    }
	}


	List<String> lowCaloricDishesName = new ArrayList<>();
	for(Dish d: lowCaloricDishes){
	    lowCaloricDishesName.add(d.getName());          <--- 정렬된 리스트를 처리 하면서 요리 이름 선택
	}


	```

	`자바 8`

	```java
	import static java.uitl.Comparator.comparing;
	import static java.uitl.stream.Collectors.toList;
	List<String> lowCaloricDishesName =
	                menu.stream()
	                    .filter(d -> d.getCalories() < 400)     <--- 400칼로리 이하의 요리선택
	                    .sorted(comparing(Dish::getCalories))   <--- 칼로리로 요리정렬
	                    .map(Dish::getName)                     <--- 요리명 추출
	                    .collect(toList());                     <--- 모든 요리명을 리스트에 저장
	```

	`stream()을 paralleStream()으로 바꾸면 이 코드를 멀티코어 아키텍처에서 병렬로 실행 가능`

	```java
	List<String> lowCaloricDishesName =
	                menu.parallelStream()
	                    .filter(d -> d.getCalories() <400)
	                    .sorted(comparing(Dish::getCalories))
	                    .map(Dish::getName)
	                    .collect(toList());
	```

	-	**선언형**으로 코드를 구현 가능. 즉, 루프와 if조건문 등의 제어 블록을 사용해서 **어떻게** 동작을 구현할지 지정할 필요 없음

	-	filter(또는 sorted, map, collect) 같은 연산은 고수준 빌딩 블록<sup>high-level building block</sup>으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있음

	-	자바 8의 스트림 API의 특징을 다음처럼 요약

		-	**선언형** : 더 간결하고 가독성이 좋아진다.
		-	**조립할 수 있음** : 유연성이 좋아진다.
		-	**병렬화** : 성은이 좋아진다.

	`다음 코드의 요리 리스트, 즉 메뉴(menu)를 주요 예제롤 사용`

	```java
	List<Dish> menu = Arrays.asList(
	    new Dish("pork", false, 800, Dish.Type.MEAT),
	    new Dish("beef", false, 700, Dish.Type.MEAT),
	    new Dish("chicken", false, 400, Dish.Type.MEAT),
	    new Dish("french fries", true, 530, Dish.Type.OTHER),
	    new Dish("rice", true, 350, Dish.Type.OTHER),
	    new Dish("season fruit", true, 120, Dish.Type.OTHER),
	    new Dish("pizza", true, 550, Dish.Type.OTHER),
	    new Dish("prawns", false, 300, Dish.Type.FISH),
	    new Dish("salmon", false, 450, Dish.Type.FISH)
	)
	```

	`Dish는 다음과 같은 불변형 클래스다.`

	```java
	public class Dish{
	    private final String name;
	    private final boolean vegetarian;
	    private final int calories;
	    private final Type type;


	    public Dish(String name, boolean vegetarian, int calories, Type type){
	        this.name = name;
	        this.vegetarian = vegetarian;
	        this.calories = calories;
	        this.type = type;
	    }


	    public String getName(){
	        return name;
	    }


	    public boolean isVegetarian(){
	        return vegetarian;
	    }


	    public int getCalories(){
	        return calories;
	    }


	    public Type getType(){
	        return type;
	    }


	    @Override
	    public String to String(){
	        return name;
	    }


	    public enum Type{
	        MEAT, FISH, OTHER
	    }
	}
	```

### 4.2 스트림 시작하기

-	스트림 중 가장 간단한 작업에 해당하는 켈렉션 스트림
-	자바 8의 컬렉션에서는 스트림을 반환하는 stream 이라는 메서드가 추가
-	숫자 법위나 I/O 자원에서 스트림요소를 만드는 등 stream 메서드 이외에도 다양한 방법으로 스트림을 얻을 수 있음
-	**스트림**이란? '데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소'

	> 연속된 요소

	-	특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공
	-	컬렉션은 자료구조이며, 스트림은 filter, sorted, map 처럼 표현 계산식이 주를 이룸

	> 소스

	-	컬렉션, 배열 I/O 자원등의 데이터 제공 소스로 부터 데이터를 소비 <sup>consume</sup>
	-	리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지

	> 데이터 처리 연산

	-	함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원
	-	filter, map, reduce, find, match, sort 등으로 데이터를 조작
	-	스트림 연산은 순차적으로 또는 병렬로 실행

	> 파이프라이닝

	-	스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인은 구성할 수 있도록 스트림 자신을 반환
	-	**게으름**<sup>laziness</sup>, **쇼트서킷**<sup>short-circuiting: 단선</sup> 같은 최적화도 얻을수 있음
	-	연산 파이프라인은 데이터 소스에 적용하는 데이터베이스 질의와 비슷

	> 내부 반복 - 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원

	```java
	import static java.util.stream.Collectors.toList;
	List<String> threeHighCaloricdishNames =
	        menu.stream()                           <--- 메뉴(요리 리스트)에서 스트림을 얻는다
	            .filter(d -> d.getCalories() > 300) <--- 파이프라인 연산 만들기, 첫 번째로 고칼로리 요리를 필터링한다
	            .map(Dish::getName)                 <--- 요리명 추출
	            .limit(3)                           <--- 선착순 세 개만 선택
	            .collect(toList());                 <--- 결과를 다른 리스트로 저장
	System.out.prientln(threeHighCaloricDishNames); <--- 결과는[pork, beef, chicken]이다
	```

	> filter

	-	람다를 인수로 받아 스트림에서 특정 요소 제외

	> map

	-	람다를 이용 한 요소를 다른요소로 변환하거나 정보를 추출

	> limit

	-	정해진 개수 이상의 요소가 ㅅ트림에 저장되지 못하게 스트리 크기를 축소<sup>truncate</sup>

	> collect

	-	스트림을 다른 형식으로 변환

-	스트림 API는 파이프라인을 더 최적화할 수 있는 유연성을 제공

### 4.3 스트림과 컬렉션

-	자바의 기존 컬렉션과 새로운 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공
-	연속된<sup>sequenced</sup>이라는 표현은 순서와 상관없이 아무 값에너 접속하는 것이 나이라 순차적으로 값에 접근한다는 것을 의미

	-	컬렉션과 스트림의 차이
		-	데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이  
			> 컬렉션
	-	현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조(컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함) > 스트림
	-	요청할 때만 요소를 계산하는 고정된 자료구조(스트림에 요소를 추가하거나 스트림에서 요소를 제거 할 수 없음)

> 딱 한 번만 탐색할 수 있다!

-	반복자와 마찬가지로 스트림도 한 번만 탐색. 즉, 탐색된 스트림의 요소는 소비<sup>consume</sup>
-	한 번 탐색한 요소를 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야 함
-	스트림은 단 한번만 소비할 수 있다는 점 명심!

> 외부 반복과 내부 반복

-	컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복, 이를 외부 반복<sup>external iteration</sup>
-	스트림 라이브러리는 내부 반복<sup>internal iteration</sup>을 사용

`컬렉션 : for-each 루프를 이용하는 외부 반복`

```java
List<String> names = new ArrayList<>();
for(Dish d: menu){              <--- 메뉴 리스트를 명시적으로 순차 반복
    names.add(d.getName());     <--- 이름을 추출해서 리스트에 추가
}
```

`컬렉션 : 내부적으로 숨겨졌던 반복자를 사용한 외부 반복`

```java
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()){      <--- 명시적 반복
    Dish d = iterator.next();
    names.add(d.getName());
}
```

`스트림 : 내부 반복`

```java
List<String> names = menu.stream()
                         .map(Dish::getName)    <--- map 메서드를 getName 메서드로 파라미터화 해서 요리명을 추출
                         .collect(toList());    <--- 파이프라인을 실행, 반복자는 필요 없음
```

### 4.4 스트림 연산

-	스트림 인터페이스의 연산을 크게 두가지로 구분

	```java
	List<String> threeHighCaloricdishNames =
	            menu.stream()                           <--- 스트림을 얻기
	                .filter(d -> d.getCalories() > 300) <--- 중간연산
	                .map(Dish::getName)                 <--- 중간연산
	                .limit(3)                           <--- 중간연산
	                .collect(toList());                 <--- 스트림을 리스트로 변환
	```

	-	filter, map, limit는 서로 연결되어 파이프라인 형성
	-	collect로 파이프라인을 실행한 다음에 닫음

-	연결할 수 있는 스트림연산을 **중간 연산**<sup>intermediate operation</sup>

-	스트림을 닫는 연산을 **최종 연산**<sup>terminal operation</sup>

> 중간 연산

-	filter, sorted 같은 중산 연산은 다른스트림을 반환
-	중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행 하지 않음
-	중간 연산을 합친 다음에 합쳐진 중간 연산을 최종연산으로 한번에 처리

> 최종 연산

-	스트림 파이프라인에서 결과를 도출
-	보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환

> 스트림 이용하기

-	질의를 수행할 (컬렉션 같은) **데이터 소스**
-	스트림 파이프라인을 구성할 **중간 연산** 연결
-	스트림 파이프라인을 실행하고 결과를 만들 **최종 연산**
	-	스트림 파이프라인의 개념은 빌더 패턴<sup>builder pattern</sup>과 비슷 <sup>http://en.wikipedia.org/wiki/Builder_pattern 참고.</sup>










