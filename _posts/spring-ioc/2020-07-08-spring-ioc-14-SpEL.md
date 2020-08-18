---
layout: post
title: spring SpEL(스프링 Expression Language)
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## SpEL(스프링 Expression Language)

* 스프링 EL이란?
  -  객체 그래프를 조회하고 조작하는 기능을 제공한다.
  -  Unified EL과 비슷하지만, 메소드 호출을 지원하며, 문자열 템플릿 기능도 제공한다.
  -  OGNL,MEVEL,JBOss EL등 자바에서 사용할 수 있는 여러 EL이 있지만, SpEL은 모든 스프링 프로젝트 전반에 걸쳐 사용할 EL로 만들었다.
  -  스프링 3.0부터 지원.
* SpEl 구성
  ```
  ExpressionParser parser = new SpelExpressionParser(); 
  StandardEvaluationContext context = new StandardEvaluationContest(bean);
  Expression expression = parser.parseExpression("SpEL 표현식");
  String value = expression.getvalue(context,String.class);
  ```
* 문법
  * #{"표현식"}
  * ${"프로퍼티"}
  * 표현식은 프로퍼티를 가질 수 있지만, 반대는 안됨.
    * #{${my.data} + 1}
  * [레퍼런스 참고](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions)

* 실제로 어디서 쓰나?
  * @Value 애노테이션
  * @ConditionalOnExpression 애노테이션
  * [스프링 시큐리티](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html)
    - 메소드 시큐리티, @PreAuthorize, @PostAuthrorize, @PreFilter, @PostFilter
    - XML 인터셉트 URL설정
  * [스프링 데이터](https://spring.io/blog/2014/07/15/spel-support-in-spring-data-jpa-query-definitions)
    - @Query 애노테이션
  * [Thymeleaf](https://blog.outsider.ne.kr/997)

> 사용 예시  
> 이외에도 여러곳에서 사용가능   
> 더 공부할것!   
```
@Component
public class AppRunner implements ApplicationRunner {

  // @Value("#{}") >> SpEL
  // (스프링 Expression Language)를 사용할수있다.
  // 표현식을 사용할수있다.
  // 표현식안에는 프로퍼티를 사용할수있지만 #{${my.value}}
  // $안에 #을 사용할수는 없다. // not !! ${#{'hello'}}
  @Value("#{1+1}")
  int value;

  @Value("#{'helloworld'}")
  String greeting;

  //#은 표현식 방법
  @Value("#{1 eq 1}")
  boolean trueOrFalse;

  // value만 써도 되긴하지만 #을 쓰면 표현식으로 인식해서 사용한다.
  @Value("hello")
  String hello;

  //프로퍼티를 참고하는 방법
  @Value("${my.value}")
  int myValue;

  // 빈의 값을 가져올수도 잇다.
  @Value("#{sample.data}")
  int sampleData;


  @Override
  public void run(ApplicationArguments args) throws Exception {
      System.out.println("===============");
      System.out.println(value);
      System.out.println(greeting);
      System.out.println(trueOrFalse);
      System.out.println(hello);
      System.out.println(myValue);
      System.out.println(sampleData);

      ExpressionParser parser = new SpelExpressionParser();
      Expression expression = parser.parseExpression("2+100");
      Integer value = expression.getValue(Integer.class);
      System.out.println(value);
  }
}
```
⬆︎ SpEL 사용법 