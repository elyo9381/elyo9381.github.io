---
layout: post
title: jpa-concept-7
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study
## 프록시
  * 즉시로딩과 지연로딩
  * 지연로딩 활용
  * 영속성 전이 : cascade
  * 고아 객체 
  * 영속성 전이 + 고아객체, 생명주기


### 프록시와 연관관계
  * Member를 조회할때 Team도 조회 해야할까?
    member를 통해서 team까지 연계 되는 비즈니스 로직이라면 효율적이나   
    member만 필요한 비즈니스 로직이라면 비효율적이다.   
    => 이를 위해서 jpa에서는 지연로딩과 프록시를 통해서 케어를한다. 

  * 프록시 기초 
    * em.find() : DB를 통해서 실제 엔티티 객체 조회
    * em.getReference() : DB 조회를 미루는 가짜(프록시) 엔티티 객체 조회

    * 프록시의 특징 1
      * 실제 클래스를 상속 받어서 만들어짐
      * 실제 클래스와 겉 모양이 같다.
      * 사용하는 입장에서는 진짜 객체인지 프롯기 객체인지 구분하지 않고 사용하면 됨(이론상)
      * 프록시 객체는 실제 객체의 참조(target)를 보관
      * 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드호출
    
    * 프록시 호출순서
      1. getName()
      2. 초기화 요청( 영속성 컨텍스트)
      3. DB조회
      4. 실제 Entity의 메소드 호출함
      5. 4번의 값을 target.getName()으로 반환함 (실제객체의 참조를 보관)

    * 프록시의 특징(주의)
      * 프록시 객체는 처음 사용할 때 한번만 초기화
      * 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
      * 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함(== 비교실패, instance of 사용)
      * 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
      * 영속성 컨텐스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화 하며 문제 발생(하이버네이트는 org.hibernate.LazyinitializationException 예외를 터트림)
    
    * 프록시 확인
      * 프록시 인스턴스의 초기화 여부 확인 
        * emf.PersistenceUnitUitl.isLoaded(Object entity) 
      * 프록시 클래스 확인 방법
        * entity.getclass()
      * 강제 초기화 
        * Hibernate.initialize(entity);

### 즉시로딩과 지연로딩
  * Member를 조회할때 Team도 조회 해야할까?
    member를 통해서 team까지 연계 되는 비즈니스 로직이라면 효율적이나   
    member만 필요한 비즈니스 로직이라면 비효율적이다.   
    => 이를 위해서 jpa에서는 지연로딩과 프록시를 통해서 케어를한다.  

  ```
  // 지연 로딩 및 즉시로딩 사용법
  @ManyToOne(fetch = FetchType.LAZY)  //FetchType.EAGAR
  @JoinColumn(name = "TEAM_ID")
  private Team team;
  ```

  * 지연로딩 내부 순서
    * LAZY로 되어있는 entity 조회
    * 그리고 관련된 team은 프록시객체 생성되어있다. 
    * 실질적인 team의 메소드를 호출할때 초기화가 진행되어 쿼리를 날린다. 


  * 실무에서는 즉시로딩 주의
    * 가급적 지연 로딩만 사용(특히 실무에서)
    * 즉시로딩을 적용하면 예상하지 못하는 sql 발생
    * 즉시로딩은 JPQL에서 N+1 문제를 일으킨다.
    * 무조건 LAZY로 사용해라

  * 지연로딩 활용 - 실무
    * 모든 연관관계에 지연로딩을 사용해라!
    * 실무에서 즉시로딩을 사용하지 마라!
    * JPQL fetch조인이나, 엔티티 그래프 기능을 사용해라!
    * 즉시로딩은 상상하지 못한 쿼리가 나간다.

### 영속성 전이 : CASCADE & orhanRemoval = true
  * 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을때
  * 저장,삽입,삭제 모두 가능
  ```
  @OneToMany(mappedBy = "parent",cascade = CascadeType.ALL,orphanRemoval = true)
  private List<Child> childList = new ArrayList<>();

  --------------- 트랜잭션 부분 -------
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();
    parent.addChild(child1);
    parent.addChild(child2);

    em.persist(parent);
    //em.persist(child1);
    //em.persist(child2);

  ```

  * 영속성 전이는 연관관계를 매핑하는것과 아무 관련이 없음
  * 엔티티를 영속화 할때 연관된 엔티티도 함께 영속화 하는 편리함을 제공할 뿐


  * 고아객체(orphanRemoval)
    * 부모엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
  
  * cascade와 고아객체 모두 주의할점
    * 참조한느곳이 하나일 때 사용해야 함
    * 특정 엔티티가 개인 소유할 떄 사용 