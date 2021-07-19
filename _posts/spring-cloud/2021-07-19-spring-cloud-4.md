---
layout: post
title: spring cloud study - 4
subtitle: "spring, cloud"
categories: spring
tags: cloud
comments: true
---
> spring study


## 7. 장애 허용 시스템 구축

  - resilience4j 를 활용한 circuit breaker 패턴 적용


  - resilience + 4j : 회복탄력성(데미지를 받았을때 극복하는성질) + 4j(for java)
  - 질병관리, 트래픽제어 

  - 트래픽양을 조절할수는 없지만 어떻게 받고 어디로 보낼지를 설정할수있다. 



  - 격백 패턴 : 장애가 발생했을때 그것을 차단하는 패턴
    - 나의 기능이 a-z에서 사용되는데 어떤 부분에(d-api) 에러가 발생했을시 d 서비스를 차단하는것
    - 장애가 발생한 일부분의 api를(시스템)을 막는 기능이라고 볼수 있다.


  1. bulkhead 패턴을 메소드 호출 방식으로 구현 with resilience4j
    - 메소드로 설정한 조건을 만족하지 않으면 거절하는 방식
  2. circuit 
    - 안정성 회복을 위해서 사용됨 
    - 상태값이 존재하여 특정 오류횟수에 도달하면 요청을 처리하지 못하게 하는것
    - 그리고 일정시간 지난뒤 정상적이 요청이 들어왔을 받아들입니다. 
    - 즉 상태값에 따라 열었다 닫았다 하는 기능입니다. 

