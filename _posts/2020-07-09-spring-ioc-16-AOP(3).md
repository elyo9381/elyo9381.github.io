---
layout: post
title: spring AOP-3(spring AOP)
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## spring AOP-3(spring AOP)

  스프링 기반 @AOP 애노테이션을 추가하려고하면 의존성부터 추가해야한다.

  * ***Target***
    : 적용이되는 대상  

  * ***advice***
    : 해당 클래스에 소스코드를 변경하지 않고 AOP를 이용한 기능을 추가하는것(적용시킬 내용)  
      부가기능을 담은 구현체 

  * ***JoinPoint***
    : 적용될 위치 (메타적인 정보 - 메소드실행 전,후 에 등등)

  * ***Pointcut***
    : 어디에 적용될것인가? (적용할 메소드()의 내부 실행 전,후,Around)

  ```
  <dependency> 
    <groupId>org.springframework.boot</groupId>    <artifactId>spring-boot-starter-aop</artifactId>
  </dependency>
  ```
  * Aspect 정의 
    * @Aspect
    * 빈으로 등록해야 하니깐( 컴포넌트 스캔을 사용한다면) @Component도 추가.

  * Pointcut 정의
    * @Pointcut(표현식)
    * 주요 표현식
      * execution
      * @annotion
      * bean
    * 포인트컷 조합
      * &&, || , !

  * advice 정의
    * @Before
    * @AfterReturring
    * @AfterThrowing
    * Around

  * 사용방법
  EventService.java , SimpleEventService.java , AppRunner.java를 사용한 이전 예제에서 PerfAspect.java, PerfLogging.java파일이 추가되었다. 

  < PerfAspect.java >
  ```
  @Component
  @Aspect
  public class PerfAspect {

    //advice가 적용되는 대상
    // execution -> pointcut 적용이 가능하다.
    // execution([접근제한자 패턴] 리턴타입패턴 [타입패턴.]이름패턴(파라미터패턴) [throws 예외패턴]) 인것 같은데

    // 자동으로 주소에 의해 등록하지만 해당 패키지의 클래스의 메소를 모두 수행하므로
    // 설정을 안쓰고 싶은 메소드가 있을때는 부적절하다
    // @Around("execution(* me.elyowon..*EventService.*(..))")

    // 직접적으로 메소드에 조건을 줄수있다.
    @Around("@annotation(PerfLogging)")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() -begin);
        return retVal;
    }

    @Before("bean(simpleEventService)")
    public void hell(){
        System.out.println("wellcome to hell !!");
        }
    }
  ```
  스프링부트기반 AOP를 사용하려면 @Aspect를 추가 해주어야하고 @빈등록및 스캔을 위해 @Component등록해야한다.  
  @Around는 강력한 advice중 하나이다. 그리고 @Around("execute(* ~~~)") execute를 사용하여 pointcut을 사용할수있다. 
  Around는 해당 패키지의 클래스의 모든 메소드에 적용된다. 그래서 특정 메소드에만 적용하고 싶을때는 제약이 있다. 하지만 이것도 execute를 사용해서 제약조건을 무마 시킬수있지만 더 좋은 @annotaion(클래스네임) 방법이 존재한다. 

  ```
  //RetentionPolicy.class이상으로 줘야한다.
  // RetentionPolicy는 애노테이션 정보를 얼마나 유지할것인가..?
  // 기본값이 class이다.

  @Documented // ?
  @Target(ElementType.METHOD) // ?
  @Retention(RetentionPolicy.CLASS)
  public @interface PerfLogging {

  }
  ```
  애노테이션 클래스를 만들어서 사용한다. 

  < SimpleEventService.java >
  ```
  @PerfLogging
  @Override
  public void createEvent() {
    .
    .
  }
  ```
  코드상으로 적용되는 모습이다.

  

  * 참고
    * [https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts-designators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts-designators)
