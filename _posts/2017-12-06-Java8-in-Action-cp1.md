---
layout: post
title:  "Java8 in Action : CHAPTER1"
date:   2017-12-06
desc: "CHAPTER 1. 자바 8을 눈여겨봐야 하는 이유"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---


## CHAPTER 1. 자바 8을 눈여겨봐야 하는 이유

-	자바 8이 등장하기 이전에는 나머지 코어를 활용하려면 스레드를 사용, 하지만 스레드를 사용하변 관리하기 어렵고 많은 문제가 발생할수 있다는 단점
-	데이터베이스 질의 언어에서 표현식을 처리하는 것처럼 병렬 연산을 지원하는 스트림이라는 새로운 API를 제공
-	**메서드에 코드를 전달하는 간결 기법**<sup>메서드 레퍼런스와 람다</sup>, 인터페이스의 **디폴트 메서드**가 추가
-	자바 8 기법은 **함수형 프로그래밍<sup>functional-style programming</sup>** 에서 위력 발휘

### 1.1 왜 아직도 자바는 변화하는가?

-	새로운 언어가 등장하면서 진화하지 않은 기존언어는 사장
-	자바는 지난 15년간 경쟁 언어를 대신하며 커다란 생태계를 성공적으로 구축

> 프로그래밍 언어 생태계에서의 자바의 위치

-	처음부터 많은 유용한 라이브러리를 포함하는 잘 설계된 객체지향 언어로 시작
-	스레드와 락을 이용한 소소한 동시성도 지원
-	코드를 JVM 바이트코드로 컴파일 하는 특징 때문에 자바는 인터넷 애플릿 프로그램의 주요 언어가 됨

> 스트림 처리(stream processing)

-	첫 번째 프로그래밍 개념
	-	**스트림** 한번에 한개씩 만들어지는 연속적인 데이터 항목들의 모임
-	입력 스트림에서 데이터를 한개씩 읽어 들이며 마찬가지로 출력 스트림으로 데이터를 한 개씩 기록
-	고수준으로 추상화 하여 일련의 스트림으로 만들어 처리 할 수 있음
-	스레드라는 복잡한 작업을 사용하지 않으면서도 **공짜**로 병렬성을 얻을 수 있음

> 동작 파라미터화로 메서드에 코드 전달하기

-	두 번째 프로그래밍 개념
	-	코드 일부를 API로 전달하는 기능
-	메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공 이러한 기능을 이론적으로 **동작 파라미터화<sup>behavior parameterization</sup>** 라고 부름

> 병렬성과 공유 가변 데이터

-	세 번째 프로그래밍 개념
	-	'병렬성을 공짜로 얻을 수 있다'라는 말에서 시작
-	스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 **안전하게 실행**될 수 있어야 함
-	다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만들려면 공유된 가변 데이터(shared mutable data)에 접근 하지 않아야 함
-	스트림을 이용하면 기존의 자바 스레드 API보다 쉽게 병렬성을 활용 할 수 있음
-	공유되지 않은 가변 데이터<sup>no shared mutable data</sup>, 메서드와 함수 코드를 다른 메서드로 전달하는 두가지 기능은 **함수형 프로그래밍** 패러다임의 핵심적인 사항 임

> 자바가 진화해야 하는 이유

-	전통적인 객체지향 프로그래밍과 함수형 프로그래밍은 완전 상극, 자바 8에서 함수형 프로그래밍을 도입함으로써 두 가지 프로그래밍 패러다임의 장점을 모두 활용 할수 있게 됨

---

### 1.2 자바 함수

-	함수(function)라는 용어는 **메서드(method)**, 특히 정적 메서드<sup>static method</sup>와 같은 의미로 사용
-	**수학적인 함수**처럼 사용되며 부작용을 일이키지 않는 함수를 의미
-	프로그래밍 언어의 핵심은 값을 바꾸는것 프로그래밍 언어에서는 이 값을 일급<sup>first-class:퍼스트클래스</sup>값 이라고 함
-	자유롭게 전달할 수 없는 구조체는 이급 시민이라고 하며 메서드, 클래스 등은 이급 자바 시민에 해당

> 메서드와 람다를 일급 시민으로

##### 메서드 레퍼런스(method reference)

-	디렉터리에서 모든 숨겨진 파일을 필터링 한다고 가정, 주어진 파일이 숨겨져 있는지 여부를 알려주는 메서드를 구현

`이전 자바 : FileFilter 객체로 isHidden 메서드를 감산 다음에 File.listFiles 메서드로 전달 해야 했음.`

```java
File[] hiddenFiles = new File(".").listFiles(new  FileFilter() {
    public bollean accept(File file){
        return file.isHidden(); ----> 숨겨진 파일 필터링!
    }
});
```

`자바 8 : 메서드레퍼런스 :: 문법을 이용해서 직접 isHidden함수를 lisstFiles 메서드로 전달 할수 있음.`

```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

-	File 클래스에 이미 isHidden이라는 함수는 준비되어있으므로 자바8의 메서드 레퍼런스<sup>method reference</sup> **::** ('이 메서드를 값으로 사용하라'는 의미) 를 이용해서 listFiles에 직접 전달 가능
-	자바 8에서는 더 이상 메서드가 이급값이 아닌 일급값
-	메서드가 아닌 **함수**라는 용어를 사용했다는 사실 주목

> 람다: 익명함수 - 기명(named)메서드를 일급값으로 취급할 뿐 아니라 **람다**(또는 익명 함수anonymous function)를 포함하여 **함수도 값**으로 취급 람다 문법형식으로 구현된 프로그램을 함수형 프로그래밍 즉 '함수를 일급 값으로 넘겨주는 프로그램을 구현한다'라고 함

-	코드 넘겨주기: 예제

`이전 자바`

```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
  List<Apple> result = new ArrayList<>();  <--반환되는 result는 List로 처음에는 비어 있지만 점점 녹색 사과로 채워진다.
  for(Apple apple : inventory){
    if("green".equals(apple.getColor())){   <-- 녹색사과만 선택
        result.add(apple);
    }
  }
  return result;
}

public static List<Apple> filterHeavyApples(List<Apple> inventory){
  List<Apple> result = new ArrayList<>();  
  for(Apple apple : inventory){
    if(apple.getWeight() > 150){   <-- 150이상의 무거운 애플만 선택
        result.add(apple);
    }
  }
  return result;
}
```

`자바 8 :: 코드를 인수로 넘겨줄 수 있으므로 filter 메서드를 중복으로 구현할 필요 없음.`

```java
public static boolean isGreenApple(Apple apple){
    return "green".equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple){
    return apple.getWeight()>150;
}

public interface Predicate<T>{ <-- 보통 java.util.function에서 import
    boolean test(T t);
}

public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }
    return result;
}
```

`다음처럼 메서드 호출`

```java
filterApples(inventory, Apple::isGreenApple);
```

`또는, 다음과 같은 호출`

```java
filterApples(inventory, Apple::isHeavyApple);
```

> 메서드 전달에서 람다로

-	람다가 몇줄 이상으로 길어진다면(즉, 조금 복잡한 동작을 수행하는 상황) 익명 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 레퍼런스를 활용하는것이 바람직.
-	자바8 에서는 filter와 비슷한 동작을 수행하는 연산집합을 포함하는 새로운 스트림 API(컬렉션Collection 과 비슷하며 함수형 프로그래머에게 더 익숙한 API)를 제공.
-	컬렉션과 스트림 간에 변환할 수 있는 메서드(map, reduce 등)도 제공.

---

### 1.3 스트림

-	자바 애플리케이션은 컬렉션을 만들고 활용 함.
-	고가의 트랜잭션<sup>Transaction</sup>만 필터링한 다음에 통화로 결과를 그룹화 해야 한다고 가정.

`이전 자바 : 기본코드`

```java
Map<Currency, List<Transaction>> transactionByCurrencies =
new HashMap<>();        <-- 그룹화된 트랜잭션을 더할 Map생성
for(Transaction transaction : transactions){        <-- 트랜잭션의 리스트를 반복
    if(transaction.getPrice() > 1000){              <-- 고가의 트랜잭션을 필터링
        Currency currency = transaction.getCurrency();      <-- 트랜잭션의 통화 추출
        List<Transaction> transactionsForCurrency =    
            transactionByCurrencies.get(currency);
        if(transactionsForCurrency == null){        <-- 현재 통화의 그룹화된 맵에 항목이 없으면 새로 만듬
            transactionForCurrency = new ArrayList<>();
            transactionsByCurrencies.put(currency, transactionsForCurrency);
        }
        transactionsForCurrency.add(transaction);    <-- 현재 탐색된 트랜잭션을 같은 통화의 트랜잭션 리스트에 추가
    }
}
```

`자바 8 : 스트림 API 이용`

```java
Map<Currency, List<Transaction>> transactionByCurrencies =
    transactions.stream()
        .filter((Transaction t) -> t.getPrice() > 1000)     <-- 고가의 트랜잭션 필터링
        .collect(groupingBy(Transaction::getCurrency));     <-- 통화로 그룹화함
```

-	스트림 API를 이용하면 컬렉션 API와는 상당히 다른 방식으로 데이터를 처리할 수 있다는 사실 기억.
-	컬렉션에서는 직접 처리 이런 방식의 반복을 **외부반복<sup>external iteration</sup>**.
-	스트림API를 이용하면 루프를 신경쓸 필요 없고 라이브러리 내부에서 모든데이터가 처리됨, 이런 방식의 반복을 **내부반복<sup>internal iteration</sup>**.

> 멀티스레딩은 어렵다`이전 자바` - 멀티스레딩 환경에서 각각의 스레드는 동시에 공유된 데이터에 접근, 데이터 갱신. - 멀티스레딩 모델은 순차적인 모델보다 다루기가 어려움.

`자바 8` 
- 스트림API(java.util.stream)로 '컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제' 그리고 '멀티코어 활용 어려움'이라는 두 가지 문제를 모두 해결.

---

### 1.4 디폴트 메서드

-	라이브러리 설계자가 더 쉽게 변화할 수 잇는 인터페이스를 만들 수 있도록 디폴트 메서드 추가.
-	특정 프로그램을 구현하는데 도움을 주는 기능이 아니라 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능.

`이전 자바` 
- List<T>(List가 구현하는 인터페이스인 Collection<T>도 마찬가지)가 stream 이나 parallelStream 메서드를 지원하지 않는다는 것이 문제

`자바 8` 
- 구현 클래스에서 구현하지 않아도 되는 메서드를 인터페이스가 포함할 수 있는 기능도 제공. - 메서드 바디는 클래스 구현이 아니라 인터페이스의 일부로 포함. - 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장, 인터페이스 규격명세서에 default라는 새로운 키워드를 지원.

---

### 1.5 함수형 프로그램에서 가져온 다른 유용한 아이디어

`자바 8` 
- NullPointer 예외를 피할 수 있도록 도와주는 Optional<T> 클래스 제공. - Optional<T>는 값을 갖거나 갖지 않을 수 있는 컨테어너 객체이며, Optional<T>는 값이 없는 상황을 어떻게 처리할지 명시적으로 구현하는 메서드를 포함. - Optional<T>를 사용하면 NullPointer 예외를 피할 수 있음.













