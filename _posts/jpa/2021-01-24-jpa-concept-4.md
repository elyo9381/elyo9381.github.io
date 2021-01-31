---
layout: post
title: jpa-concept-4
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study

## 연관관계 매핑 기초

 * 객체와 테이블 연관관계의 차이를 이해
 * 객체의 참조와 테이블의 외래키를 매핑
 * 용어 이해
   * 방향 : 단방향, 양방향
   * 다중성 : 다대일, 일대다, 일대일, 다대다
   * 연관관계의 주인 : 객체 양방향 연관관계는 관리 주인이 필요


 ### 단방향 연관매핑
  ```
    @ManyToOne
    @JoinColumn(name = "TEAM_ID") 
    private Team team;
  =========================
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    // 연관관계설정
    Member member = new Member();
    member.setName("memeber1");
    member.setTeam(team);
    em.persist(member);

    // 연관관계 설정시 그래프 탐색
    Member findMember = em.find(Member.class,member.getId());

    Team findTeam = findMember.getTeam();
    System.out.println("findTeam = " + findTeam.getName());

    // 새로운 팀 B
    Team teamB = new Team();
    member.setName("teamB");
    em.persist(teamB);0

    // 회원1에 새로운 팀 B설정
    member.setTeam(teamB);
  ```
    * 다대일 연관관계는 @ManyToOne 애노테이션으로 설정 가능하다.
    * @JoinColumn을 설정하면 jpa가 name을 확인하고 fk로 설정해준다. 타입을 Team인 이유는 객체스럽게 사용하기 위해서 이다. 
  
  ### 양방향 연관관계와 연관관계의 주인 
  * 객체는 참조라는것을 사용하고 테이블은 조인을사용한다. 
  * 둘의 차이점을 명확하게 알아야한다. 왜? 필요한지 알아야한다. 

    양방향 객체 연관관계 : Member는 Team의 fk를 가지고 있어 참조 가능하다. Team이 Member를 참조하려면 List members라는 것을 가지고 있어야 참조(탐색) 가능하다.

    테이블 연관관계 : Member와 team은 각각 pk를 가지고 있으면 member는 fk를 가지고 있다. 이를 통해 조인하여 서로 탐색이(조회가) 가능하다.
  ```
  [Team.java]

   // mappedBy는 Member의 Team타입의 team에 연결되어있다는 뜻
   @OneToMany(mappedBy = "team")
   private List<Member> members = new ArrayList<>();
  ===================================

  [JpaMain.java]

    Team team = new Team();
    team.setName("TeamA");
    // team.getMembers().add(member);// mappedBy 의해서 주인이 아니다.
    em.persist(team);

    // 연관관계설정
    Member member = new Member();
    member.setName("memeber1");
    // member.changeTeam(team); // 멤버에 팀을 추가
    em.persist(member);

    team.addMember(member); // 팀에 멤버를 추가
    // <양방향 객체 jpa 참조할시 둘중에 한곳에서만 하는게 났다. >

    // 양쪽으로 데이터를 넣어야한다. 왜? 한쪽으로 넣으면 commit 날리기 전에
    // 멤버에 팀을 넣거나 팀에 멤버를 넣을수없으므로
    // 그렇다면 어떻게 해야하나? 양방향 편의 메소드를 생성하라

    // em.flush();
    // em.clear();

    // 연관관계 설정시 그래프 탐색
    Member findMember = em.find(Member.class,member.getId());
    List<Member> members = findMember.getTeam().getMembers();

    System.out.println("================");
    for(Member m : members){
        System.out.println("m = " + m.getName());
    // System.out.println("team name = " + m.getTeam().getName());
    }
    System.out.println("================");
    // 가장 많이 하는 양방향 실수 to.string,loombok,json 생성라이브러리

  ```
  * 컨트롤러에서 엔티티를 json으로 반환하면 에러가 생길수 있다. 
    * 이를 방지하는것은 엔티티를 json으로 반환하지 마라 >> DTO이용해라 
  
  * 연관관계의 주인은 외래키의 위치를 기준으로 정해야함 


 ### 연관관계 매핑 시작

   * jpashop을 프로젝트에 연관관계 설정을 해본다.


   * 양방향 연관관계를 설정하는 이유?
     * 조금더 편한 개발을 위해서   