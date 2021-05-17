---
layout: post
title: spring AOP-4(spring AOP)
subtitle: "spring, framework"
categories: spring
tags: framework
comments: true
---
> spring study

# AOP & logback-spring.xml

  - AOP를 사용하여 로그(파일,콘솔) 찍는 연습 및 코드 
  - setry를 이용한 온라인 로그 찍기 

### **logback-spring**
 로그를 파일로 저장하기 위해서는 logback-spring.xml 파일을 만들어 설정을 해줘야한다. 
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOGS_ABSOLUTE_PATH" value="./logs" />
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss}:%-3relative][%thread] %-5level %logger{36} - %msg%n</Pattern>
        </layout>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOGS_ABSOLUTE_PATH}/logback.log</file>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}:%-3relative][%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOGS_ABSOLUTE_PATH}/logback.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>5MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>
    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
    <logger name="com.ddoel.person.demo.config" level="WARN">
        <appender-ref ref="FILE" />
    </logger>
</configuration>
 ```

`이 코드는 거의 복붙하는것이다. 우리가 설정해줘야하는 부분은 <root>, <logger> 부분이다. `

## **AOP 연습**
우리는 validate 그리고 로그를 aop를 이용해서 처리(중복처리) 할수있다.

이는 joinpot, pointcut, advice를 통해서 가능하는데 이는 이전 포스트를 참고하자.

AOP시에 중요한것은 Advice를 설정과 Dto 설정이다. 

validate를 설정하고 이를 판별할때 dto를 사용하지 않는다면 entity의 직접적인 접근이되어 값이 변할 가능성이 존재한다. 그러므로 commonDto,joinReqDto,updateReqDto를 설정하여 관리하는것이 좋다. (user-domain(entity))에서

```
//@Before
//@After
@Around("execution(* com.ddoel.person.demo.web..*Controller.*(..))")
public Object validCheck(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {


    //request 값 처리 못하나요?
    HttpServletRequest request =
            ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
    log.info("주소 : {}",request);
    
    
    String type = proceedingJoinPoint.getSignature().getDeclaringTypeName();
    String method = proceedingJoinPoint.getSignature().getName();

    log.info("type confirm : {}", type);
    log.info("method confirm : {}", method);

    // 아규먼트 리턴
    Object[] args = proceedingJoinPoint.getArgs();

    for (Object arg : args) {
        if(arg instanceof BindingResult){
            BindingResult bindingResult = (BindingResult) arg;


            // 서비스 : 정상적인 화면 -> 사용자요청
            if(bindingResult.hasErrors()){
                Map<String,String> errorMap = new HashMap<>();

                for(FieldError error : bindingResult.getFieldErrors()){
                    errorMap.put(error.getField(),error.getDefaultMessage());
                    log.warn(type+"."+method+"()=>필드 : "+error.getField()+", 메시지:"+error.getDefaultMessage());
                    Sentry.captureMessage(type+"."+method+"()=>필드 : "+error.getField()+", 메시지:"+error.getDefaultMessage());
                }

                return new CommonDto<>(HttpStatus.BAD_REQUEST.value(),errorMap);
            }
        }
    }

    return proceedingJoinPoint.proceed();// 함수의 시택을 실행하라.
}

```

위의 코드는 advice를 설정한 코드이다.

@Around,@Before,@After 등을 통해서 조인포인트를 설정할수있다. 또한 excution을 통해 어떤 메소드를 포인트컷할지 설정도 가능하다.

위의 설정을 끝내고 실질적인 advice (중복 메서드) 가 실행된다.


        

 