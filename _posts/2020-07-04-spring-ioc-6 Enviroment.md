---
layout: post
title: spring Enviroment 
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## IoC 컨테이너 6부 : Enviroment 

Enviroment는 ApplicatinoContext가 가지고 있는 기능중 하나로 Application에 등록되어 있는 여러 환경들을 제어하고 이용할수 있도록 도와주는 기능입니다.

* ApplicationContext - IoC컨테이너가 갖추어야 할 기능들을 정의하고 있는 인터페이스로 다른 추가적인 기능들을 상속받고 있습니다. 그중하나가 EnviromentCapable입니다.

* ### Profile
  Profile이란 환경에 따라 필요한 Bean들이 달라질수 있는데 이것을 쉽게 관리할 수 있도록 도와주는 기능입니다. 예를 들어 Test환경에서 사용할 Bean들과 Develop중 사용할 Bean들 그리고 release에서 사용될 Bean들이 다른경우 이것을 쉽게 관리 할수있도록 돕습니다.

  ```
   @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
        System.out.println(Arrays.toString(environment.getActiveProfiles()));
        System.out.println(Arrays.toString(environment.getDefaultProfiles()));
    }
  ```
  getActiveProfiles() 메소드는 현재 활성화된 Profile을 return해주는 메소드입니다. 활성화된 Profile이 없으면 공백이 출력됩니다. 아무런 값이 나오지 않습니다.  
  이때 Profile이 적용되지 않았으니 어떠한 Bean들도 IoC에 등록이 안되어 있는걸까요? 아닙니다. 바로 Default Profile이란 것이 활성화 되어있습니다.  
  getDefaultProfiles()메소드는 어떤 Profile이 활성화된 상태여도 기본적으로 추가되는 Profile이 있다는 의미입니다. DefaultProfile에 들어가는 Bean들은 @Profile 설정이 없는 Bean들이 들어가게됩니다.  
  ex) @Service, @Component, @Repository 등등.. 

  * 프로파일 사용방법
    * Edit Configuration을 통해서 profile설정방법
    * Bean Configuration class에서 설정하는 방법
   
   ```
   @Configuration
   @Profile("test")
   public class TestConfiguration{
     @Bean
     public BookRepository bookRepository(){
       return new TestBookRepository();
     }
   }
   ```
   ⬆ Bean Configuration class에서 설정하는 방법

   ```
   Edit Configuration >> VM options >>  -Dspring.profiles.active="test"
   ```
   ⬆ Edit Configuration 방법

* ### Property
  Property란 Application의 외부에서 입력한 정보를 이용해 설정값을 변경하는 방법들을 제공하는 기능입니다. 외부에서 제공되는 정보들은 우선순위를 가지고 있고 다음과 같은것들이 존재합니다.

  우선순위 
  - ServletConfig 매개변수
  - SErvletContext 매개변수
  - JNDI(java:com/env/)
  - JVM 시스템 프로퍼티(-Dkey="value")
  - JVM 시스템 환경변수(운영체제 환경변수)

  xml(application.properties) 파일에 기술하고 이것을 enviroment를 통해 가져와 설정하므로써 java코드의 수정없이 설정정보를 변경할수있습니다.

  * Environmention Property가 추가되는 과정
    1. 우선 Context의 getEnvironment()메소드를 통해 ConfigurableEnvironment를 가져옵니다.(이때 곡 ConfigurableEnvironment를 가져와야 MutablePropertySource를 가져올수있습니다.)
    2. 그후 ConfigurableEnvironment의 getPropertySources()메소드를 이용해 MuablePropertySources를 업어옵니다.
    3. MutablePropertySources에 존재하는 add*()메소드들에 매개변수로 PropertySource를 다시 PropertySource의 배개변수로 외부파일을 전달하여 Property들을 추가합니다. 이때 각가의 값들은 Propertysource Type으로 MutablePropertySources안에 존재하는 propertySourceList에 추가됩니다.
  
  
   ### 자세히 살펴보기
   
   * ConfigurableEnvironment
    ```
    public interface Environment extends PropertyResolver {
      String [] getActiveProfiles();

      String [] getDefaultProfiles();

      boolean acceptProfiles(Profiles profiles);
    }
    ```
    ⬆ 위에서 보았단 getActiveProfiles(),getDefaultProfiles()메소드가 보입니다. 하지만 Property추가를 위한 메소드는 보이지 않습니다.
    상속받은 PropertyResolver를 살펴보자

    ```
    public interface PropertyResolver {
      boolean containsProperty(String var1);

      @Nullable
      String getProperty(String var1);

      String getProperty(String var1, String var2);

      @Nullable
      <T> T getProperty(String var1, Class<T> var2);

      <T> T getProperty(String var1, Class<T> var2, T var3);

      String getRequiredProperty(String var1) throws IllegalStateException;

      <T> T getRequiredProperty(String var1, Class<T> var2) throws IllegalStateException;

      String resolvePlaceholders(String var1);

      String resolveRequiredPlaceholders(String var1) throws IllegalArgumentException;
    }
    ```
    getProperty() 등의 메소드들이 보입니다. Environment에서 Property들을 가져올 수 있었던 이유였습니다. Environment에서 Property를 추가하는 메소드들은 보이지 않습니다. 그러므로 Envrionment를 상속하는 하위 인터페이스 중에 찾아봅니다.

    ConfigurableEnvironment, ConfigurableReactiveWebEvironment, ConfigurableWebEnvironment 등 인터페이스가 Environment를 상속받습니다.  
    ConfigurableEnvirionment부터 또 찾아봅니다.

    ```
    public interface ConfigurableEnvironment extends Environment, ConfigurablePropertyResolver {
    void setActiveProfiles(String... var1);

    void addActiveProfile(String var1);

    void setDefaultProfiles(String... var1);

    MutablePropertySources getPropertySources();

    Map<String, Object> getSystemProperties();

    Map<String, Object> getSystemEnvironment();

    void merge(ConfigurableEnvironment var1); 
    }
    ```
    Evironment에서 보였던 메소드가 보이며 오버로드 된것을 알수있다.  getPropertySource()메소드는 프로퍼티자원에대한 메소드같으며 SystemProperties()는 시스템관련 일것이고 merge는 ConfigurableEnvironment 타입을 연결(머지) 해주는거같다고 생각된다.  
    Source부터 확인해보쟈
    
    ```
    public class MutablePropertySources implements PropertySources {
    private final List<PropertySource<?>> propertySourceList;

    @Nullable
    public PropertySource<?> get(String name) {
        int index = this.propertySourceList.indexOf(PropertySource.named(name));
        return index != -1 ? (PropertySource)this.propertySourceList.get(index) : null;
    }

    public void addFirst(PropertySource<?> propertySource) {
        this.removeIfPresent(propertySource);
        this.propertySourceList.add(0, propertySource);
    }

    public void addLast(PropertySource<?> propertySource) {
        this.removeIfPresent(propertySource);
        this.propertySourceList.add(propertySource);
    }

    public void addBefore(String relativePropertySourceName, PropertySource<?> propertySource) {
        this.assertLegalRelativeAddition(relativePropertySourceName, propertySource);
        this.removeIfPresent(propertySource);
        int index = this.assertPresentAndGetIndex(relativePropertySourceName);
        this.addAtIndex(index, propertySource);
    }

    public void addAfter(String relativePropertySourceName, PropertySource<?> propertySource) {
        this.assertLegalRelativeAddition(relativePropertySourceName, propertySource);
        this.removeIfPresent(propertySource);
        int index = this.assertPresentAndGetIndex(relativePropertySourceName);
        this.addAtIndex(index + 1, propertySource);
    }
    .  
    .  
    .  
    }
    ```
    PropertySources 인터페이스 받고 이것은 PropertySource 타입의 리스트를 반환한다.  
    PropertySources 타입의 리스트를 추가,삭제 하는 add*()가 존재함을 알수있고 PropertySources 타입의 내용이 PropertySourcesList에 추가되는것 같다.

    > MutablePropertySources에 존재하는 add*()메소드들에 매개변수로 PropertySource를 다시 PropertySource의 매개변수로 외부파일을 전달하여 Property들을 추가합니다. 이때 각가의 값들은 Propertysource Type으로 MutablePropertySources안에 존재하는 propertySourceList에 추가됩니다.
  
  * 1. Environment에 외부파일을 이용해 Property추가하기
    resources 파일의 하위목록으로 app.properties생성한다. app.name, app.pw 를 설정한다.
    
   
    ```
    @Component
    public class AppRunner implements ApplicationRunner{
      @Autowired
      ApplicationContext Context;

      @Override
      public void run(ApplicationArguments args) throws Exception{
        ConfigurableEnvironment env = (ConfigurableEnvironment) context.getEnvironment();
        MutablePropertySources propertySources = env.getPropertySources();
        propertySources.addLast(new ResourcePropertySource("classpath:app-properties"));
        System.out.println(env.getProperty("app.name"));
        System.out.println(env.getProperty("app.pw"));
      }
    }

    결과
    "elyo9381"
    "1234"
    ```
   
  * 2. @PropertySource를 이용한 방법(애노테이션) 
    ```
    @Component
    @PropertySource("외부파일주소")
    public class AppRunner implements ApplicationRunner{
      @Autowired
      ApplicationContext Context;

      @Override
      public void run(ApplicationArguments args) throws Exception{
        Environment env =  context.getEnvironment();
        System.out.println(env.getProperty("app.name"));
        System.out.println(env.getProperty("app.pw"));
      }
    }

    결과
    "elyo9381"
    "1234"
    ```

  * 3. @Value 애노테이션 사용하기
    application.properties 파일에 밸류값넣기 
    ```
    @Value("${app.name}") String name;
    public void valueTest(){
      Systemo.out.println(name);
    }
    ```
    @Value("${key}") String 변수명; key갑은 위의 application.properties에서 기입한 값에서 '='을 기준으로 앞에 있는 값을 의미합니다.

  * 4. JVM
    JVM의 옵션을 이용해 값을 Environment에 전달할 수 도 있습니다. 
    Vm options  :  -Dapp.name= galid1234

    ```
    @Component
    public class AppRunner implements ApplicationRunner{
      @Autowired
      ApplicationContext Context;

      @Override
      public void run(ApplicationArguments args) throws Exception{
        Environment env =  context.getEnvironment();
        System.out.println(env.getProperty("app.name"));
      }
    }
    ```
    어노테이션등이 필요하지 않다. 
    

    
     



  

  




  



