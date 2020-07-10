---
layout: post
title: ApplicationContext와 다양한 빈 설정방법
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## IoC 컨테이너 2부 : ApplicationContext와 다양한 빈 설정 방법

* # 스프링 IoC 컨테이너의 역할
  * 빈 인스턴스 생성
    ```
    @Bean
    public BookRepository bookRepository(){
        return new BookRepository();
    }    
    @Bean
    public BookService bookService(BookRepository bookRepository){
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository);
      //bookService.setBookRepository(bookRepository());
        return bookService;
    ```
    > bookService가 빈 인스턴스가 되고 bookRepository가 IoC가 된다. 
  * 의존 관계 설정
    > 의존 관계 설정은 scope태그에서 prototype, request는 request당, session는 session당, 기본은 싱글톤(하나)이다.
  * 빈 제공
    
    
* # Bean setting
   * 빈 명세서
    ~~~
    // 빈을 xml로 명시하는 방법
    <bean id="bookService" class ="me.elyowon.springapplicationcontext.BookService">
        <property name="bookRepository" ref="bookRepository"/>
    </bean>
    <bean id="bookRepository"
          class = "me.elyowon.springapplicationcontext.BookRepository"/>

    // componentScan으로 애노테이션붙여서 스캔하는 방법
    <context:component-scan base-package="me.elyowon.springapplicationcontext"/>
      
    ----------------  

    // 빈설정파일을 사용하여 ApplicationconText 만들어서 사용하면 됩니다.
    // ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");

    // xml을 사용하지 않고 자바 파일에서 빈을 명시하는 방법
    // Applicationconfig.class를 이용한 빈설정

    ApplicationContext context = 
        new AnnotationConfigApplicationContext(ApplicationConfig.class);  

    ---------------main.java------------

    @Bean
    public BookRepository bookRepository(){
        return new BookRepository();
    }
    @Bean
    public BookService bookService(BookRepository bookRepository){
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository);
    //  bookService.setBookRepository(bookRepository());
        return bookService;
    ---------------ApplicationConfig.java------------  

    @Service //@Configuration을 설정하였기때문에 주석처리함
    public class BookService {

    @Autowired //@Configuratio을 설정하였기때문에 주석처리함
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
        }
    }
    ---------------BookService.java------------    
    @Repository //@Configuration을 설정하였기때문에 주석처리함
    public class BookRepository {

    }
    ---------------BookRepository.java-----------
      ~~~

    > 빈 정의를 이루는 클래스에 겟터,생성자 등등을 만들어줘야한다. 또한 ComponentScan을 사용하기 위해서는 빈정의 클래스에 어노태이션을 붙여서 스캔 가능함을 알려줘야한다. 
    ApplicationConfig를 사용할땐 클래스내부에 xml에서 설정하는 것을 대신하여 @Bean을 설정해줘야한다.

   * 빈에 대한 정의를 담고 있다.
     * 이름
     * 클래스
     * 스코프
     * 생성자
     * 프로퍼트


* # 컴포넌트 스캔
   * 설정방법
     * XML 설정에서는 context:conponent-scan
     * 자바 설정에서는 @ComponentScan
   * 특정 패키지 이하의 모든 클래스중에 @Component 애노테이션을 사용한 클래스를 빈으로 자동으로 등록해줌.
  

