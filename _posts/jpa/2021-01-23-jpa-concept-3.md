---
layout: post
title: jpa-concept-3
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study

## 앤티티 매핑

 * 객체와 테이블 매핑 : @Entity, @Table
 * 필드와 컬럼 매핑 : @Column
 * 기본키 매핑 : @Id
 * 연관관계 매핑 : @ManyToOne, @JoinColumn


### 객체와 테이블 매핑
   * @Entity 
    * jpa가 관리하는 엔티티
      * 기본 생성자 필수
      * final 클래스, enum,interface, inner 클래스 사용 x
      * 저장할 필드에 final 사용 x

   * 데이터베이스 스키마 자동생성
     * DDL을 애플리케이션 실행 시점에 자동생섯ㅇ
     * 테이블 중심 -> 객체중심
     * 개발하다 sql가서 DDL 실행할 일이없어진다. 
     * 스키마 자동생성은 개발단계에서만 사용해야한다. 
     * persistence.xml 에서 ```<property name="hibernate.hbm2ddl.auto" value="create" /> ``` 옵션을 키면 된다. 
     * 제약조건도 설정가능

   * 필드와 컬럼 매핑
     * 매핑 어노테이션
       * @Column : 컬럼매핑
         * name : 필드와 매핑할 테이블의 칼럼 이름
         * insertable, updatable : 등록 ,변경 가능 여부
         * nullable : false라고 할시 not null의 의미이다. 
         * unique : 중복을 방지함 <하지만 잘쓰지 않는다. 이름이 자동생성 이므로>
         * columnDefinition : DB컬럼 정보를 직접줄수있다.ex) varchar(100) default 'EMPTY'
         * length : 문자길이 제약조건, string 타입에만 사용한다.
         * precision,scale : BigEdcimal 타입에서 사용한다. precision은 소수점을 포함한 전체 자릿수를 ,scale은 소수의 자릿수다. 
       * @Enumerated : enum type
         * EnumType.ORDINAL : enum순서를 DB에 저장 << 왠만하면 쓰지마라
         * EnumType.STRING : enum 이름을 DB에 저장
       * @Temporal : 시간 
       * @Lob : varchar를 넘어선 대용량의 데이터
       * Transient : 메모리에 임시로 실행할때 without DB

   * 기본키 매핑
     * identity 
       * identity 전략에서 DB에 들어가봐야 id값을 알수있다. 
       * 영속컨테스트를 사용하기 위해서는 Id값이 있어야한다. 
       * indentity 전략에서만 .persist(entity)에서 insert쿼리를 날린다. commit()에서가 아니라 why?? -> 영속 컨테스트 사용하기 위해서 
     * SEQUENCE
       ```
         @Entity
         @SequenceGenerator(
         name = “MEMBER_SEQ_GENERATOR",
         sequenceName = “MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
         initialValue = 1, allocationSize = 1)
         public class Member {
         @Id
         @GeneratedValue(strategy = GenerationType.SEQUENCE,generator = "MEMBER_SEQ_GENERATOR")
         private Long id
       ```   
       위와같은 코드에서 시퀀스를 설정 할 수 있다.
       * initialValue : DDL을 생성할 때 처음 1 시작하는 수를 지정한다. 
       * allocationSize : 시퀀스 한번 호출에 증가하는수(성능 최적화에 사용됨) 

   * 권장하는 식별자 전략
     * 기본키 제약조건 : null아님, 유일, 변하면 안된다. 
     * 권장 : Long형 + 대체키 + 키 생성 전략 사용(시퀀스등)


 * 엔티티설계와 매핑
   * UML을 참고로 jpa 매핑을 진행한다.
   * 하지만 DB(UML)는 데이터(테이블)중심 설계이므로 객체 그래프 탐색이 불가능하다.
   * 이는 객체적인 코딩이 안되는것이며 a테이블에서 b테이블의 관계를 설정시 많은 타입과 설정이 필요하다는 것임 => 객체스럽지 않음

  ```
   @Entity
   @Table(name = "ORDERS") // ORDER는 DB에서 예약어로 걸린상황이 많음
   public class Order {

      @Id @GeneratedValue()
      @Column(name = "ORDER_ID")
      private Long id;

      @Column(name = "MEMBER_ID")
      private Long memberId;

      private LocalDateTime orderDate;

      @Enumerated(EnumType.STRING)
      private OrderStatus status;

      public Long getId() {
         return id;
      }
      .
      .

  ```   
  Order table의 예시

  ```
   @Entity
   public class OrderItem {

      @Id @GeneratedValue()
      @Column(name = "ORDER_ITEM_ID")
      private Long id;

      @Column(name = "ORDER_ID")
      private Long orderId;

      @Column(name = "ITEM_ID")
      private Long itemId;

      private int orderPrice;
      private int count;

      public Long getId() {
         return id;
      }
      .
      .

  ```   
  OrderItem table의 예시 