---
layout: post
title: security batch-1
subtitle: "spring, batch"
categories: spring
tags: security
comments: true
---
> spring study

## spring batch


## 1.spring batch

  - 큰 단위의 작업을 일괄처리
  - 대부분 처리량이 많고 비 실시간성 처리에 사용
    - 대용량 데이터 계산, 정산, 통계, 데이터베이스,변환 등
  - 컴퓨처 자원을 최대로 활용
    - 컴퓨터 자원 사용이 낮은 시간대에 배치를 처리하거나
    - 배치만 처리하기 위해 사용자가 사용하지 않는 또 다른 컴퓨터 자원을 사용
  - 사용자 상호작용으로 실행되기 보단, 스케줄러와 같은 시스템에 의해 실행되는 대상
    - 예를들면 매일 오전 10시에 배치 실행, 매주 월요일 12 시마다 실행
    - crontab, jenkins... 


  - 배치 처리를 하기 위한 spring Framework기반 기술
    - spring에서 지원하는 기술 적용 가능
    - DI, AOP 서비스 추상화 
  - 스프링 배치의 실행 단위인 Jop과 step
  - 비교적 간단한 작업(Tasklet) 단위처리와 대량 묶음(Chunk) 단위처리 알아볼것


## 2. 스프링 배치 example를 위한 프로젝트 설정

  - spring boot 기반
  - java8
  - gradle
  - JPA, JDBC
  - SpringBatch
  - loombook
  - h2, mysql


  - @EnableBatchProcessing // 스프링 배치를 사용하기 위한 애노테이션 

## 3. 스프링 배치의 과정 


  - 스프링배치는 job타입의 빈이 생성되면 잡런처 객체에 의해서 잡을 실행합니다. 
  - 잡은 스탭을 실행하게 됩니다. 

  - 잡리파지터리는 DB,memory에 스프링배치가 실행될수있도록 배치의 메타 데이터를 관리하는 클래스입니다. 
  - 잡리파지터리를 스프링배치의 전반적인 데이터를 관리하는 클래스입니다. 

  - job은 jobLauncher에 의해 실행
  - Job은 배치의 실행 단위를 의미
  - Job은 N개의 step을 실행할 수 있으며, 흐름을 관리할 수 있다. 
    - 예를 들면, A Step 실행 후 조건에 따라 B Step 또는 C Step을 실행 설정


  - Step은 Job의 세부 실행 단위이며, N개가 등록되 실행된다.
  - Step의 실행 단위는 크게 2가지로 나눌 수 있다. 
    - 1. chunk 기반 : 하나의 큰 덩어리를 n개씩 나눠서 실행
    - 2. Task기반 : 하나의 작업 기반으로 실행
  - chunk기반 step은 itemReader, ItemProcessor, ItemWriter가 있다. 
    - 여기서 item은 배치 처리 대상 객체를 의미한다.
  - ItemReader는 배치처리 대상 객체를 읽어 itemProcessor또는 ItemWriter에게 전달한다.
    - 예를들면, 파일 또는 DB에서 데이터를 읽는다. 
  - ItemProcessor는 input객체를 output객체로 filtering 또는 processing 해 ItemWriter에게 전달한다. 
    - 예를들면, iterReader에서 읽은 데이터를 수정 또는 ItemWirter 대상인지 filtering한다.
    - ItemProcessor는 optional 하다.
    - ItemProcessor가 하는일을 itemreader또는 itemWriter가 대신할 수 있다. 
  - ItemWriter는 배치 처리 대상 객체를 처리한다.
    -  예를들면, DB update를 하거나, 처리 대상 사용자에게 알림을 보낸다. 


  - Job은 하나의 이름과 파라미터로 실행된다. 
    - job instance의 생성기준은 이름과 파라미터를 가지고 생성, 재실행,실패를 판단한다. 
    - 하나의 job은 같은 파라미터로 여러개 생성하거나 실행 할 수 없다. 
  - ExcutionContext는 job,step을 모두 포함하는 최상위 클래스이며 이를 통해서 데이터가 공유될수있다. 

  > 중요 
  - JobExcution은 해당 Job내에서 데이터 공유가 가능하다 (key를 불러올수있음)
  - 하지만 각각의 Step은 본 Step에서만 데이터 입력등이 가능하고 저장, 삽입 가능합니다. 
  - Job 내부에 2~n 개의 step존재시 step 끼리 데이터 공유는 불가합니다. 


 