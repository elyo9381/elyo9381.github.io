---
layout: post
title: 시큐리티 JWT - 2
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

## JWT
  JSON WEB TOKEN : JWT

  정의 : 당사자간에 정보를 JSON 객체로 안전하게 전송하기 위한 컴팩트하고 독립적인 방식을 정의 하는 개방형 표준 입니다.   
  JWP는 비밀(RSA or ECDSA)로 암호화및 복호화를 통해 서명 할 수 있습니다.

  JWT는 다음과 같이 구성되어있다. 
  - header
    - 알고리즘과 타입으로 구성되어있고 Base64UrI로 인코딩 되어있다.
    
  - payload : 어떤 정보이다. 
    - 클래임을 가진다. 

  - signature
    - 헤더의 정보와 페이로드와 개인키를 암호알고리즘으로 암호화 한다. 

  ```
  xxxx-yyyy-zzzz
  ```
  
  JWT는 header,payload,signature를 각각 base64인코딩하여 JWT를 구성하고 web의 로컬스토리지에 담긴다.

  그리고 이를 서버에 넘긴다. 

  JWT를 받은 서버는 신뢰할수있는 토큰인지 검증한다. 

  JWT를 암호화할때 RSA또는 hs256으로 암호화 할수있다. (전자서명)