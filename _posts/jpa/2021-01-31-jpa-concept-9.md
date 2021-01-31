---
layout: post
title: jpa-concept-9
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study

## 객체지향 쿼리 언어 


### JPQL
  * EntityManager.find() 가장 단순한 조회 방법

  * jpa를 사용하면 엔티티 객체를 중심으로 개발
  * 문제는 검색쿼리
  * 검색을 할때도 테이블이 아닌 엔티티 객체를 대상으로 검색
  * 모든 DB데이터를 객체로 변환해서 검색하는것은 불가능
  * 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요
  * 객체를 대상으로 쿼리 => JPQL

  ```
  String jpql = "selece m from Member m where m.name like '%hello%'";

  List<Member> result = emd. createQuery(jpql, Member.class).getResultList();
  ```

  * QueryDSL은 사용하는것이 좋다. 하지만 criteriaJPA 사용하지마라
  * QueryDSL장점
    * 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
    * JPQL 빌더 역할
    * 컴파일 시점에 문법 오류를 찾을 수 있음
    * 동적 쿼리 작성 편리함
    * 단순하고 쉬움
    * 실무사용 권장
  

### 기본문법과 기능
  * JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는것이 아니라 엔티티 객체를 대상으로 쿼리한다.
  * JPQL은 SQL을 추상화해서 특정데이터ㅔ이스 SQL에 의존하지 않는다.(방언을 지원하므로 )
  * JPQL은 결국 SQL로 변환된다.

  * DML 사용가능(select - where,groupby,having,orderby 등등)
  * TypeQuery : 반환타입이 명확할 때 사용
    ```
      TypeQuery<Member> query =
          em.createQuery("select m from Member m",Member.class);
    ```
    * Query : 반환 타입이 명확하지 않을 때 사용
    ```
    Query query = 
          em.createQuery("select m.usernaem, m.age from Member m");
    ```   

  * 결과 조회 API
    * query.getResultList(): 결과가 하나 이상일때, 리스트반환
      * 결과가 없으면 빈 리스트 반환
    * query.getSingleResult() : 결과가 정확히 하나, 단일 객체 반환  
      * 결과가없으면 : javax.persistence.NoResultException
      * 둘이상이면 : javax.persistence.NonUniqueExcetion


    * 파라미터 바인딩 - 이름,위치 있는데 이름만 사용할것
    ```
    select m from Member m where m.username = :username
    query.setParameter("username",usernameParam);
    ```

  * 프로젝션
    * 엔티티 프로젝션
    * 임베디드 타입 프로젝션
    * 스탈라 타입 프로젝션
    * 프로젝션 여러값조회
    ```
    objectp[] 타입으로 조회
    ```

  * 페이징API
    * JPA는 페이징을 추상화함
    * setFirstResult(int startPosition) : 조회시작 위치
    * setMaxResult(int maxResult):조회할 데이터수 
    ```
    String jpql = "select m from Member m order by m.name desc";
    List<Member> resultList = em.createQuery(jpql, Member.class)
        .setFirstResult(10);
        .setMaxResult(20)
        .getResultList();
    ```

  * 조인
    * 내부조인 : select m from Member m [INNER] join m.team t
    * 외부조인 : select m from Member m left [outer] join m.team t
    * 세타조인 : selece count(m) from Member m, Team t where m.username = t.name

    []는 생략가능

    * 조인 - on 사용가능
      * 조인 대상 필터링
      ```
      -jqpl-
      select m.t from Member m left join m.team t on t.name = 'A'

      -sql-
      select m.*, t.* from Member m left join team t on m.team_ID=t.id and t.name = 'A'
      ```
      * 연관관계 없는 엔티티 외부 조인 가능
      ```
      -jqpl-
      select m,t from Member m left join Team t on m.username = t.name

      -sql-
      select m.*, t.* from Member m left join Team t on m.username = t.name
      ```

    * 서브쿼리 사용가능
      * exists, all, any,some, in 등 서브쿼리 지원함수 사용가능
      ``` 
      //팀a의 소속인 회원(팀a가 존재하는지 확인후 회원확인)
      select m from Member m where exist (select t from m.team t where t.name = '팀a')

      //전체상품 각각의 재고보다 주문량이 많은 주문들
      select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)

      //어떤 팀이든 팀에 소속된 회원
      select m from Membeer m where m.team = any(select t from Team t)
      ```

      * 서브쿼리의 한계
        * JPA는 where,having 절에서만 서브쿼리 사용가능
        * select절도 가능(하이버네이트지원)
        * From절의 서브쿼리는 현재 JPQL에서 불가능
          * 조인으로 풀수있으면 풀어서 해결
    
    * 조건식 사용가능
     ```
     select
        case when m.age <=10 then '학생요금'
             when m.age >=60 then '경로요금'
             else '일반요금'
        end
      from Member m

      select
        case t.name
             when 'teamA' then '인센110'
             when 'teamB' then '인센120'
             else '인센105'
        end
      from Team t
     ```

     * coalesce : 하나씩 조회해서 null이 아니면 반환
     ```
      select  coalesce(m.username,'이름없는 회원')from Member m
     ```
     * nullif : 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
     ```
      select  NULLIF(m.username,'관리자')from Member m
     ```

### 페치조인 ( 실무에서 정말정말 중요함 )

  * SQL 조인 종류 X
  * JPQL에서 성능 최적화를 위해 제공하는 기능
  ##### * 연관된 엔티티나 컬렉션을 sql 한번에 함께 조회 하는 기능
  * join fetch 명령어 사용
  * 페치 조인 ::= [ left [ outer ] | inner ] join fetch 조인경로

  
  * 회원을 조회하면서 연관된 팀도 함께 조회
   ```
    String jpql = "select m from Member m join fetch m.team"
    List<Member> members = em.createQuery(jpql,Member.calss).getResultList();

    for(Member member : members){
      System.out.println("username = " = member.getUsername() + ", " + "teamName = " + member.getTeam().name());
    }
   ```

  * 컬렉션 페치 조인 사용코드
    ```
    String jpql = "select t from Team t join fetch t.members where t.name = '팀A'" List<Team> teams = em.createQuery(jpql, Team.class).getResultList();

    for(Team team : teams) {
    System.out.println("teamname = " + team.getName() + ", team = " + team); for (Member member : team.getMembers()) {
    //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩 발생 안함
    System.out.println(“-> username = " + member.getUsername()+ ", member = " + member); }
    }
    ```

    * 위와 같은 컬렉션 페치 조인은 중복된 결과를 가져온다.
    * SQL에 DISTINCT를 추가하지만 데이터가 다르므로 SQL 결과 에서 중복제거 실패
    * JPQL의 distinct 애플리케이션에서 중복 제거 시도 - 2가지 기능제공
      * sql에 distinct를 추가
      * 애플리케이션에서 엔티티 중복제거


    * 페치 조인과 일반 조인의 차이
      * JPQL은 결과를 반환할 때 연관관계 고려X
      * 단지 select 절에 지정한 엔티티만 조회할 뿐
      * 여기서는 팀 에네티티만 조회하고, 회원 엔티티는 조회X
      * 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시로딩)
      * 페치조인은 객체 그래프를 SQL 한번에 조회하는 개념

    * 페치 조인 대상에는 별칭을 줄 수 없다.
    * 둘이상의 컬렉션은 페치 조인 할 수 없다.
    * 컬렉션을 페치 조인하면 페이징AP를 사용할 수 없다.
      * 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
      * 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우위험)
    * 연관된 엔티티들을 sql 한번으로 조회-성능최적화
    * 엔티티직접 적용하는 글로벌 로딩 전략보다 우선함
      * @OneToMany(fetch = FetchType.LAZY) // 글로벌 로딩 전략
    * 실무에서 글로벌 로딩 전략은 모두 지연로딩
    * 최적화가 필요한 곳은 페치 조인 적용

    * 페치조인 정리
      * 모든 것을 페치 조인으로 해결할 수 는 없음
      * 페치조인은 객체 그래프를 유지할 때 사용하면 효과적
      * 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야하면, 페치 조인보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적


### 경로 표현식
  * 상태 필드 : 단순히 값을 저장히기 위한 필드
    ```
    (경로 탐색의 끝, 탐색 X)
    select m.username, m.age from Member m
    ```
  * 연관필드 : 연관관계를 위한 필드
    ```
    단일값 연관경로 : 묵시적 내부 조인 발생, 탐색 O
    select m.team from Team t
    ```
    연관필드는 사용치 말고 이를 명시적인 join을 사용해라   
      ```
      select m from Member m join m.team t
      ```
  ###### why???    
  * n+1의 문제 발생가능하므로 

  * 경로 표현식 예제
    ```
    select o.member.team from Order o -> 성공

    select t.members from Team -> 성공

    select t.members.username from Team t -> 실패

    select m.username from Team t join t.members m -> 성공
    ```

  * 실무조언
    * 가급적 묵시적 조인 대신에 명시적 조인 사용
    * 조인은 SQL튜닝에 중요 포인트
    * 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움



### 다형성 쿼리
  * 조횧 대상을 특정 자식으로 한정
    ```
    select i from Item i
    where type(i) IN (BOOK,Movie)
    -------

    -jpql-
    select i from Item i 
    where treat(i as Book).auther = 'kim'
    -sql-
    select i.* from Item i 
    where i.DTYPE = 'B' and i.author = 'kim'
    ```

### 엔티티 직접 사용
  * JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본키 값을 사용
### Named query
  * 미리 정의해서 이름을 부여해두고 사용하는 jpql
  * 정척쿼리
  * 어노테이션,xml에 정의
  * 애플리 케이션 로딩 시점에 초기화 후 재사용
  * 애플리케이션 로딩 시점에 쿼리를 검증

  * => named query의 장점
    * 로딩시점에 쿼리를 검증하므로 애러를 막을수있음
    * 로딩시점에 쿼리를 파싱하므로 빠름
### 벌크 연산
  * 쿼리한번으로 여러 테이블 로우 변경(엔티티)
   ```
    String jpql = "select m from Member m where m.age = 20"

    int resultCount = em.createQuery(jpql,INTEGER.class).executeUpdate();
   ```

  * 벌크연산주의
    * 벌크 연산은 영속성 컨테스트를 무시하고 데이터 베이스에 직접 쿼리
      * 벌크연산을 먼저수행한다.
      * 벌크 연산 수행후 영속성 컨텍스트 초기화
      
    * why?? 초기화 해야하는가?
      * DB상에 직접쿼리를 날리므로 그전에 persist(entity)한 값들이 1차캐시에 남아있다. 컨텍스트를 초기화 하지않고 조회하면 그전에 1차캐시의 값들을 가져오기 때문에


