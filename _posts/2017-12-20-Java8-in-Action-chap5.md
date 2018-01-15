---
layout: post
title:  "자바 8 인 액션(Java8 in Action) - CHAPTER 5"
date:   2017-12-20
excerpt: "스트림 활용"
categories: [book]
comments: true
---

# CHAPTER 5. 스트림 활용

```text
- 필터링 슬라이싱 매칭
- 검색, 매칭, 리듀싱
- 특정 범위의 숫자와 같은 숫자 스트림 사용하기
- 다중 소스로부터 스트림 만들기
- 무한 스트림
```

-	데이터를 어떻게 처리할지 스트림 API가 관리함으로 편리하게 데이터 관련 작업을 진행
-	스트림 API 내부적으로 다양한 최적화가 이루어질수 있음
-	내부 반복 분 아니라 코드를 병렬로 실행할지 여부도 결정
-	순차적인 반복을 단일 스레드로 구현했기 때문에 외부 반복으로는 불가능

### 5.1 필터링과 슬라이싱

> 프레디케이트로 필터링 - 스트 인터페이스는 filter 메서드를 지원 - filter 메서드는 **프레디케이트**(불린을 반환하는 함수)를 인수로 받아서 프레디케이트와 잃치하는 모든 요소를 포함하는 스트림으로 반환

`예제`

```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)     <--- 채식요리인지 확인하는 메서드 레퍼런스
                                .collect(toList());
```

> 고유 요소 필터링 - 고유 요소로 이루어진 스트림을 반환하는 distinct 라는 메서드도 지원

`예제`

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i%2 ==0)
       .distinct()
       .forEach(System.out::println);
```

> 스트림 축소 - 주어진 사이즈 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원 - 스트림이 정렬되어 있으면 최대 n개의 요소를 반환

`예제`

```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .limit(3)
                        .collect(toList());
```

-	정렬되지 않은 스트림에도 limit 를 사용할수 있음, 소스가 정렬되지 않았다면 limit의 결과도 정렬되지 않은 상태로 반환

> 요소 건너뛰기 - 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원 - n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환 - limit(n)과 skip(n)은 상호 보완적인 연산을 수행

`예제`

```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .skip(2)
                        .collect(toList());
```

### 5.2 매핑

-	스트림API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공

> 스트림의 각 요소에 함수 적용하기 - 함수를 인수로 받는 map 메서드를 지원 - 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑

`예제 (Dish::getName을 map 메서드로 전달해서 스틀미의 요리명을 추출하는 코드)`

```java
List<String> dishNames = menu.stream()
                             .map(Dish::getName)
                             .collect(toList());
```

`예제 (메서드 레퍼런스 String::length를 map에 전달해서 문제를 해결)`

```java
List<String> words = Arrays.asList("Java8", "Lambdas", "In", "Action");
List<Integer> wordLengths = words.stream()
                                 .map(String::length)
                                 .collect(toList());
```

`다른 map 메서드를 연결 <sup>chaining</sup>`

```java
List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
```

> 스트림 평면화 - 리스트에서 **고유 문자**로 이루어진 리스트를 반환

```java
word.stream()
    .map(word -> word.split(""))
    .distinct()
    .
```
