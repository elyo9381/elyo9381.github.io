---
layout: post
title: spring Vaildation
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## Vaildation 추상화

* ### Vaildation

   ### ***애플리케이션에서 사용하는 객체 검증용 인터페이스***
  
  * 특징
    * 어떠한 계층과도 관계과 없다. -> 모든 계층(웹,서비스,데이터)에서 사용해도 좋다.
    * 구현체중 하나로 JSR-303(Bean Vaildation 1.0)과 JSR-349(Bean Vaildation 1.1)을 지원한다.(LocalVaildatorFactoryBean)
    * DataBinder에 들어가 바인딩 할때 같이 사용되기도 한다.

  * 인터페이스 
    * boolean supports(Class clazz): 어떤 타입의 객체를 검증할때 사용할것인지 결정함
    * void vaildate(Object obj, Errors e): 실제 검증로직을 이안에서 구현
      * 구현할때 VaildationUtils 사용하면 편리함.

  Event객체를 검증하며 검증한 내용을 errors에 입력한다. 
  ```
  public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.equals(aClass);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "notempty","Empty title is not allowed");
    }
  }
  ```
  * 스프링 부트 2.5이상 버전을 사용할 때
    * LocalVaildatorFactoryBean 빈으로 자동등록
    * JSR-380(BeanVaildation 2.0.1) 구현체로 hibernate-vaildator 사용.
    * [https://beanvaildation.org/]https://beanvaildation.org/
  
  ```
  @Component
  public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(validator.getClass());

        Event event = new Event();
        event.setLimit(-1);
        event.setEmail("aaa2");
        Errors errors = new BeanPropertyBindingResult(event, "event");

        validator.validate(event,errors);

        System.out.println(errors.hasErrors());

        errors.getAllErrors().forEach(e ->{
            System.out.println("======error code=======");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });

    }
  }
  ```
  처음에 소개했던 EventValidator 클래스를 만들고 이것을 빈으로 등록하여 사용하였으나   
  2.0.5이상 버전에서는 Validator의 빈등록을 통해서 간단한 검증은 할수있게 되었다. 
  Event Class에 @NotEmpty, @Min @Email등과 같은 애노테이션을 붙여 검증할수있다. 

  ### ***스프링부트 2.3이후버전에서는 Validation이 Validation-starter로 붙어서 실행되지 않으므로 직접적으로 설정해줘야한다.***