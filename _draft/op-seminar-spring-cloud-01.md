

## Spring Cloud Stream



### 우선 Spring Cloud 란 ?

- 마이크로서비스 구축을 위해 필요한 모든 라이브러리의 집합



  ![img](https://spring.io/img/homepage/diagram-distributed-systems.svg)


> Building distributed systems doesn't need to be complex and error-prone. Spring Cloud offers a simple and accessible programming model to the most common distributed system patterns, helping developers build resilient, reliable, and coordinated applications. Spring Cloud is built on top of Spring Boot, making it easy for developers to get started and become productive quickly.

- Spring Cloud는 가장 보편화된(범용적인) 분산시스템 패턴에 대해서 단순하고 접근 가능한 프로그래밍 모델을 제공
- 개발자가 복원성(resilient), 신뢰성(reliable), 조정된 어플리케이션(coordinated applications)을 구축할 수 있도록 지원
- Spring Cloud는  '스프링 부트' 기반위에서 구성됨으로써 개발자가 쉽고 빠르게 만들 수 있고, 빠르게 생산성을 높일 수 있음.



### 우리의 관심사 ( 시스템 결합도가 계속 높아짐 ), Spring-cloud는 따로 공부할 것.

- Message brokers (Spring Cloud Stream)

  - Reactor기반 Apache Kafka, RabbitMQ
  - 마이크로서비스(꼭 지금은 마이크로서비스가 아니더라도)간 정보의 교환, 커뮤니케이션을 위한 라이브러리들.



### Spring Cloud Stream 이란 ?

*  마이크로서비스간, 외부시스템간 메세시 통신에 필요한 기술, 또는 메세지 기반 마이크로 서비스를 구현하기 위한 기술

  > A lightweight event-driven microservices framework to quickly build applications that can connect to external systems. Simple declarative model to send and receive messages using Apache Kafka or RabbitMQ between Spring Boot apps.

  *외부시스템과 연결가능한 어플리케이션을 빠르게 생성할 수 있는 경량의 이벤트 기반 마이크로서비스 프레임워크*

 쉽게 얘기하면 메세지 기반 어플리케이션을 구축하기 위해 Rabbit-MQ 나 Apache Kafka 와 같은 메세지 브로커와 쉽게 바인딩 할 수 있는 프레임워크 



#### 주요 어노테이션 

  - `@EnableBinding` 메세지 브로커와의 연결(Binding) 담당
  - `@StreamListener` 메세지브로커의 이벤트 수신 (Input 인터페이스를 통한)
  - `@InboundChannelAdapter` 메세지를 전송받는 채널에 메세지를 내보내기 위한 어댑터 (Outpu 인터페이스를 통한)
  - `@ToSend` 지정한 Topic or Queue 로 메세지 전달



#### 주요 인터페이스 

> - **Sink:** Identifies the contract for the message consumer by providing the destination from which the message is consumed.
> - **Source:** Identifies the contract for the message producer by providing the destination to which the produced message is sent.
> - **Processor:** Encapsulates both the sink and the source contracts by exposing two destinations that allow consumption and production of messages.

![spring cloud stream](https://zuminternet.github.io/images/portal/post/2017-7-11-spring_cloud_stream_rabbitmq_01/20170610_sourcesink.png)



위의 어노테이션 선언만으로 메세지 브로커를 통해 외부메세지를 받기 위한 어플레케이션 생성가능

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class VoteRecordingSinkApplication {

  public static void main(String[] args) {
    SpringApplication.run(VoteRecordingSinkApplication.class, args);
  }

  @StreamListener(Sink.INPUT)
  public void processVote(Vote vote) {
      votingService.recordVote(vote);
  }
}
```



- 스프링부트 기반으로 간편하고 직관적인 어노테이션으로 빠른 개발이 가능

- 개발자는 메세지브로커의 커넥션, 설정등에 신경쓰지 않고 비즈니스 로직에 집중할 수 있음


![Spring Cloud Stream overview](https://cloud.spring.io/spring-cloud-stream/img/SCSt-overview.png)



### 왜 메세지 기반 (Messaga-Driven) 기술이 필요한가?

- 느슨한 결합이 가능하므로 분산시스템, 마이크로서비스간 협업 관계에 있어 상호 의존성을 배제 할 수 있다.
- 상대시스템의 성능, 서버 상태에 전혀 영향을 받지 않는다.
- 서로 시스템 정보를 몰라도 메세지 브로커 시스템간 메세지 교환-상호간 정보 교환-이 가능하다.
- API 방식의 경우, 목적지 시스템의 성능과 서버의 상태에 직접적인 영향을 받기 때문에 예외처리, FallBack처리가 필수
- 상대 시스템은 서로 독립적으로 인스턴스를 증가 감소할 수 있다.
- 상호 정보 교환을 하는 시스템들은 서로가 어디에 위치하는지 (Endpoint가 어디인지), 어떤 기술기반인지 관심을 가질 필요가 없다. 알지 못한다. 오로지 메세지 전송(Pub) 과 수신(Sub)에만 관심을 가지면 된다.



### 메세지 브로커 를 이해하기 위한 주요개념

- 프로토콜 
  - AMQP, STOMP, MTQQ, TCP/IP
- Connection : 발생자와 소비자, Broker 사이의 물리적인 연결
- Channel - 외부에서 메세지 브로커를 통해 메세지를 주고받을 수 있는 채널, 논리적인 연결고리 (발행시스템과 구독시스템은  채널을 통해 Queue에 접근 )
- Exchange - 메세지의 라우팅 규칙(교환자), 실제 메세지를 담는 Queue와 바인딩 됨.
- Queue - 메세지의 최종 목적지 

![request-response](https://zuminternet.github.io/images/portal/post/2017-7-11-spring_cloud_stream_rabbitmq_01/20170610_mq_flow_04.png)



### 운영 플랫폼 시스템(서비스)에 발행자-구독자관계를 대입해 보면

- 발주시스템(ESCM)
  - 발행자(Publisher) -  발주정보, 상품정보를 발행(생산)
  - 구독자(Subscriber)  - 재고정보, 출고정보, 배송정보를 구독(소비)
- 주문처리시스템(ODIS)  
  - 발행자(Publisher) - 주문정보를  발행(생산)
  - 구독자(Subscriber)  - 권역정보, 재고정보를 구독(소비)
- 창고관리 및 생산시스템(WMS)
  - 발행자(Publisher) - 재고정보, 출고정보를 발행(생산) 
  - 구독자(Subscriber)  - 주문정보, 상품정보, 발주정보를 구독(소비)
- 배송 시스템(TMS) 
  - 발행자(Publisher) - 권역정보, 배송정보를 발행(생산)하는 시스템
  - 구독자(Subscriber)  - 주문정보, 출고정보를 구독(소비)



	메세지기반 구조로 바로본 시스템 - 정보간 발행과 구독 관계

- |            | 주문정보 | 상품정보 | 발주정보 | 재고정보 | 출고정보 | 권역정보(주소) | 배송정보 |
  | ---------- | -------- | -------- | -------- | -------- | -------- | -------------- | -------- |
  | eSCM(발주) |          | Pub      | Pub      | Sub      | Sub      |                | Sub      |
  | ODIS(주문) | Pub      |          |          | Sub      |          | Sub            |          |
  | WMS        | Sub      | Sub      | Sub      | Pub      | Pub      |                |          |
  | TMS        | Sub      |          |          |          | Sub      | Pub            | Pub      |



### 현재 레거시에서 바로 메세지 브로커 방식으로 전환 할  수 있는 영역은?

- 우선 OP 시스템 내부간 인터페이스 중에서 선택 (OP운영경험 확보후 외부시스템(CMS,데농)으로 확장)
- 가장 간단하게 해 볼 수 있는, 상기에서 언급한 효과를 얻을 수 있는 인퍼




### 대표적인 메세지브로커 (external message broker)

- Rabbit MQ
  - 가장 범용적이고, 널리 사용되는 오픈소스 메세지 브로커 (35,000개 이상)
  - 강력한 메세지 라우팅 기능, Queue의 다양한 옵션 설정
  - Supports [multiple messaging protocols](https://www.rabbitmq.com/protocols.html), [message queuing](https://www.rabbitmq.com/tutorials/tutorial-two-python.html), [delivery acknowledgement](https://www.rabbitmq.com/reliability.html), [flexible routing to queues](https://www.rabbitmq.com/tutorials/tutorial-four-python.html), [multiple exchange type](https://www.rabbitmq.com/tutorials/amqp-concepts.html).(direct, topic, )
- Amazon SQS / SNS 
  - 메세지 브로커를 설정할 필요없는 단순 대기열(queue)과 주제(topic) 서비스
  - 단순히 Queue용도로만 사용할 경우 적절한 대안이 될 수 있음. 메세지 라우팅 기능, 채널 그룹핑기능 없음.
  - 관련문서: [https://www.baeldung.com/aws-queues-java](https://www.baeldung.com/aws-queues-java)
- Amazon MQ (Apache MQ)
  - Apache ActiveMQ 기반의 클라우드 기반 메세지 브로커
  - 표준 프로토콜 모두 지원 ( 향후 ApacheMQ뿐만이 아니라 다른 Third-Party 추가될 가능성 )
  - 브로커의 생성, 이중화(Active-Standby), 모니터링(CloudWatch)를 AWS기반에서 간단하게 구성할 수 있음 (매우 매력적인 장점)
  - 레퍼런스 문서: [https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html](https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html)
- Apache Kafka 
  - 범용 메세지 브로커중에서는 가장 뛰어난 성능
  - 통신프로토콜이 TCP/IP방식임
  - 메세지 전송보장



> RabbitMQ. Kafka 와 AmazonMq와의 가장 큰 차이점은 
>
> - RabbitMQ. Kafkas는 spring-cloud-stream에서 제공하는 단순화돤 공통 어노테이션을 통해 구현.
> - AmazonMq는 spring-cloud-stream 기반의 바인딩 및 Pub/Sub 구현이 안됨. JMS 방식으로 바인딩 및 메세지교환처리 구현. 즉, 사용하는 어노테이션이 틀림. (추가될 가능성은 있음)
> - 예제를 통해 확인이 가능





------

### RabbitMQ & Apache Kafka의 비교

-  메세지 브로커의 대표 솔루션, 레퍼런스가 가장 많음

- |                 | RabbiMQ                               | Apache Kafka                         |
  | --------------- | ------------------------------------- | ------------------------------------ |
  | 메세지 저장방식 | 파일시스템                            | Memory                               |
  | 통신 프로토콜   | TCP/IP                                | AMQP                                 |
  | 기능            | 타 솔루션대비 뛰어난 메세지 처리 성능 | 다양한 기능, 익스체인지(라우팅) 지원 |
  | 부가기능        | 최소1번의 메세지 전송 보장            | 자체 클러스터링 지원                 |



### AmazonMQ 의 Active -StandBy 구조

![img](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/images/amazon-mq-architecture-active-standby-deployment.png)



### 메세지 브로커의 선택기준

- 내구성, 범용성
  - 메세지의 전송 보장, 재처리, 보관
- 범용성 - 실 서비스에 적용된 많은 레퍼런스, 기술자료, 샘플소스
- 가용성 - 클러스터링 여부(무중단), 안정적인 메세지 처리성능

- 관리의 용이성
  - 시스템의 느슨한 결합에 따라오는 또 다른 관리포인트
  - 최소한의 관리비용



------



### Message Broker 실습 - 1. RabbitMq

1. **Spring Cloud Stream Project 생성**

   1. [SpringInitializar](https://start.spring.io/) 를 이용한 프로젝트 생성

   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-stream-dependencies</artifactId>
               <version>Fishtown.M2</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-stream</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-stream-rabbit</artifactId> <!-- or '*-stream-kafka' -->
       </dependency>
   </dependencies><repositories>
       <repository>
           <id>spring-milestones</id>
           <name>Spring Milestones</name>
           <url>https://repo.spring.io/libs-milestone</url>
           <snapshots>
               <enabled>false</enabled>
           </snapshots>
       </repository>
   </repositories>
   ```

2. **Message Broker 설치 - 우선 RabbitMQ 로...**

   1. **RabbitMQ 설치**

   2. Docker를 이용한 설치. docker-compose 이용 

      1. Docker 없는 경우 Docker for Mac 부터 설치

      `$ docker-compose -f docker-compose-rabbit.yml up -d`





기본 Sink 인터페이스를 참조하는 Subscribe 클래스 생성후 구동시 확인할 수 있는 Rabbit MQ 커넥션 및 채널, 큐 생성 및 바인딩 로그

```
declaring queue for inbound: input.anonymous.0gGUhwSLQa633CoQfuUjgQ, bound to: input
2018-09-07 00:16:58.150  INFO 21476 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [localhost:5672]
2018-09-07 00:16:58.237  INFO 21476 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#6f89292e:0/SimpleConnection@2e8a1ab4 [delegate=amqp://guest@127.0.0.1:5672/, localPort= 51418]
2018-09-07 00:16:58.312  INFO 21476 --- [           main] o.s.c.stream.binder.BinderErrorChannel   : Channel 'application.input.anonymous.0gGUhwSLQa633CoQfuUjgQ.errors' has 1 subscriber(s).
2018-09-07 00:16:58.312  INFO 21476 --- [           main] o.s.c.stream.binder.BinderErrorChannel   : Channel 'application.input.anonymous.0gGUhwSLQa633CoQfuUjgQ.errors' has 2 subscriber(s).
2018-09-07 00:16:58.333  INFO 21476 --- [           main] o.s.i.a.i.AmqpInboundChannelAdapter      : started inbound.input.anonymous.0gGUhwSLQa633CoQfuUjgQ
2018-09-07 00:16:58.335  INFO 21476 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 2147483647
```

기본 Sink 인터페이스를 통해 바인딩을 하는 경우,

- Exchange Name -> input  (Sink인터페이스에 지정된 디폴트 이름)
- Queue -> input.anonymous.0gGUhwSLQa633CoQfuUjgQ  (큐이름을 따로 지정하지 않았기 때문에 임의로 생성)



#### 메세지 브로커 서비스가 정지되는 경우,

- Rabbit MQ 서비스가 내려가 있는 경우,  구독자 시스템은 계속 Rabbit MQ와 커넥션을 시도한다.
- RabbitMQ 서비스가 다시 올라오면 자동으로 바인딩 되고 메세지를 수신한다.

```
2018-09-07 00:23:19.448  INFO 21476 --- [a633CoQfuUjgQ-2] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [localhost:5672]
2018-09-07 00:23:24.529  WARN 21476 --- [a633CoQfuUjgQ-2] o.s.a.r.l.SimpleMessageListenerContainer : Consumer raised exception, processing can restart if the connection factory supports it. Exception summary: org.springframework.amqp.AmqpConnectException: java.net.ConnectException: Connection refused (Connection refused)
2018-09-07 00:23:24.529  INFO 21476 --- [a633CoQfuUjgQ-2] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@5dbb31af: tags=[{}], channel=null, acknowledgeMode=AUTO local queue size=0
```

외부 구독자 시스템은 메세지브로커(Rabbit MQ)가 내려가면,  비즈니스 로직레벨에서 에러가 발생하지 않고 서버로의 재접속을 계속 시도한다. 기존 5초 간격으로 접속을 계속 시도한다.



```
2018-09-07 00:27:28.038  INFO 21476 --- [633CoQfuUjgQ-51] o.s.amqp.rabbit.core.RabbitAdmin         : Auto-declaring a non-durable, auto-delete, or exclusive Queue (input.anonymous.0gGUhwSLQa633CoQfuUjgQ) durable:false, auto-delete:true, exclusive:true. It will be redeclared if the broker stops and is restarted while the connection factory is alive, but all messages will be lost.
```

메세지 브로커가 다시 올라오는 순간, 바로 접속이 되고 메세지를 수신받게 된다. 단, 커넥션 팩토리가 유지되고 있는 상태에서 메세지브로커가 중지되거나 재시작되면 모든 메세지는 사라진다. (Queue의 옵션과 관련) 



#### Message Broker 실습 - 2. AmazonMQ

- 다음 세미나때...

