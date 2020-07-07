---
layout: post
title: spring Resource
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## Resource 추상화

* ### Resource

  Resource 추상화는 java.net.URL을 추상화 한것입니다. 

  URL은 인터넷상의 주소를 표현 합니다.  
  URL클래스는 웹상에 존재한는 자원에 접근할 떄 사용하는 클래스입니다.(자바에서)

  * Why?? 추상화를 하였을까?
    * classpath 기준으로 리소스 읽어오는 기능 부재
    * ServletContext를 기준으로 상대 경로로 읽어오는 기능 부재
    * 새로운 핸들러를 등록하여 특별한 UR접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부족하다.  
    
  * Resource는 인터페이스
    * **주요메소드**
      * getInpuStream()
      * exitst()
      * isOpen()
      * getDesciption()
  
  * 구현체
      * UrlResource : java.net.URL참고, 기본으로 지원하는 프로토콜 http,https, ftp,file,jar.
      * classpathResource : 지원하는 접두어 classpath:
      * FileSystemResource
      * ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스 찾는다.

  * 리소스 읽어오기
      * Resource의 타입은 location문자열과 ApplicationContext의 타입에 따라 결정된다.
        * ClassPathXmlApplicationContext -> ClassPathResource
        * FileSystemXmlApplicationContext -> FileSystemResource
        * WebApplicationContext -> ServletContextResource
      * ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL접두어(+ classpath:)중 하나를 사용할 수 있다.
        * classpath:me/elyowon/confing.xml ->ClassPathResource
        * file:///some/resource/path/config.xml -> FileSystemResource

  ```
  @Component
  public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // ~~~.xml 문자열 자체가 Resource 변환이되고 >>> getResource("~~~.xml")로 사용되는것이다 <클래스패스 기준>
        var ctx = new ClassPathXmlApplicationContext("sdfsdf.xml");
        // 파일시스템경로를 기준으로
        var ctx1 = new FileSystemXmlApplicationContext("sfsdf.xml");


        Resource resource = resourceLoader.getResource("classpath:text.txt");
        System.out.println(resource.exists());
        System.out.println(resource.getDescription()); // 풀패키지 경로
        System.out.println(Files.readString(Path.of(resource.getURI())));
        System.out.println(Path.of(resource.getURI())); // string으로 된 파일경로를를 파일시스템이 인지할수있는 경로로 변환하는 메소드
        System.out.println(resource.getURI()); // 파일경로를 문자열로
        // Files.readString << java11의 메소드 파일시스템의패스를 형성해준다.
    }
  } 
  ```
  resource인스턴스를 이용하여 getResource()메소드에서 classpath:text.txt 인자를 통해서 text.txt라는 리소스를 받아올수있는데
  만약에 classpath:가 없이 인자에 text.txt만 존재한다고하면  
  resource.exists()메소드는 정상적으로 실행된다.  
  resource.getDescription()메소드또한 정산적으로 실행된다.  
  Files.readString(Path.of(resource.getURI())) 실행되지 않는다.  
  why? getResource()메소드에서 classPath를 명시적으로 지정하지 않으면 스프링은 ServletContextResource기본으로 설정되어 있어 리소스를 못찾게 된다  
  그러므로 리소스타입을 강제하여서 사용하여야 지정된 위치의 리소스를 불러올 수 있다.

