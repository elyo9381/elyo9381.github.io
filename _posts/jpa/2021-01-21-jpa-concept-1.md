---
layout: post
title: jpa-concept-1
subtitle: "spring, framework, jpa"
categories: spring
tags: framework
comments: true
---
> spring jpa study

# sql의 문제점
 * 상속을 사용하듯이 sql을 사용할수가 없다. 
 ```
  String memberId = "100";
  Member member1 = memberDAO.getMember(memberId);
  Member member2 = memberDAO.getMember(memberId);
  member1 == member2; //다르다. class MemberDAO {
  public Member getMember(String memberId) {
  String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
  ...
  //JDBC API, SQL 실행
  return new Member(...);
  }
 ```   
다음과 같은 상황에서 sql을 사용하게 된다면 쿼리는 동일하나 반환하는 member의 값이 다르다    
그런데 아래와 같이 자바컬렉션에서와 같이 조회 가능하다면 좋겠다는 생각이 들고    
이를 해결하기 위해서 나온것이 jpa이다.    
```
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);
member1 == member2; //같다.
```

* 객체답게 모델링 할수록 매핑 작업만늘어난다. >> sql과 객체지향의 문제점

# JPA?
 * java 진형의 orm이다.

 jpa는 jdbcapi를 사용하여 DB접근을 하는데 패러다임의 불일치를 해결해준다.

 jpa는 인터페이스의 모음이다.

## why? jpa를 사용하는가.??
 - 생산성적인 측면에서 저장,조회, 수정, 삭제 등이 만들어진다. 쿼리가 만들어진다._
 - 유지 보수측면에서는 필드변경시 모든 sql수정해야하지만 jpa사용하게 되면 필드만 수정하면된다. 
 - 데이터베이스의 구조를 생각치 않아도 DB의 구조를 생각해서 (조인및 insert를 알아서)패러다임을 만들어준다.
 - jpa의 연관관계,객체 그래프탐색을 자바에서 사용하듯 사용할수있다. 또한 신뢰할수있는 엔티티를 보장한다. 
 - 동일한 엔티티의 조회에서 트랜잭션을 보장한다.
 

 * 성능 최적화 기능
  - 1차 캐시와 동일성 보장
  - 트랜잭션을 지원하는 쓰기 지연
  - 지연로딩

