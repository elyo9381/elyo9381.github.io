1. 샘플소스  - RabbitMq 

   1. 의존성 설정 - pom.xml

      - ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        	<modelVersion>4.0.0</modelVersion>
        
        	<groupId>com.spring</groupId>
        	<artifactId>streamdemo2</artifactId>
        	<version>0.0.1-SNAPSHOT</version>
        	<packaging>jar</packaging>
        
        	<name>streamdemo2</name>
        	<description>Demo project for Spring Boot</description>
        
        	<parent>
        		<groupId>org.springframework.boot</groupId>
        		<artifactId>spring-boot-starter-parent</artifactId>
        		<version>2.0.4.RELEASE</version>
        		<relativePath/> <!-- lookup parent from repository -->
        	</parent>
        
        	<properties>
        		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        		<java.version>1.8</java.version>
        		<spring-cloud.version>Finchley.SR1</spring-cloud.version>
        	</properties>
        
        	<dependencies>
        		<dependency>
        			<groupId>org.springframework.cloud</groupId>
        			<artifactId>spring-cloud-stream</artifactId>
        		</dependency>
        		<dependency>
        			<groupId>org.springframework.cloud</groupId>
        			<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        		</dependency>
        		<dependency>
        			<groupId>org.projectlombok</groupId>
        			<artifactId>lombok</artifactId>
        			<optional>true</optional>
        		</dependency>
        	</dependencies>
        
        	<dependencyManagement>
        		<dependencies>
        			<dependency>
        				<groupId>org.springframework.cloud</groupId>
        				<artifactId>spring-cloud-dependencies</artifactId>
        				<version>${spring-cloud.version}</version>
        				<type>pom</type>
        				<scope>import</scope>
        			</dependency>
        		</dependencies>
        	</dependencyManagement>
        
        	<build>
        		<plugins>
        			<plugin>
        				<groupId>org.springframework.boot</groupId>
        				<artifactId>spring-boot-maven-plugin</artifactId>
        			</plugin>
        		</plugins>
        	</build>
        	
        </project>
        
        ```

   2. 프로젝트 생성

   3. RabbitMq 설치

      - 

   4. Publisher Class

      ```java
      package com.spring.streamdemo2;
      
      
      import org.springframework.cloud.stream.annotation.EnableBinding;
      import org.springframework.cloud.stream.annotation.Output;
      import org.springframework.context.annotation.Bean;
      import org.springframework.integration.annotation.InboundChannelAdapter;
      import org.springframework.integration.annotation.Poller;
      import org.springframework.integration.core.MessageSource;
      import org.springframework.messaging.Message;
      import org.springframework.messaging.MessageChannel;
      import org.springframework.messaging.support.MessageBuilder;
      
      @EnableBinding(PersonSource.class)
      public class Pub {
      
      
          @Bean
          @InboundChannelAdapter(value = PersonSource.OUTPUT1, poller = @Poller(fixedDelay = "5000", maxMessagesPerPoll = "1"))
          public MessageSource<String> pubMessage() {
      
              return new MessageSource<String> ( ) {
                  @Override
                  public Message<String> receive() {
      
                      String value = "{\"name\":\"willis\", \"age\":20}";
                      System.out.println ("===== Publish Message - output1 =====" );
      
                      return MessageBuilder.withPayload ( value ).build ( );
                  }
              };
          }
      
      
          @Bean
          @InboundChannelAdapter(value = PersonSource.OUTPUT2, poller = @Poller(fixedDelay = "5000", maxMessagesPerPoll = "1"))
          public MessageSource<Person> pubPersonInfo() {
      
              return new MessageSource<Person> ( ) {
                  @Override
                  public Message<Person> receive() {
      
                      System.out.println ("===== Publish Message -  output2 =====" );
      
                      Person p1 = new Person ( );
                      p1.setAge ( 50 );
                      p1.setName ( "Brown" );
      
                      return MessageBuilder.withPayload ( p1 ).build ( );
                  }
              };
          }
      
      }
      
      /** Customize Source Interface **/
      interface PersonSource {
          String OUTPUT1 = "person-output1";
          String OUTPUT2 = "person-output2";
      
          @Output(OUTPUT1)
          MessageChannel output();
      
          @Output(OUTPUT2)
          MessageChannel output2();
      
      }
      ```

   5. Consumer Class

      ```java
      package com.spring.streamdemo2;
      
      
      import org.springframework.cloud.stream.annotation.EnableBinding;
      import org.springframework.cloud.stream.annotation.Input;
      import org.springframework.cloud.stream.annotation.StreamListener;
      import org.springframework.messaging.SubscribableChannel;
      
      
      @EnableBinding(PersonSink.class)
      public class Sub {
      
          @StreamListener(PersonSink.INPUT1)
          public void subscribeMessage1(Person person) {
      
              System.out.println ( "===== Subscribe1 Message - INPUT1 =====" );
              System.out.println ( "INPUT1: " + person.toString ( ) );
          }
      
          @StreamListener(PersonSink.INPUT1)
          public void subscribeMessage3(Person person) {
      
              System.out.println ( "===== Subscribe2 Message - INPUT1 =====" );
              System.out.println ( "INPUT1: " + person.toString ( ) );
          }
      
      
      
          @StreamListener(PersonSink.INPUT2)
          public void subscribeMessage2(Person person) {
      
              System.out.println ( "===== Subscribe Message - INPUT2 =====" );
              System.out.println ( "INPUT2: " + person.toString ( ) );
          }
      
      }
      
      
      interface PersonSink {
      
          String INPUT1 = "person-input1";
          String INPUT2 = "person-input2";
      
          @Input(INPUT1)
          SubscribableChannel personInput1();
      
          @Input(INPUT2)
          SubscribableChannel personInput2();
      }
      ```



   6. application.properties

      ```properties
      spring.rabbitmq.host=localhost
      spring.rabbitmq.port=5672
      spring.rabbitmq.username=guest
      spring.rabbitmq.password=guest
      
      # Exchange for subscribe
      spring.cloud.stream.bindings.person-input1.destination=person1
      spring.cloud.stream.bindings.person-input2.destination=person2
      
      # Exchange for publish
      spring.cloud.stream.bindings.person-output1.destination=person1
      spring.cloud.stream.bindings.person-output2.destination=person2
      
      # Queue
      #spring.cloud.stream.bindings.input.group=person-group
      ```

      - 접속정보는 기본설정값
      - 채널의 목적지(Exchange)를 명시해주지 않으면 채널이름으로 Exchange생성됨
      - group속성을 지정해주는 않으면 임의의 큐이름으로 생성됨.
        ![image-20180911005623194](/var/folders/qs/_l_704t13qdb4w_w0tc3bfbc0000gn/T/abnerworks.Typora/image-20180911005623194.png)



**Amazon MQ**

