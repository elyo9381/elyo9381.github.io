---
layout: post
title: 리팩터링 리뷰 - 4
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> repactoring

----
캡슐화
====

## 1.레코드 캡슐화하기

  - 레코드는 명확한 값과 명확하지 않은값을 구분해서 저장해야하는 점이 번거롭다.
  - 객체를 사용하면 어떻게 저장했는지 숨긴 채 세 가지값을 각각의 메소드로 제공할 수 있다. 
  - `레코드를 캡슐화하는 목적은 변수 자체는 물론 그 내용을 조작하는 방식도 통제하려함이다.`
  - 캡슐화에서는 값을 수정하는 부분을 명확하게 드러내고 한곳에 모아두는 일이 굉장히 중요하다.
  - 캡슐화는 모든쓰기 를 함수 안에서 처리한다는 원칙이 존재한다.


  - getter, setter 를 이용해서 OOP의 클래스를 만드는 경우
  - 중첩된 레코드 (JSON)- 레코드 아래 또 다른 레코드가 있을때
      - 하위 레코드에 대한 읽기 쓰기도 메소드로 생성(상위 레코드에) 
      - 최상단 클래스화 - 그다음도 클래스화 하면 중첩구조도 클래스를 중첩된 경우도 재귀적으로 사용할수있음

## 2. 컬렉션 캡슐화하기

  - 게터가 컬렉션 자체를 반환하도록 한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀌어 버릴 수 있다. 
  - 컬렉션을 소유한 클래스를 통해서만 원소를 변경하도록 프로그램을 개선하면서 컬렉션 변경 방식도 원하는 대로 수정할수있다.
  - 컬렉션에 접근하려면 컬렉션이 소속된 클래스의 적절한 메서드를 반드시 거치게 하는 것이다. 

  - add/remove를 통한 컬렉션 접근
  - 게터는 생성하되 세터는 만들지 않는걸로 

## 3. 기본형을 객체로 바꾸기

  - 단순 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의하는 편이다. 
  - 기본형 집착 악취

## 4. 임시 변수를 질의 함수로 바꾸기

  - 임시 변수를 사용하면 값을 계산하는 코드가 반복되는걸 줄이고 값의 의미를 설명할 수도 있어서 유용하다.
  - 

## 5. 클래스 추출하기

  - 클래스는 반드시 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다는 가이드라인이 존재한다.
  - 메서드와 데이터가 너무 많은 클래스는 이해하기가 쉽지 않으니 잘 살펴보고 적절히 분리하는것이 좋다. 


## 6. 클래스 인라인하기
  - 리팩터링을 하고 난 후 특정 클래스에 남은 역할이 거의 없을때 이런 현상이 생긴다. 
  - 두 클래스의 기능을 지금과 다르게 배분하고 싶을 때도 클래스르 인라인 한다. 
      - : 
  
  - 우려사항 : 하는일이 별로없는데 여러클래스에서 사용한다면 인라인 하기 어렵다고 생각한다.
      - 추출과 인라인 사이는 직관에 따라 가야한다. 책은 단순 기법만 설명하는것이다. 

## 7. 위임 숨기기

  - 위임객체 : 중첩구조에서 중간 객체 
      - 하위 객체의 정보를 입출력하기 위해 위임
  - 중첩구조를 알 필요가 없어짐
  - 위임 객체의 인터페이스가 바뀌면 인터페이스를 사용하는 모든 클라이언트가 코드를 수정해야한다. 


## 8. 중개자 제거하기

  - 위임 숨기기의 배경에서 반대되는 개념이다. 
  - 위임 메서드를 통해서 추가적인 단순 전달만 하는기능만 수행하다 보면 메서드들이 점점 성가셔 진다. 
  - 이러면 위임 메서드는 중개자 역할을 수행하게 되고 차라리 직접 클라잉언트가 위임 객체를 직접 호출하는게 나을수 있다.

  - 중개자 제가하기는 판단하기 힘들다. 왜냐하면 개발자의 직관으로 판단해야함이다. 혹은 팀내(회사) 코딩 컨벤션등을 따라야한다
  - 위임객체 혹은 중개자 설정은 트레이프 관계 임으로 상당한 고려(회의) 통해서 진행되어야한다. 


## 9. 알고리즘 변경하기 

  - 복잡한 알고리즘(지저분) 존재할때 간편하게 할수있다면 알고리즘 교체도 좋은 방법이고 이를 캡슐화 한다면 좋다. 
  