---
layout: post
title: spring cloud study - 2
subtitle: "spring, cloud"
categories: spring
tags: cloud
comments: true
---
> spring study


# 스프링 클라우드 


## 1. 스프링 클라우드?

  클라우드는 분산된 시스템입니다. 복잡성은 애플리케이션 레벨에서 네트워크 레벨로 움직이고 이들끼리 서로 상호작용을 진행합니다. 

  > 출처 : https://spring.io/cloud

  클라우드를 위한 12 요소 

  1. 코드베이스 : 버전관리되는 하나의 코드베이스와 여러가지 배포 
  2. 종속성 : 명시적으로 선언되고 분리된 종속성
  3. 설정 : 환경에 저장된 설정
  4. 백엔드 서비스 : 백엔드 서비스를 연결된 리소스로 취급
  5. 빌드,릴리즈,실행 : 철저하게 분리된 빌드와 실행단계
  6. 프로세스 : 애플리케이션을 하나 혹은 여러개의 무상태 프로세스로 실행
  7. 포트바인딩 : 포트 바인딩을 사용해서 서비스를 공개함
  8. 동시성 : 프로세스 모델을 사용한 확장
  9. 폐기가능 : 빠른시작과 그레이스풀 셧다운을 통한 안정성 극대화 
  10. 개발/프로덕션환경 일치 : 개발, 스테이징, 프로덕션 환경을 최대한 비슷하게 유지
  11. 로그 : 로그를 이벤트 스트림으로 취급
  12. 관리자 프로세스 : 관리/유지 작업을 일회성 프로세스로 실행
   
  > 출처 : www.12factor.net

  
  ### 스프링 클라우드 아키텍처 
  
  IoT,Mobile,Browse -> APIGateway -> Serviceregistry -> MicroService ;

  - APIGateway 
    - Zuul : 더이상 개발하지 않는 모델
    - Zuul2 : 넷플릭스가 개발한 게이트웨이
    - Spring cloud gateway : 스프링에서 관리하는 클라우드 게이트 웨이
    
  - Service registry
    - DNS & IP vs Navtice Cloud
    - Eureka를 사용할 것임 
 
  - Config server : 공통적으로 사용하는 리소스 관리 (DB,gateway.. Service registry.. 등)
    - Spring cloud Config : 공통적으로 사용하는 리소스 관리 (DB,gateway.. Service registry.. 등) 
    - Spring cloud event bus : 업데이트(변경사항 관리 )
    - Spring Vault : 암호화 및 민감정보 관리 
  
  - Distributed Tracing : 분산된 서버의 로깅 방법
    - MDC(Mapped Diagnostic Context) : SL4j등에서 로깅할때 사용하는 방법
    - Spring Cloud sleuth / zipkin : 스프링에서 관리하는 로깅 추적 방법 

## 2. 스프링 클라우드 프로젝트 셋팅

  - image upload 시스템
  - api만 제공되고 테스트는 swagger를 통해서 진행함
  - 
