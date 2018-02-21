---
layout: post
title:  "Java8 in Action : CHAPTER10"
date:   2018-02-17
desc: "CHAPTER 10. null 대신 Optional"
keywords: "Java,Java8"
categories: [Book]
tags: [Java,Java8]
icon: icon-java-bold
---

## CHAPTER 10. null 대신 Optional

```text
- null 레퍼런스의 문제점과 null을 멀리해야 하는 이유
- null 대신 Optional: null로부터 안전한 도메인 모델 재구현하기
- Optional 활용: null 확인 코드 제거하기
- Optional에 저장된 값을 확인하는 방법
- 값이 없을 수도 있는 상황을 고려하는 프로그래밍
```

> null 때문에 발생하는 문제

- 에러의 근원이다
    - NullPointerException은 자바에서 가장 흔히 발생하는 에러
    
- 코드를 어지럽힌다
    - 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어짐
    
- 아무 의미가 없다
    - null은 아무 의미도 표현하지 않음. 특히 정적 형식 언어에서 값이 없을을 표현하는 방법으로 적절하지 않음
    
- 자바 철학에 위배 된다
    - 자바는 개발자로부터 모든 포인터를 숨김. 하지만 예외가 있는데 그것이 바로 null 포인터
    
- 형식 시스템에 구멍을 만든다
    - null은 무형식이며 정보를 포함하고 있지 않으므로 모든 레퍼런스 형식에 null을 할당할 수 있음. 이런식으로 null이 할당되기 시작하면서
    시스템의 다른 부분으로 null이 퍼졌을때 애초에 null이 어떤 의미로 사용되었는지 알 수 없음
    
> 다른 언어는 null 대신 무엇을 사용하나?

- **안전 내비게이션 연산자**<sup>safe navigation operator</sup>(?.)를 도입해서 null문제를 해결함.

`groovy 예제`

```groovy
def carInsuranceName = person?.car?.inserance?.name
```

- 하스켈, 스칼라 등의 함수형 언어
    - 하스켈은 선택형값<sup>optional value</sup>을 저장할 수 있는 Maybe라는 형식을 제공
    - 스칼라도 T 형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 Option[T]라는 구조를 제공
    
> Optional 클래스 소개

- 자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공
- Optional은 선택형값을 캡슐화하는 클래스
- 값이 없으면 Optional.empty 메서드로 Optional을 반환
- Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드

```java
public class Person {
    private Optional<Car> car;      <-- 사람이 차를 소유했을 수도 소유하지 않았을 수도 있으므로 Optional에 정의
    public Optional<Car> getCar() { return car; }
}

public class Car {
    private Optional<Insurance> insurance;   <-- 자동차가 보험에 가입되어 있을 수도 가입되어 있지 않았을 수도 있으므로 Optional에 정의
    public Optional<Insurance> getInsurance(){ return insurance; }
}

public class Insurance {
    private String name;    <-- 보험회사에는 반드시 이름이 있음
    public String getName() { return name; }
}
```
    
- Optional 클래스를 사용하면서 모델의 의미<sup>semantic</sup>가 더 명확해졌음을 확인


> Optional 적용 패턴

>> Optional 객체 만들기

>>> 빈 Optional

- 정적 팩토리 메서드 Optional.empty로 빈 Optional 객체를 얻을수 있음
        
    ```java
    Optional<Car> optCar = Optional.empty();
    ```
    
>>> null이 아닌 값으로 Optional 만들기

- 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들수 있음

    ```java
    Optional<Car> optCar = Optional.of(car);
    ```

    - car가 null이라면 즉시 NullPointerException이 발생(Optional을 사용하지 않았다면 car의 프로퍼티에 접근하려 할때 에러 발생)

>>> null값으로 Optional 만들기

- 정적 팩토리 메서드 Optional.ofNullable로 null 값을 저장할 수 있는 Optional을 만들수 있음
    
    ```java
    Optional<Car> optCar = Optional.ofNullable(car);
    ``` 

    - car가 nulls이면 빈 Optional 객체 반환

> 맵으로 Optional의 값을 추출하고 변환

- 보통 객체의 정보를 추출할때는 Optional을 사용할 때가 많음

```java
String name = null;
if(insurance != null){
    name = insurance.getName();
}
```

- 이런 유형의 패턴에 사용할 수 있도록 Optional은 map 메서드를 지원

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

- Optional의 map 메서드는 스트림의 map 메서드와 개념적으로 비슷

> flatMap으로 Optional 객체 연결

```java
Optional<Person> optPerson = Optional.of(parson);
Optional<String> name = 
        optPerson.map(Person::getCar)
                 .map(Car::getInsurance)
                 .map(Insurance::getName);
```

- 위 코드는 map 연산의 결과는 Optional<Optional<Car>> 형식의 객체, getInsurance는 또 다른 Optional 객체를 반환하므로 
getInsurance를 지원 하지 않음
- 스트림의 flatMap 함수를 인수로 받아서 다른 스트림을 반환하는 메서드, 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화 됨
- 이차원 Optional을 일차원 Optional로 평준화 해야 함

```java
public String getCarInsuranceName(Optional<Person> person){
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");   <-- 결과 Optional이 비어있으면 기본값 사용
}
```

> 디폴트 액션과 Optional 언랩

- Optional이 비어있을 때 디폴트값을 제공할 수 있는 orElse 메서드 로 값을 읽자. Optional 클래스는 Optional 인스턴스에서 값을 읽을 수 있는 
다양한 인스턴스 메서드를 제공
    - get()은 값을 읽는 가장 간단한 메서드면서 가장 안전하지 않은 메서드 임
    - orElse(T other) 메서드를 이용하면 Optional이 값을 포함하지 않을 때 디폴트값을 제공
    - orElseGet(Supplier<? extends T> other)는 orElse 메서드에 대응하는 게으른 버전의 메서드, Optional에 값이 없을때만
    Supplier가 실행되기때문임. 디폴트 메서드를 만드는 데 시간이 걸리거나 Optional이 비어 있을때만 디폴드 값을 생성하고 
    싶다면(디폴트 값이 반드시 필요한 상황) orElseGet(Supplier<? extends T> other)를 사용
    - orElseThrow(Supplier<? extends X> exceptionSupplier)는 Optional이 비어있을때 예외를 발생시킨다는 접에서 get메서드와 비슷, 
    하지만 이 메서드는 발생시킬 예외의 종류를 선택 할수 있음
    - ifPresent(Consumer<? super T> consumer)를 이요하면 값이 존재 할때 인수로 넘겨준 동작을 실행할 수 있음, 
    값이 없으면 아무 일도 일어나지 않음
    
> 두 Optional 합치기

- 두 Optional을 인수로 받아서 Optional<Insurance>를 반환하는 null 안전 버전<sup>null-safe version</sup>의 메서드를 구현해야 한다고 가정,
인수로 전달한 값중 하나라도 비어있으면 빈 Optional<Insurance>를 반환. Optional 클래스는 Optional이 값을 포함하는지 여부를 알려주는 
isPresent라는 메서드도 제공

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car){
    if(person.isPresent() && car.isPresent()){
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    }else{
        return Optional.empty();
    }
}
```

> 필터로 특정값 거르기

- Insurance 객체가 null인지 여부를 확인한 다음에 getName 메서드를 호출

```java
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
    System.out.println("ok");
}
```

- Optional 객체에 filter 메서드를 이용해서 다음과 같이 코드를 재구현 

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurnace -> 
                        "CambridgeInsurance".equals(insurance.getName()))
                        .ifPresent(x -> System.out.println("ok"));
```