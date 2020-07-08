---
layout: post
title: Component와 ComponentScan
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## IoC 컨테이너 4부 : @Component와 ComponentScan

* # ComponentScan주요기능
  
  * 컴포넌트의 하위 빈
    - @Repository
    - @Service
    - @controller
    - @configuration

  * componentScan의 주요기능
    - 스캔위치를 어디서부터 어디까지 할것이냐?
    - 필터 : 어떤 애노테이션을 스캔할지 또는 하지 않을지 정한다.

  ```
  // 두개의 애노테이션에 ComponentScan이 존재하는데
  // 이때 springboot의 설정은 덮어쓰게 되고
  // ComponentScan을 사용한다. pkg를 지정하면 pkg에 존재하는 모든 bean을 등록한다.  
  @SpringBootApplication
  @componentScan(package = "me.elyowon")
  public class Demospring51Application {
  
    @Autowired
    MyService myService;

    @Autowired
    BookService bookService;
  }
  ```

  * ComponentScan의 단점
    - 싱글톤의 ComponentScan이 많을때는 초기에 생성한다.
    - 빈의 등록이 많을때는 구동(시간)성능을 잡아먹는다.(프록시)
    - 이를 예방 하기위해서는 다른방법을 사용할수있다 (펑션을 사용한 빈등록이다.)  

  ```
  @Autowired
    MyService myService;

    public static void main(String[] args) {
        //인스턴스를 만들어서 사용하는 방법
        var app = new SpringApplication(Demospring51Application.class);
        // run 되는 중간에 뭘 하고싶다.
        // 빈을 추가적으로 등록할수있다.
        app.addInitializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
            // 이부분에 조건에 따라 코딩을 할수잇다.

            // 펑션널하게 빈을 등록했다. 애플리케이션 구동시에 성능상의 이점이 있다.
            ctx.registerBean(MyService.class);
            ctx.registerBean(ApplicationRunner.class, () -> args1 -> System.out.println("Functional Bean Definition!!"));
        });
        app.run(args);
    }
  ```
  위의 방법은 add.Initializers(new ApplicationContextInitializer~~~)를 통해서 메소드를 구현하고 람다식으로 소스코드를 줄여준것이다. 



  ### 더 공부할 내용
    * 스캔의 위치설정은 어떻게 하는것인가?
    * BeanFactoryPostProcessor란?
    * ConfiguraionClassPostProcessor란? 

  

  

