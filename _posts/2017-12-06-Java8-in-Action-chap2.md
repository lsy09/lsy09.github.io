---
layout: post
title:  "자바 8 인 액션(Java8 in Action) - CHAPTER 2"
date:   2017-12-06
excerpt: "동작 파라미터화 코드 전달하기"
categories: [book]
comments: true
---

# CHAPTER 2. 동작 파라미터화 코드 전달하기

-	동작 파라미터화<sup>behavior parameterization</sup>를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응
-	동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미

### 2.1 변화하는 요구사항에 대응하기

-	예제 코드를 점점 개선하면서 유연한 코드를 만드는 모범사례.

> 첫 번째 시도 : 녹색 사과 필터링

```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<>();     <-- 사과 누적 리스트
    for(Apple apple: inventory){
        if("green".equals(apple.getColor())){   <-- 녹색 사과만 선택
            result.add(apple);
        }
    }
    return result;
}
```

> 두 번째 시도 : 색을 파라미터화

-	색을 파라미터화 할수 있도록 메서드에 파라미터

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, String color){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(apple.getColor().equals(color)){
            result.add(apple);
        }
    }
    return result;
}
```

-	메서드 호출

```java
List<Apple> greenApples = filterApplesByColor(inventory, "green");
List<Apple> redApples  = filterApplesByColor(inventory, "red");
```

-	다양한 무게에 대응 할수 있도록 무게 정보 파라미터 추가

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(apple.getWeight() > weight){
            result.add(apple);
        }
    }
    return result;
}
```

-	위의 코드도 좋은 해결책이라고 할수 있지만 소프트웨어 공학의 **DRY(don't repeat yourself)** 원칙을 어기는 것.
-	성능을 개선하려 한다면, 한줄이 아니라 메서드 **전체** 구현을 고쳐야 함.

> 세 번째 시도 : 가능한 모든 속성으로 필터링

-	모든 속성을 메서드 인수로 추가한 모슴

```java
public static List<Apple> filterApples(List<Apple> inventory, String color, int weight, boolean flag){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if((flag && apple.getColor().equals(color)) ||
            (!flag && apple.getWeight() > weight)){     <-- 색이나 무게를 선택하는 방법이 맘에 들지 않음.
            result.add(apple);
        }
    }
    return result;
}
```

-	다음처럼 위 메서드를 활용

```java
List<Apple> greenApples = filterApples(inventory, "green", 0, true);
List<Apple> heavyApples = filterApples(inventory, "", 150, false);
```

-	여러 중복된 필터 메서드를 만들거나 아니면 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현, 지금까지 문자열, 정수 불린 등의 값으로 filterApples 메서드를 파라미터화 함.
-	어떤기준으로 사과를 필터링할 것인지 효과적으로 전달 할수 있다면 더 좋은 방법.

---

### 2.2 동작 파라미터화

-	어떤 속성에 기초해서 불린값을 반환, 이와 같은 동작은 **프레디케이트<sup>Predicate</sup>** 라고 함.
	-	프레디케이트<sup>Predicate</sup> : 함수형 인터페이스, 조건식을 표현하는데 사용되는 인터페이스 이며, 매개변수는 1개을 받고 반환 타입음 boolean을 리턴.

`그림 2-1 사과를 선택하는 다양한 전략`

![스크린샷 2017-12-05 오후 6.02.34](https://i.imgur.com/Moe1QwQ.png)

-	위를 전략 디자인 패턴<sup>strategy design pattern</sup>
	-	전략 디자인 패턴은 각 알고리즘(전략이라 불리는)을 캡슐화 하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법.
-	**동작 파라미터화**, 즉 메서드가 다양한 동작(또는 전략)을 **받아서** 내부적으로 다양한 동작을 **수행** 할 수 있음.

> 네 번째 시도 : 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){          <-- 프레디케이트 객체로 사과 검사 조건을 캡슐화
            result.add(apple);
        }
    }
    return result;
}
```

> 코드/동작 전달하기

-	위의 코드를 통해 ApplePredicate를 적절하게 구현하는 클래스를 만들면 된다. 속성과 관련한 모든 변화에 대응할 수 있는 유연한 코드를 준비 한것.
-	이용자가 원하는 조건을 Predicate로 구현하여 언제든 요구사항을 반영.

`filterApples 메서드의 동장을 파라미터화`

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate{
    public boolean test(Apple apple){
        return "red".equals(apple.getColor())
                && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples =
    filter(inventory, new AppleRedAndHeavyPredicate());
```

##### 한 개의 파라미터, 다양한 동작

-	컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리 할 수 있다는 것이 동작 파라미터화의 강점.
-	유연한 API를 만들려면 동작 파라미터화라는 도구가 반드시 필요.

---

### 2.3 복잡한 과정 간소화

`동작 파라미터화 : 프레디케이트로 사과 필터링`
