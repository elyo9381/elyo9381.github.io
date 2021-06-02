---
layout: post
title: mysql on Docker with centos - 1
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> mySQL on Docker 


## container
  
  도커에서 mysql 실행
  - docker run -i -t --name db001 -e MYSQL_ROOT_PASSWORD="root" -d percona:5.7.30
  
  실행된 컨테이너 확인
  - docker ps

  컨테이너 접속(execute)
  - docker exec -it db001 /bin/bash
  

  mysql 접속
  - mysql -uroot -p
  


  -i -t : container에 shell 로 접속해서 사용하기 위한 옵션   
  --name : container의 이름   
  -e:환경변수 세팅   
  -d: background mode로 container 실행

## 외부에서 mysql 접속하기

  container 외부에서 Mysql접속하기 
  - docker run -i -t --name db001 -p 3306:3306 -e MYSQL_ROOT_PASSWORD="root" -d percona:5.7.30
    
    -p 옵셥을 통해서 포트를 설정할수있다. 


  Mysql 접속하기
  - mysql -uroot -p -h {docker_host_ip}
  
  이때 나는 percona-mysql을 사용하였다. 그러므로 이를 들어갈 클라이언트를 등록해야 사용할 수 있다. 

 `Stateless VS Stateful`   
  - container는 언제든지 재 시작 될 수 있다.
  - Docker image만 있으면 언제든지 동일한 구성의 Container를 실행시킬 수 있다. 
  - container가 삭제 후 재 생성되면 docker image 초기의 상태로 시작된다.
  - Web server처럼 특정 요청을 받아서 처리해주고 상태값이나 데이터를 갖지 않는 형태의 서비스에 적합 : Stateless
  - 하지만, DB는 데이터를 저장.
  - MYSQL Container
