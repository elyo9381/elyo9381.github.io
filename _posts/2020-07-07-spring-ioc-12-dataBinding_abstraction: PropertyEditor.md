---
layout: post
title: spring dataBinding
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## 데이터 바인딩 추상화: PropertyEditor

* ### 데이터 바인딩 추상화: PropertyEditor

  데이터 바인딩 추상화  
    * 기술적 관점 : 프로퍼티 값을 타켓 객체에 설정하는 기능
    * 사용자 관점 : 사용자 입력밧을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능.  
  
  WHY? -> 사용자가 입력한 값을 주로 문자열인데, 그값을 객체가 가지고 있는 int,long,Boolean,Data등 심지어 Event,Book 같은 도메인 타입으로도 변환해서 넣어주는 기능이 필요해서 바인딩 필요 

  데이터바인딩을 위해 데이터바인터 인터페이스가 존재한다.

  * ProPertyEditor
    * 스프링 3.0 이전까지 Datainder가 변환 작업 사용하던 인터페이스 
    * 쓰레드-세이프 하지 않음(상태 정보 저장 하고 있음, 따라서 싱글톤 빈으로 등록해서 사용하다가는 서로다은 쓰레드에 공유된다.)
      PropertyEditor구현체는 여러쓰레드에 공유해서 쓰면 안된다. 
    * Object와 String간의 변환만 할 수 있어, 사용 범위가 제한적임(그래도 그런경우가 대부분이기 때문에 잘 사용해 왔음. 조심해서..)

  * 1. 사용방법(Example)
    Event , EventController , EventEditor 를 사용하였습니다.
    Event클래스는 id와 title이 멤버변수로 선언

    ```
    @RestController
    public class EventController {
    // EventEditor를 @Component와 같이 전역빈으로 등록하면 안되고
    // InitBinder를 통해서 일정한 부분에 등록할수있다.
    @InitBinder
    public void init(WebDataBinder webDataBinder){
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event){
        System.out.println(event);
        return event.getId().toString();
    }
    }
    ```
    위의 코드에서 @GetMapping부분은 SpringMVC부분이고 나중에 공부하자  
    위의코드 @GetMapping부분은 Event를 받아와서 id를 리턴해주는 코드같다.  
    @InitBinder 부분은 EventEditor를 전역 빈으로 설정하면 안되므로 지정된 빈을 설정하려는 부분이다. 


    ```
    public class EventEditor extends PropertyEditorSupport {
    @Override
    public String getAsText() {
        Event event = (Event)getValue();
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
      }
    }
    ```
    위의코드는 PropertyEditor를 이용하여 바인딩을 하는코드이고  setAsText()메소드를 통해서 Event로 날아온 부분을 원하는타입으로 바꾸고 setValue()메소드를 통해 객체를 PropertyEditor로 날린다.  
    그리고 getAsText()는 getValue()메서드를 통해서 PropertyEditor가 받은 객체를 가져올수있다.

    ***PropertyEditoSupport를 상속받는 이유는 PropertyEditor는 사용하려면 많은 메소드를 구현해야하는데 Support는 원하는 메소드만 Overriding하여 사용할수있기 때문에***

    하지만 이러한 방법은 편리하지가 않고 쓰레드세이프 하지않기때문에 빈으로 등록하여 사용하기도 어렵다.

  