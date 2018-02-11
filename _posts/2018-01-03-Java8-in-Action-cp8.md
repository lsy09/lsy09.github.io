---
layout: post
title:  "Java8 in Action : CHAPTER8"
date:   2018-01-03
desc: "CHAPTER 8. 리팩토링, 테스팅, 디버깅"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---

## CHAPTER 8. 리팩토링, 테스팅, 디버깅

```text
- 람다 표현식으로 코드 리팩토링하기
- 람다 표현식이 객체지향 설계 패턴에 미치는 영향
- 람다 표현식 테스팅
- 람다 표현식과 스트림 API 사용 코드 디버깅
```

-	람다 표현식을 이용해서 가독성과 유연성을 높이려면 기존 코드를 어떻게 리팩토링 해야 하는지 설명
-	람다 표현식으로 전략<sup>strategy</sup>, 템플릿 메서드<sup>template method</sup>, 옵저버<sup>dbserver</sup>, 의무 체인<sup>chain of responsibility</sup>, 팩토리<sup>factory</sup> 등의 객체지향 디자인 패턴을 어떻게 간소화 할수 있는가 설명
-	람다 표현식과 스트림 API를 사용하는 코드를 테스트하고 디버깅하는 방법을 설명

> 가독성과 유연성을 개선하는 리팩토링

-	람다 표현식은 익명 클래스보다 코드를 <b>좀 더 간결하게</b> 만듬
-	람다 표현식은 동작 파라미터화의 형식을 지원하므로 <b>더 큰 유연성</b>을 갖출수 있으며, 다양한 요구사항 변화에 대응할 수 있도록 동작을 파라미터화

> 코드 가독성 개선

-	코드의 장황함을 줄여서 쉽게 이해할 수 있는 코드를 구현
-	메서드 레퍼런스와 스트림 API를 이용해서 코드의 의도(코드가 무엇을 수행하려는 것인지)를 쉽게 표현

> 익명 클래스를 람다 표현식으로 리팩토링

-	하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩토링 할수 있음
-	Runnable객체를 만드는 익명 클래스와 이에 대응하는 람다 표현식 비교

	`익명클래스`

	```java
	Runnable r1 = new Runnable(){
	  public void run(){
	    System.out.println("Hello");
	  }
	};
	```

	`람다 표현식`

	```java
	Runnable r2 = () -> System.out.println("Hello2");
	```

-	모든 익명클래스를 람다 표현식으로 변환 할수 있은것은 아님

	-	첫째, 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 가짐
	-	익명 클래스에서 this는 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가르킴
	-	둘째, 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있음(새도 변수<sup>shadow variable</sup>\)

		```java
		int a = 10;
		Runnable  r3 = () -> {
		    int a = 2;
		    System.out.println(a);      <-- 컴파일 에러
		};
		```

	-	람다 표현식으로는 변수를 가릴 수 없음

		```java
		Runnable r4 = new Runnable() {
		    @Override
		    public void run() {
		        int a = 2;
		        System.out.println(a);    <-- 정상 작동
		    }
		};
		```

-	익명 클래스를 람다 표현식으로 바꿀경우, 오버로딩에 따른 모호함 초래

	-	익명 클래스는 인서턴스화할 때 명시적으로 형식이 정해지는 반면, 람다의 형식은 콘텍스트에 따라 달라지기 때문

		```java
		interface Task{
		    public void execute();
		}


		public static void doSomething(Runnable r){ r.run();}
		public static void doSomething(Task r){ r.execute();}
		```

		`Task 구현 익명클래스 전달`

		```java
		doSomething(new Task() {
		    @Override
		    public void execute() {
		        System.out.println("Danger danger!!");
		    }
		});
		```

-	익명 클래스를 람다 표현식으로 바꾸면 메서드 호출 시 Runnable과 Task 모두 대상 형식이 될수 있으므로 모호함 발생

-	명시적 형변환(Task)를 이용해서 모호함을 제거

	```java
	doSomething((Task)() -> System.out.println("Danger danger!!"));
	```

> 람다 표현식을 메서드 레퍼런스로 리팩토링

-	람다 표현식 대신 메서드 레퍼런스를 이용하면 가독성을 높일 수 있음
-	메서드 레퍼런스의 메서드명으로 코드의 의도를 명확하게 알릴 수 있음

	```java
	Map<CaloricLevel, List<lambdasinaction.chap6.Dish>> dishesByCaloricLevel() {
	    return lambdasinaction.chap6.Dish.menu.stream().collect(
	            groupingBy(dish -> {
	                if (dish.getCalories() <= 400) return CaloricLevel.DIET;
	                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
	                else return CaloricLevel.FAT;
	            }));
	}
	```