---
layout: post
title: jpa-concept-8
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study

## 데이터 타입 분류

### 기본값 타입 
  * 엔티티 타입
    * @Entity로 정의하는 객체
    * 데이터가 변해도 식별자로 지속해서 추적 가능
    * 예) 회원 엔티티의 키나 나이값을 변경해도 식별자로 인식 가능

  * 값 타입
    * int, Integer,String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
    * 식별자가 없고 갑만 있으므로 변경시 추적 불가

  * 기본값타입 : int ,string 자바 기본타입
  * 컬렉션 값 타입 : 자바 컬렉션에 기본값타입을 넣은것

  * 기본 값 타입 특징
    * 생명주기를 엔티티의 의존
    * 값 타입은 공유하면 X

### 임베디드 타입
  * 임베디드 타입 : 커스텀한 값타입 (새로운 값타입을 직접 정의할 수 있음)

  ```
    <member.class>
    @Embedded
    private Period workPeriod;

    // 주소
    @Embedded
    private Address homeAddress;

    ---------------------
    <Period.class>
    @Embeddable // 값타입이라고 알려주는 애노테이션
    public class Period {
  ```
  
  * 장점 
    *  재사용이 가능하다  
    *  높은 응집도 
    *  Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음.
    *  임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함

  * 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.
  * 임베디드 타입은 엔티티의 값일 뿐이다. 

  * 컬럼 명이 중복됨( 한 엔티티에서 같은 값 타입을 사용하려면?)
    * @AttributeOverrides를 사용하자 

### 값타입과 불변 객체 
  * 임베디드 타입을 사용할시에 여러가지 튜플이 존재할수있고 이를 하나의 어트리뷰트를 가르킬수있다. 
    객체(jpa)상에서 회원1과 회원2가 같은 Address를 가리키는것!
  * 이때 임베디드 타입은 객체 타입이므로 회원1의 Address 변경시 회원2또한 변경된다. 
    * 객체타입은 주소값을 넘기는 참조값이므로 (값을 복사하는 기본값타입이 아님)
  * 이러한 부작용(side effect)이 발생가능 하고 이를 추적하기 매우 까다로움

  * 그러므로 이를 미연에 방지하는 코드가 좋은 코드이다. 이를 방지하기 위해서는
  * 객체를 불변객체로만드는것과 값(인스턴스)를 복사해서 사용하는것이다. 

  ```
  Address address = new Address("city","street","10000");
  Member member = new Member();
  member.setName("aaa");
  member.setHomeAddress(address);
  em.persist(member);

  Address newAddress = new Address("city","street","10000");
  Member member2 = new Member();
  member2.setName("bbb");
  member2.setHomeAddress(newAddress);
  em.persist(member2);

  //member.getHomeAddress().setCity("newCity");

  ```
  
  위의 코드와 같이 각각의 멤버에 새로운 인스턴스를 설정해서 사이트이팩트를 방지해야한다. 


### 값타입 컬렉션
  * 값 타입을 하나 이상 저장할 때 사용
  * @ElementCollection, @CollectionTable사용
  * 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다. 
  * 컬렉션을 저장하기 위한 별도의 테이블이 필요함
  
  * 값타입 컬렉션의 제약사항
    * 값타입은 엔티티와 다르게 식별자 개념이 없다. 
    * 값은 변경하면 추적이 어렵다.
    * 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제 하고, 값타입 컬렉션에 있는 현재 값을 모두 다시 저장한다. 

  * 값타입 컬렉션 대안
    * 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려 