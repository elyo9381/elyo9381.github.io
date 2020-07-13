---
layout: post
title: Bean 의 scope
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# Spring

## IoC 컨테이너 5부 : Bean 의 scope

빈의 스코프에 대해서 생각해본다. 
싱글톤스코프 빈들만 설정해왔다. 

싱글톤 스콥이란 : 애플리케이션 전반에 걸쳐서 해당 빈에 인스턴스가 한개뿐인 것이다.

```
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Single single;

    @Autowired
    Proto proto;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(proto);
        System.out.println(single.getProto());
    }
}
```
Autowired 애노테이션을 통해서 빈으로 등록할수있고 주입할수있다.  
이를 진행하기 위해서는 Single, Proto 클래스에 @Component 를 설정하고 스코프를 지정해줄수있다.

```
@Component
public class Single {

    @Autowired
    private Proto proto;

    int value = 0;

    public Proto getProto() {
        return proto;
    }
}
```
위의 코드는 싱글톤으로 싱글 클래스를 빈으로 등록하기위한 컴포넌트설정이다.  
의존객체 프로토 클래스를 주입받아 사용할것이다. 
```
@Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class Proto {
}
```
위의 코드는 프로토타입으로 프로토 클래스를 빈으로 등록하기위한 컴포넌트설정이고 프록시모드 또한 설정되어 있다. 프록시 모드는 뒤쪽에서 다시 설명하겠다.  
그리고 프로토 클래스는 의존객체로 사용할것이다.  

```
<출력값>
proto
me.elyowon.demospring51.Proto@d277579
me.elyowon.demospring51.Proto@5db6b845
me.elyowon.demospring51.Proto@378f002a
single
me.elyowon.demospring51.Single@1afd72ef
me.elyowon.demospring51.Single@1afd72ef
me.elyowon.demospring51.Single@1afd72ef
proto by single
me.elyowon.demospring51.Proto@2cc75074
me.elyowon.demospring51.Proto@445bb139
me.elyowon.demospring51.Proto@b9a77c8
```
싱글톤과 프로토 타입의 차이점을 보여주는데 싱글톤으로 설정하면 변하지 않는 정적데이터를 가지지만 프로터 타입은 동적인 데이터 변화를 보여준다.  
> 프로토타입은 Request, Session, WebSocket등에서 사용된다. 

* ### 프로토타입 빈이 싱글톤 빈을 참조하면?
  * 아무문제 없음.

* ### 싱글톤 빈이 프로토타입 빈을 참조하면? 
  * 프로토타입 빈이 업데이터가 안된다. 
  * 업데이트 하러면 
    * scoped-proxy
    * Object-Provider
    * Provier(표준)
  ```
  @Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public class Proto {
    }
  ```
  위의 방법을 통해서 빈의 스코프를 다양하게 지정해줄수있다. 여기서 proxyMode = ScopedProxyMode.TARGET_CLASS 는 Proto 클래스를 싱글톤 빈에서 프로토 타입을 참조하게 되면 프로토 타입은 싱글톤으로 지정되게 된다.  
  우리는 싱글톤에서 프로토타입빈을 사용해야하므로 위와같은 프록시를 지정해야한다.


 원래 defalut 설정은 proxMode = ScopedProxMode.DEFAULT 이다.
 디폴트를 프록시를 사용하지 않는다는 옵션이다.
 타켓클래스는 cg라이브러리를 사용한 다이나믹 프록시를 사용한다는것이다.

 @Scope의 옵션에서 proxMode ScopedProxMode.TARGET_CLASS 의 의미는
 프록시를 쓴다는 것은 Proto클래스를 프록시로 감싸라는 의미이다.
 TARGET_CLASS 기반의 프록시로 감싸라 Proto(Bean)을 다른빈들이 사용할때  
 왜? 프록시로 감싸야하는냐 >> 다른인스턴스들이 Proto(Bean)을 참조하면 안되기 때문이다.
 프록시를 거쳐서 참조해야하기 때문이다.
 직접참조한다면 값이 싱글일테니 다이나믹 프록시를 거쳐 Prototype으로 변경하는것이다.

 Proto가 현재 클래스이므로 TARGET_CLASS를 사용한것이다.
 만약에 interfaces라면 interfaces를 썻을것이다. (이것은 jdk기반의 라이브러리다.)

 Proto(Bean)을 감싸는 프록시 인스턴스가 생성이 되는것이다.
 이 프록시빈을 Single(bean)의 Proto(bean)에 주입하는것이다.


싱글톤 객체 사용 시 주의할 점
* 프로퍼티가 공유.
* ApplicationCotext 초기 구동시 인스턴스 생성.
    
```
   @Component
    public class Single {

    @Autowired
    private Proto proto;

    int value = 0;

    public Proto getProto() {
        return proto;
        }
    }
```
프로퍼티가 공유된다.  
만약에 value = 0 이라면 
proto(bean)은 하나의 인스턴스를 가진다.
이럴때 value가 스레스 세잎한다고 할수없다.
스레드가 계속적으로 동일한곳을 보고있기 때문에 공유가 된다.(스레드 세잎하지않다)
모든 싱글톤스코프의 빈들은 기본값이 ApplicationContext를 만들때 인스턴스를 생성하게 되어있다.
그렇기에 애플리케이션 구동할때 시간이 좀더 걸리수가 있다.
