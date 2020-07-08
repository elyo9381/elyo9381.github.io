---
layout: post
title: spring ConverterANDFormatter
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## 데이터 바인딩 추상화: Converter&Fomatter

* ### 데이터 바인딩 추상화: Converter&Fomatter

  스프링 3.0이후 버전부터는 PropertyEditor를 대신하여 Converter 와 Fomatter를 사용한다.

* Converter
    * S 타입을 T타입으로 변환 할 수 있는 매우 일반적인 변환기
    * 상태정보 없음 == Statless == 쓰레드세이프
    * ConverterRegistry에 등록해서 사용
  
   < EventConverter.java >

   ```
   public class EventConverter {
   //상태정보가 없기때문에 빈으로 등록해서 써도된다.
    @Component
    public static class StringToEventConveter implements Converter<String,Event> {
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConveter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
     }
   }
   ```
    위와같이 PropertyEditor를 대신해서 Converter를 이용해서 변환이 가능하다. 
    간단하게 convert()메소드를 통해서 다양한 타입을 원하는 타입으로 바꿀수 있다.  
    등록하기 위해서 ConvertRegistry interface를 등록해서 사용해야한다. 하지만 이것을 사용하지는 않는다.  

    < WebConfig.java >
    ```
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addFormatters(FormatterRegistry registry) {
          registry.addConverter(new EventConverter.StringToEventConveter());
      }
    }
    ```
    이것을 등록하기 위해서는 스프링MVC를 사용한다는 가정하에 WebConfig클래스를 만들어 오버라이드로 addFormatters()를 사용해 registry.addConverter(new EventConverter.StringToEventConverter())등록해서 사용해야한다.  

    우리가 넣어준 Converter가 모든 컨트롤러에서 동작합니다. 컨트롤러에 요청한 1이 컨버터에서 이벤트로 변환이되서 컨트롤러에서 이벤트 타입으로 받을수있는것이다.   
    >기본적으로 Integer.. 등등은 자동으로 등록해준다.   
    아닌것들은 직접 만들어줘야한다.  

* Formatter

    웹쪽은 사용자 입력값이 문자가 들어오고 문자를 내보내는 경우가 많다. 
    메시지 다국화를 이용해서 여러가지 언어를 기반으로 메시지 소스를 사용해서 메시지를 보내야하는 경우도 존재한다.  
    조금 더 웹쪽에 특화되어 있는 인터페이스를 만들어서 제공한다 그것이 바로 formatter이다.

   < EventFormatter.java > 
   ```
    public class EventFormatter implements Formatter<Event> {

    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(text));
    }

    @Override
    public String print(Event event, Locale locale) {
            return event.getId().toString();
        }
    }
   ```
   PropertyEditor, Converter를 대신하여서 Formatter를 사용할수있다.  
   foramtter도한 쓰레드 세이프하게 사용할수있다.

   < WebConfig.java >
   ```
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addFormatters(FormatterRegistry registry) {
          registry.addFormatter(new EventFormatter);
      }
    }
    ```
    formatter를 사용하기 위해서는 registry에 등록해야하는데 boot또는 bean을 등록하지 않고 사용하는 방법은 위와같이 WebConfig클래스를 설정하고 registry.addFormatter를 이용하는 방법이다.

    * ***데이터바인더 대신에 ConversionService가 사용된다.***
    * ***ConversionService은 스프링MVC, 빈설정, SpEL  사용된다,***
    * ***DefaultFormattingConversionService를 사용한다. 이것은 formatter와 converter를 구현할수있다. 또한 ConsersionService를 사용할수도 있다. AppRunner를 만들어서 ConversionSerivce를 찍어보면 WebConversionService가 출력되는데 이는 DefaultFormattingConversionService를 상속받아서 만들 부트에서 제공하는 클래스이다.***

* converter와 formatter의 자동등록  
  converter와 formatter에 빈이 등록되어 있다면 그 빈들을 자동으로 ConversionService가 등록을 해줍니다.
  
  이 방법은 WebConfig.class를 사용하지 않고 스트링부트에서 지원하는것이다. 그리고 ComponentScand이 가능한 Controller 및 Component등와 같은 빈을 설정하면 자동으로 등록하여 준다. 

   @Autowired
   ConversionService conversionService;
   
   빈으로 등록하고 등록 되어 있는 Converter들을 볼수있는 방법은 ***System.out.println(conversionService);*** 를 하게 되면 등록한 컨버터를 볼수있다.  

  *** 추천하는 내용은 formatter를 사용하는 방법이다. ***








