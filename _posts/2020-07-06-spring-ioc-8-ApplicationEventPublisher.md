---
layout: post
title: spring ApplicationEventPublisher
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## IoC 컨테이너 8부 : ApplicationEventPublisher

* ### ApplicationEventPublisher 

  4.2 이전의 이벤트 발생하는 과정은 

  ```
  public static void main(String[] args) {
      SpringApplication.run(Demospring51Application.class, args);
  }
  ```
  애플리케이션이 실행되는 메인에서 AppRunner를 실행하면

  ```
  @Component
  public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext applicationContext;

    @Autowired
    ApplicationEventPublisher publishEvent;
    @Override
    public void run(ApplicationArguments args) throws Exception {
        publishEvent.publishEvent(new MyEvent(this, 100));
    }
  }
  ```
  AppRunner는 빈으로 등록되고 ApplicationRunner 인터페이스를 상속받아 ApplicationEventPublisher타입의 인스턴스를 가지고 MyEvent를 실행해야한다.

  ```
  @Component
  public class MyEvenHandler implements ApplicationListener<MyEvent> {


    @Override
    public void onApplicationEvent(MyEvent myEvent) {
        System.out.println("이벤트 받았다. 데이터는 "  + myEvent.getData());
    }
  }
  ```
  그러면 빈으로 등록된 ApplicationListener인터페이스를 상속받는 MyEventHandler가 이벤트를 캐치해서 화면에 뿌려주는 역할을 한다. 

  ```
  public class MyEvent extends ApplicationEvent {

    private int data;
    public MyEvent(Object source) {
        super(source);
    }

    public MyEvent(Object source, int data){
        super(source);
        this.data = data;
    }
    public int getData(){
        return data;
    }
  }
  ```
  ⬆︎ <이벤트 클래스>
* ### 하지만 4.2 부터는 클래스상속 No

  * 우선 event클래스부터 어떻게 바뀌는지 알아보겠습니다.
  ```
  public class MyEvent {
        private int data;
        private Object source;

        public MyEvent(Object source, int data){
            this.source = source;
            this.data = data;
        }
        public Object getSource(){
            return source;
        }
        public int getData(){
            return data;
        }
  }
  ```
  상속이 사라지고 super를 통해서 Object에 값을주고 불러오는것을 getter와 메소드를 통해서 가능하게 되었다. 
  

  event클래스에 따른 handler또한 변하게 되었다.
  ```
  @Component
  public class MyEvenHandler  {
  //    @Order(Ordered.HIGHEST_PRECEDENCE+2)
      @EventListener
      @Async
      public void handle(MyEvent myEvent) {
          System.out.println(Thread.currentThread().toString());
          System.out.println("이벤트 받았다. 데이터는 "  + myEvent.getData());
      }

      @EventListener
      @Async
      public void handle(ContextRefreshedEvent event) {
          System.out.println(Thread.currentThread().toString());
          System.out.println("ContextRefreshedEvent");
  //         ApplicationContext applicationContext = event.getApplicationContext()
      }

      @EventListener
      @Async
      public void handle(ContextClosedEvent event) {
          System.out.println(Thread.currentThread().toString());
          System.out.println("ContextClosedEvent");
  //         ApplicationContext applicationContext = event.getApplicationContext()
      }
  }
  ```
  event클래스와 마찬가지로 상속이 사라지고 오버라이딩을 통한 특정 메소드가아닌 일반적인 임의의 메소드를 사용할수있게 되었다.  
  동기적으로 실행되게 할때(순서-우선순위) @Ordered(Orederd.HIGHEST_PRECEDENCE)를 통해서 가능하고  
  비동기적인 이벤트를 주려고 하면 @Async를 사용해서 할수있다. 하지만 @Async애노테이션만 붙인다고 해서 비동기적으로 실행되는건 아니다. 
  main문에서 @EnableAsync 애노테이션을 붙이고 Thread관련 설정을 더해야한다. 

  ```
  @SpringBootApplication
  @EnableAsync
  public class Demospring51Application {
      public static void main(String[] args) {
          SpringApplication.run(Demospring51Application.class, args);
      }
  }
  ```

  * #### 애플리케이션 관리 이벤트 
    스프링이 제공하는 기본 이벤트  
    ● ContextRefreshedEvent: ApplicationContext를 초기화 했거나 리프래시 했을 때 발생.  
    ● ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작신호를 받은 시점에 발생.  
    ● ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지신호를 받은 시점에 발생.  
    ● ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에발생.  
    ● RequestHandledEvent: HTTP 요청을 처리했을 때 발생.