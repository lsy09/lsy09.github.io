---
layout: post
title:  "RESTful API"
date:   2018-01-22
desc: "RESTful API에 대한 정의"
keywords: "RESTful,API"
categories: [Web]
tags: [RESTful,API]
icon: fa-internet-explorer
---


> 월드 와이드 웹(World Wide Web a.k.a WWW)과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다.
> 
> `네트워크 아키텍처 원리`란 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반을 일컽는다.
>
> `REST`란, REpresentational State Transfer의 약자이다. 

`RESTful`
- 설계방식, 가볍고 개발 속도를 올릴 수 있는 개발방법이다.
- REST 원리를 따르는 시스템은 종종 RESTful이란 용어로 지칭한다.
- 아키텍처. 웹의 특징을 잘살리는것을 말하고 이를 위한 아키텍처 룰이 몇가지 있다.

#### REST 아키텍처에 적용되는 6가지 제한 조건(제한조건 준수한다면, 개별 컴포넌트 자유로운 구현 가능)
- 클라이언트/서버구조(Client-Server) : REST서버는 API제공, 클라이언트와 서버에서 개발해야할 내용이 명확해지고 서로간의 의존성이 줄어들게 된다.
- 무상태(Stateless) : 작업을 위한 상태정보를 따로 저장하고 관리 하지 않는다. 별로로 저장하고 관리하지 않기 때문에 API서버는 들어오는 요청만을 단순히 처리하면된다.
세션과 같은 컨텍스트 정보를 신경쓸 필요가 없기 때문에 구현이 단순해진다.
- 캐시처리가능(Cacheable) : HTTP라는 기존 웹표준을 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 활용이 가능하므로 HTTP프로토콜 표준에서 사용하는 
Last-Modifiend태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.
- 자체 표현 구조(Self-descriptiveness) : REST의 또 다른 특징 중 하나는 REST API 메시지만 보고도 이를 쉽게 이해 할 수 있는 자체 표현 구조로 되어있다는 것이다.
- 계층화(Layered System) : REST서버는 다중 계층으로 구성될 수 있으며 보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고 PROXY, 게이트웨이 같은 
네트워크 기반의 중간매체를 사용할 수 있게 한다.
- 유니폼 인터페이스(Uniform Interface) : URI로 지정한 리소스에 대한 조작을 통일되고 한정적 인터페이스로 수행하는 아키텍처 스타일을 말한다.









