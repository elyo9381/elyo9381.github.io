---
layout: post
title: spring cloud study - 3
subtitle: "spring, cloud"
categories: spring
tags: cloud
comments: true
---
> spring study


# 스프링 클라우드 gateway


## 1. gateway
 
  처리과정 
  - client -> GateWay Handler Mapping -> Gate Way Web Handler -> Filter chain ~~(Filter, Filter) -> Dest
  

## 2. API GW를 이용한 API proxy실습

  - API GW를 통해 요청이 전달되도록 설정하는 방법

  API GW(localhost:7080) -> photoAPP(localhost:8080)

  GW를 별도로 열고 이를 연동하는 식으로 진행한다. 

  요구사항은 다양하게 진행할수있다. 
  - 시간에 따라 서비스를 열수있도록 
  - 특정 요청만 받는 서비스 등등

  자세한 참고는 spring-gateway 브런치 참고 

## 3. eurekaApp

  - api 를 관리하는 eureka server



## 4. 쓰기 요청 분산

  -  rabbitmq / 카프카 두 종류가 나와있다. 
  -  어떠한 queue서버에 메시지를 보내고 특정 topic이 발견되었을시 리시버를 통해서 받을수있는 시스템이다. 
  -  rabbimq 예제는 동작과정을 이해를 못했고.... 예제도 못해봤다...

## 5. 분산환경에서 client개발

  - ribbon
    - 로드발란스 역할을 수행가능한 client side loadBalance 입니다.
    - 대응하기가 어려운 점이 있습니다.
    - REST 
  - FeignClient(집중)
    - Declarative REST Client 
    - 어떤 클라이언트의 요청을 페인 클라이언트가받고 페인클라이언트가 해당 요청을 수행하게 됩니다. 
      - 일종의 로드밸런서 입니다. 
      - 
    - web service 클라이언트를 보다 쉽게 작성할 수 있도록 도와줍니다. 
    - 인터페이스를 작성하고 어노테이션을 붙여주면 세부적인 내용없이 사용할 수 있기 때문에 코드 복잡도가 낮아집니다. 
  

  - eureka와 feignClient를 연동하면 편리하다 
    - why? feignClient(내가 요청한 API)를 몰라도 eureka에 등록한 이름 정보만으로 쉽게 해당 API를 불러올수있습니다. 

