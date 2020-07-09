---
layout: post
title: spring AOP-1(개념소개)
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## AOP(Aspect-oriendted Programming)

* AOP(Aspect-oriendted Programming)
  : 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법.

  ```
  <===========================>
  클래스 A     클래스 B     클래스 C
  블루메소드    노랑메소드    노랑메소드
  레드메소드    블루메소드    레드메소드
  노랑메소드    블루메소드
  <===========================>

  ---------------- AOP적용하면??
  <===========================>
  클래스 A     클래스 B     클래스 C


  <===========================>
  
  AspectX    AspectY     AspectZ
  A,B,C      A,B         A,C

  ///////////////////////////////
  설명 클래스별로 실행되는 메소드가 있다.
  이때 각클래스마다 같은 코드의 메소드(코드)가 중복되고 흩어져있다.
  이를 모으고 모듈을 만드는것이 Aspect이다.
  그리고 실행되어야할곳은 Target이고
  Aspect에 존재하는 메소드 들은 Advice라고 할 수있다.
  어떤시점(join point)에 메소드를 실행 할지에 대한 정보는
  Pointcut에 정보가 저장되어있다.
  
  알맞게 적용해야하는데
  AOP적용방법을 통해서 적용해야한다. 
  ```

  Aspect ? 

  OOP를 보완하는 관계이다. 

* 주요 개념
  Aspect는 모듈이라고 생각하면된다. 
  Aspect는 Advice와 Pointcut을 가지고 있다.  
  Advice는 해야할 일들이다.  
  Pointcut은 어디에 Advice를 적용해야하는지에 대한 정보를 가지고 있다.  
  Target은 적용이 될(되는) 대상
  Join point : 메소드 호출, 메소드 실행시점 (메소드 실행할때 이 어드바이스를 끼워 넣어라의 지점)

* AOP구현체
  * 자바
    * AspectJ
    * Spring AOP 

* AOP적용 방법
  * 컴파일 타임 - 자바파일을 클래스파일로 만들때 바이트코드를 조작하면서 조작이된 바이트 코드를 생성해내는것
  * 로드 타임 - 클래스파일을 로딩하는 시점에 바이트코드는 그대로 있지만 foo()라는 메소드전에 헬로우라는 메소드를 위빙(끼어넣어서) 메모리상에 올라가있는 것 >> 로드타임에 위빙하는것
  * Run타임 - A클래스는 읽어왔고 A라는 클래스타입의 빈을 만들때 A라는 타입의 프록시빈을 만든다. 실제 A가 가지고 있는 메소드를 호출하기 직전에 헬로우를 먼저찍는일을하고 A를 호출한다. 

* ***컴파일 타임은 따로 컴파일을 한번 더해야하는점*** -> AspectJ 사용
* ***로드 타임은 위버를 사용해야 하므로 약간의 비용가 존재함*** -> AspectJ 사용
* ***Spring AOP가 사용하는 방법 처음에 약간의 비용가 존재*** -> Spring AOP 사용


