---
layout: post
title: spring AOP-2(프록시기반 AOP)
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## AOP-2(프록시기반 AOP)

* 스프링 AOP 특징
  * 프록시 기반의 AOP 구현체
  * 스프링 빈에만 AOP를 적용할 수 있다.
  * 모든 AOP 기능을 제공하는것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는것이 목적

* 프록시 패턴  
  ***왜? (기존코드 변경없이) 접근제어 또는 부가기능 추가하기위해서***
  
  ![프록시패턴](../assets/img/spring/proxyPatten.png)

  * 기존코드를 건드리지 않고 성능을 측정해 보자.

* 문제점
  * 매범 프록시 클래스를 작성해야하는가?
  * 여러 클래스 여러 메소드에 적용하려면?
  * 객체들 관계도 복잡하고....

* 그래서 등장한것이 스프링 AOP
  * 스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic프록시를 사용하여 여러 복잡한 문제해결.
  * 동적 프록시 : 동적으로 프록시 객체 생성하는 방법
    * 자바가 제공하는 방법은 인터페이스 기반 프록시 생성.
    * CGlib은 클래스 기반 프록시도 지원
  * 스프링 IoC : 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록 시켜준다.
    * 클라이언트 코드 변경없음.
    * AbstractAutoProxyCreator​ implements ​BeanPostProcessor

* 예제 내용
  
  EventService.java , SimpleEventService.java , AppRunner.java 로 구성되어 있는 예제  
  @Primary 는 같은 타입의 애노테이션이 있다면 primary를 우선적으로 빈으로 받는 애노테이션이다.

  < EventService.java >
  ```
  //<<interfacae>>
  //Subject

  public interface EventService {

    void createEvent();

    void publishEvent();

    void deleteEvent();
  }
  ```

  < SimpleEventService.java >
  ```
  //real Subject
  @Service
  public class SimpleEventService implements EventService{

    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Create an event");

    }

    @Override
    public void publishEvent() {
  //        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Published an event");
  //        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
  }
  ```

  < ProxyEventService.java >
  ```
  @Primary
  @Service
  public class ProxySimpleEventService implements EventService{

    @Autowired
    SimpleEventService simpleEventService;

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);

    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
    }
  }
  ```

  < AppRunner.java >
  ```
  @Component
  public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.publishEvent();
    }
  }
  ```

  1. @Componet 통해서 AppRunner가 빈으로 등록되고 run이 실행될수있는 상태로 만드는것입니다. 그리고 @Autowired 통해서 EvnetService를 빈으로 등록하고 eventService.createEvent()실행
  2. EventService 인터페이스를 실행하는 SimpleEventService 클래스는 @Service 를 통해 빈으로 등록되고 createEvent()를 실행합니다. 이때 각메소드에 성능을 체크하는 코드를 추가하고싶은데 ***이것은 코드중복이며 부가기능을 넣기 위해서는 프록시를 사용한다. 
  3. 프록시를 사용하기 위해서 프록시클래스를 지정하고 simpleEventSerivce와 같이 EventService를 인터페이스로 갖는다. 그리고 같은 @Service로 빈으로 등록하고 @Primary를 통해 SimpleEventService가 아닌 ProxySimpleEventService를 빈으로 먼저 사용하게 한다.  그리고 프록시클래스에서 SimpleEventService에 부가기능과 중복되는 코드를 사용하게 하여 Simple에서의 코드 중복을 없애는 것이다. 

  위와같이 Crosscutting Concerns가 존재하며 부가기능 및 접근제어를 조작하기 위해서 프록시 패턴을 사용한다.  
  하지만 위와같은 코드와 방법론은 문제점을 가지고 있습니다.  

  * 매번 프록시 클래스를 작성해야합니다.
  * 여러 클래스 여러 메소드에 적용하기 반복적인 작업이 많이 발생합니다.

  그렇기 때문에 위와 같은 문제점을 해결하기 위해서 동적인 프록시를 만드는 방법이 있습니다.
  동적이라는것은 (runtime)애플리케이션이 동작하는중에 동적으로 어떤 객체에 프록시객체를 만드는 방법이 있습니다. 어떤객체를 감싸는 프록시객체를 런타임에 만드는 방법이 있다.


  

  
