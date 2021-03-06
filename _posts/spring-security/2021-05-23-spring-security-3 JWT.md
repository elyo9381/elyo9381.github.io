---
layout: post
title: 시큐리티 JWT
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

## 세션
  
  우리가 알아볼내용 jwt에서 가장 중요한 질문이다.

  - jwt을 왜 사용되는지 
  - jwt가 어디에 쓰는지 

  jwt를 공부하기 앞서 session에 대해서 다시 한번 정리 해보겠습니다.

  session은 어떠한 정보가 내포 되어있는 캐시의 개념입니다.    
  session은 쿠키에 저장됩니다.    
  쿠키는 http.header에 보관(저장) 됩니다.    

  우리는 홈페이지에 여러번 로그인하거나 방문할수있습니다. 
  재접속 하거나 재방문 할때마다 session(쿠키)를 요구한다면 매우 비효율적일것입니다. 

  그렇기 때문에 우리는 쿠키를 보관하고 이를 통해서 재방문,로그인,다양한 정보를 보관합니다.

  세션(쿠키)는 다음과 같은 방식으로 동작합니다.   
  1. 최초 요청시 서버는 목록에 쿠키를 저장하고 헤더에 쿠키를 담아 클라이언트에 던져줍니다.   
  2. 브라우저 내부에 쿠키를 보관하고 있습니다.
     1.  클라이언트가 재요청시 요청에 쿠키의 정보를 사용하여 재요청을 보냅니다.    
  3. 서버는 요청을 받고 쿠키를 확인하고 목록에서 쿠키가 있으면 기존의 사용자임을 확인합니다.     
   
  세션은 서버측에서도 삭제할수있고 클라이언트측에서도 삭제 할수있습니다. 특별한 경우가 없을경우 시간에 따라 삭제됩니다. 

  
  세션의 단점 : 클라이언트가 많은때 `로드밸런싱`이 일어난다. 서버가 여러대일때 내가 최초 들어갔던 서버가 아니면 세션은 계속 생성된다.

  세션의 단점을 처리는 방법은 여러 방법이 존재한다. 해결방법은 : 공유되는 메모리 `메모리서버`를 사용한다. 

  이러한 고질점은 해결하기 위해서 JWT를 사용한다. 


## TCP
  
  물,데,네,트,세,프,용 !!

  OSI 7계층을 알아야한다.   
  OSI 7계층은 각 계층별로 선정되는 데이터가 존재하고 다음 계층으로 전달한다.   
  대략적인 느낌은 이렇게는 것이다.    
  정확한 내용은 다른 블로그를 참고 하길 바랍니다.  

  - TCP 
    - 신뢰를 기반으로 전송하는 방법이다.
    - ack를 기반으로 통신한다. ack가 오지 않는다면 다시 전송을 진행한다. 
  - UDP
    - ack를 보내지 않고 데이터를 전송한다. 
    - 신뢰적 전송을 제공하지 않는다. 
    - UDP는 하나의 회선을 차지하고 있는다. 그리고 stream, 전화등에서 사용된다.

## CIA(confidential, Integrety, authability)
  
  위에서 TCP에 대해서 간단하게 정립하였다.   
  TCP 전송중에 누군가 데이터를 확인한다면 ? 어떻게 되는가?? : 기밀성이 깨진다. 

  그 데이터를 변경하면 ?? : 무결성이 깨진다.

  마지막으로 우리는 원래 받아야할 데이터를 못받았므로 가용성이 깨진다. 

  통신에서 CIA를 지키기 위해서 암호화를 한다. 

  암호화를 하기위해서 다양한 방법이 필요하다 (대칭키 , 비대칭키)

## RSA

  public key : 공개키

  private key : 개인키

  송신자는 수신자의 공개키로 암호화하고 수신자는 본인의 개인키로 복호화 할수있다.   
  우리는 이렇한 방법을 통해서 암호화가 가능하다   
  이것이 `RSA` 암호 방식이다. 

  또한 공개키 , 개인키를 기반으로 전자서명이 존재한다. 

  A는 본인의 개인키로 암호화하고 누군가에게 송신한다.   
  A의 메세지를 수신받은 B는 A의 공개키로 메세지를 열어볼수있고 이를 통해서 메세지 내용을 확인할수있다.   
  우리는 이를 `전자서명`이라 한다. 

  RSA는 이산대수 문제의 원리를 통해서 암복호화 한다.

  ```
주어진 g, x, p를 이용하여 y = g^x mod p를 구하긴 쉽지만, g, y, p 값을 이용하여 원래의 x는 찾기 어렵다는 것이다
  ``` 


## RFC문서
  인터넷 통신을 프로토콜로 정의해놓은 문서이다.   
   
  