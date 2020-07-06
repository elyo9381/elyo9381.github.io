---
layout: post
title: spring MessageSource
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## IoC 컨테이너 6부 : MessageSource

* ### MessageSource 
  IoC 컨테이너의 기능 MessageSource에 대해서 알아보려고 합니다. MessageSource는 국제화(i18n)기능을 제공하는 인터페이스 입니다.  
  즉하나의 메시지에 대해 다국러오 변역을 해주는 기능을 제공합니다. 

  >public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver 

  ApplicationContext Interface가 상속받고 있는 여러 interface중 messageSource를 상속되어 있습니다. 이것에서 메소드를 확인할수있습니다.

* #### 1.messages.* 파일생성
  - messages_ko_KR.properties
  - messages_en.properties
  위와 같은 파일을 만들어줘서 설정해야합니다. 여기서 default~()메소드 또한 있습니다. 그런데 컴퓨터의 설정에따라 default는 달라질수있습니다 

  messages_ko_KR.properties
  ```
  greeting = 안녕, {0}
  //Message     arguments
  ```
  messages_en.properties
  ```
  greeting = hi {0}
  //Message     arguments
  ```


* #### 2.사용

```

  @Autowired
  ApplicationContext applicationContext;
  @Autowired
  MessageSource messageSource;
  @Override
  public void run(ApplicationArguments args) throws Exception {
      while(true){
      System.out.println(messageSource.getMessage("greeting",new String[]{"elyowon"},Locale.KOREA));
      System.out.println(messageSource.getMessage("greeting",new String[]{"ely"},Locale.ENGLISH));
      Thread.sleep(1000);
      }
  }
```
ApplicationContext타입의 인스턴스를 생성해 빈으로 등록하여 사용할수도 있으며 MessageSource타입의 인스턴스를 생성해 빈으로 등록하여 사용할수도있습니다.  
messageSource.getMessage("greeting",new String[]{"elyo"},Locale.KOREA); 등과같은 방법으로 사용하여
첫번째 파람(인자)는 string, arguments, default,Locale 순으로 인자를 전달하면 됩니다. 

* ReloadableResourceBundleMessageSource
  위의 예제를 실행하면서 Thread를 통해서 계속적으로 출력할때 messages.*을 변경하여 실행도중에 변경하는 작업을 수행하였습니다. 
  즉 messages.properties파일이 변경되어도 애플리케이션을 다시 시작할 필요가 업다는 장점이 있습니다.
  이때 사용된 ReloadableResourceBundleMessageSource를 이용하여 진행하였습니다.

  ```
  @SpringBootApplication
public class Demospring51Application {
    public static void main(String[] args) {
        SpringApplication.run(Demospring51Application.class,args);
    }
    @Bean
    public MessageSource messageSource(){
        //        var messageSource = new ReloadableResourceBundleMessageSource();
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(3);
        return messageSource;
    }
}

  ```