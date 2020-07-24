---
layout: post
title: spring @Autowired의 이해 및 예제
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## IoC 컨테이너 3부 : @Autowire

필요한 의존객체 "타입"에 해당하는 빈을 찾아 주입힌다.

* # @Autowired
    @Autowired(required = true)  
    기본값을 true이로 되어있고 true일때 못찾으면 애플리케이션 구동 실패

    Autowired를 사용할수있는 위치
    - 생성자
       @Autowired     
       public BookService(BookRepository bookRepository) {   
       this.bookRepository = bookRepository;
       }   
    - 세터 
       @Autowired(required = false)   
       public void setBookRepository(BookRepository bookRepository) {   
       this.bookRepository = bookRepository;
       }   
    - 필드
       @Autowired(required = false)   
       BookRepository bookRepository;   


    ```
    <의존성이 한개이거나 없거나 하는경우>
    // Field에도 붙일수있다.
    @Autowired(required = false)
    BookRepository bookRepository;

    //생성자에 의존성 주입은 다른 빈이었던 BookRepositoty를 못찾아서 실패했구나 알수있다.

    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    // 셋터에 의존성 주입
    // 북서비스의 자체의 인스턴스는 만들수가 있다. 하지만 실패한다
    // 왜 ??  @Autowired가 존재하기 때문에 의존성 주입을 시도한다. << 이과정이 실패한다.
    //이러한 @Autowired의 의존성 주입이 옵션이고 반드시 필요없다면 required = flase 할것
    @Autowired(required = false)
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    ```    
    위의코드는 의존 객체없이도 required = false를 통해서 빈등록이 가능하다.  
    그리고 의존 객체가 하나일때는 Autowired를 통해서 의존객체 빈등록이 가능하다.

    ```   
    //굳이 추천한다면 Primary를 추천한다
    //@Primary는 등록하고자 하는 의존객체빈에 애노테이션을 등록한다.
    //<의존성이 여러개일 경우 Repository가 여러개>
       // 임의로 빈을 지정하는 경우
       @Autowired @Qualifier("elyoBookRepository")
       BookRepository bookRepository;
       public void printBookRepository(){
              System.out.println(bookRepository.getClass());
       }

       // 빈 여러개를 모두 주입하는 경우
       @Autowired
       List<BookRepository> bookRepositories;

       public void printBookRepository(){
              this.bookRepositories.forEach(System.out::println);
       }
    ```

    ```
    @Autowired
       BookRepository myBookRepository;

       public void printBookRepository(){
              System.out.println(myBookRepository.getClass());
       }
    ```
    의존객체의 주입받고 싶을때 빈의 이름과 타입이름을 같게하면 동일한 빈의 이름을 사용한다. (추천하지는 않는다.)

    * 위의 빈의 이름을과 타입이름을 동일하게 하는 방법 동작원리 

     빈이 만들어진 다음에 해야할일을 적을수도있다.
     뭔가 부가적인 작업을 하는것이다. 두가지 방법이 존재한다. 
     @PostConstruct 방법을 사용할때는 run가 필요하지 않으므로 
     BookServiceRunner 클래스를 삭제해야한다. 
     ```
     @Autowired
       BookRepository myBookRepository;

     @PostConsturct
     public void setUp(){
         System.out.println(myBookRepository.getClass());
     }

       -----------------------
      @Service
      public class BookService implements InitializingBean {
       
       @Autowired
       BookRepository myBookRepository;

       public void printBookRepository(){
              System.out.println(myBookRepository.getClass());
       }

       @Override
       public void afterProperiesSet() throws Excetion {

       }
      }
     ```
     - BeanPostProcessor    
       - 새로 만든 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
     - AutowiredAnnotaionBeanPostProcessor extends BeanPostProcessor
       - 스프링이 제공하는 @Autowired와 @Value 애노테이션 그리고 JSR-330의 @Inject 애노테이션을 지원하는 애노테이션 처리기.

    ```
    @Commponet
    public class MyRunner implements ApplicationRunner{
        @Autowired
        ApplicationContext applicationContext;

        @Override
        public void run(ApplicationArguments args) throws Exception{
            AutowiredAnnotationBeanPostProcessor bean = applicationContext.getBean(AutowiredAnnotationBeanPostProcessor.class);
            System.out.println(bean);
        }
    }
    ```
    AutowiredAnnotationBeanPostProcessor을 사용하여서 빈이 만들어진 다음 부가적인 기능을 명시하는도록 하는 방법이다. 

    PostConsturct 와 AutowiredAnnotationBeanPostProcessor 을 이용하여서 빈이 만들어진 다음 부가적인 기능을 사용하는 방법은 

    스프링 실행도중에 위의 코드가 실행되고 컴파일이되어 앱이 실행된다. 
    (앱이 실행되고나서 부가적인 기능이 사용되는것이 아니다. )

