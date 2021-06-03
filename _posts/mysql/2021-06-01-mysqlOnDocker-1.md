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
  <!-- - mysql -uroot -p -h 172.31.2.88 -->
  
  이때 나는 percona-mysql을 사용하였다. 그러므로 이를 들어갈 클라이언트를 등록해야 사용할 수 있다. 

 `Stateless VS Stateful`   
  - container는 언제든지 재 시작 될 수 있다.
  - Docker image만 있으면 언제든지 동일한 구성의 Container를 실행시킬 수 있다. 
  - container가 삭제 후 재 생성되면 docker image 초기의 상태로 시작된다.
  - Web server처럼 특정 요청을 받아서 처리해주고 상태값이나 데이터를 갖지 않는 형태의 서비스에 적합 : Stateless
  - 하지만, DB는 데이터를 저장.
  - MYSQL Container가 삭제되고 재생성되면 어떻게 될까 ? -> data loss발생

## Volume 설정

  ### ***host와 볼륨 공유하기***

  우리는 하나의 서버를 만들었고 이를 마스터로 사용할 것이다. 

  - mkdir -p /db/db001/data
  - mkdir -p /db/db001/log /db/db001/conf
  - chmod 777 /db /db/db001 /db/db001/data
  - chmod -p /db/db001/log /db/db001/conf
  - docker run -i -t --name db003 -p 3308:3306 -v /db/db003/data:/var/lib/mysql -v /db/db003/log:/var/log/mysql -v /db/db003/conf:/etc/percona-server.conf.d -e MYSQL_ROOT_PASSWORD="root" -d percona:5.7.30

  데이터 생성
  ```
  create database testdb default character set=utf8;

  create table t1(id int not null);

  insert into t1 values(1),(2),(3);
  ```

## DB replication
  
  슬레이브를 생성한다

  외부에서 접근가능하며 호스트에서 데이터공유가능하며 마스터와 데이터 복제를 진행한다.
  
  - mkdir -p /db/db002/data /db/db003/data
  - chmod 777 /db/db002 /db/db002/data
  - chmod 777 /db/db003 /db/db003/data
  - mkdir -p /db/db002/log /db/db002/conf
  - mkdir -p /db/db003/log /db/db003/conf
  - chmod 777 /db/db002/log /db/db002/conf
  - chmod 777 /db/db003/log /db/db003/conf

  - docker run -i -t --name db001 -h db001 -p 3306:3306 -v /db/db001/data:/var/lib/mysql -v /db/db001/log:/var/log/mysql -v /db/db001/conf:/etc/percona-server.conf.d -e MYSQL_ROOT_PASSWORD="root" -d percona:5.7.30 : mysql 만들기
  
  - docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}" : 내가 원하는 항목만 볼수있다.

  ```
  CREATE USER 'rep1'@'%' IDENTIFIED BY 'rep1';

  GRANT REPLICATION SLAVE ON *.* TO 'rep1'@'%';
  ```
  마스터 컨테이너의 rep1유저 생성(MySQL)

  master container의 ip : 172.17.0.3

  그후에 db002 컨테이너의 mysql로 들어간다. 

  ```
  CHANGE MASTER TO MASTER_HOST='172.17.0.3', MASTER_USER='rep1', MASTER_PASSWORD='rep1', MASTER_AUTO_POSITION=1;
  ```

  그후 start slave 명령어를 통해서 slave를 시작하고 show slave status\G 명령어를 통해서 잘 연결되었는지 확인한다. 

## 브릿지 네트워크 구성
  컨테이너는 언제든지 재 시작 될 수 있고 컨테이너가 재시작 되면 해당 컨테이너의 IP가 변경될수있습니다. 

  MYSQL 의 Replication 설정이나 HA 설정에 IP를 사용하게 되면 컨테이너가 재 시작 될 경우 변경된 ip 때문에 replication이 깨질 수 있습니다. 

  이러한 문제를 방지하기 위해 Brige Network를 구성하고 net alias를 사용하여 ip변경에도 문제가 발생하지 않도록 할 수 있습니다. 


  - docker network ls : 도커의 네트워크 list를 볼수있는 명령어

  - docker network create --driver bridge mybridge : 브릿지 네트워크생성 명령어

  - docker run -i -t --name db001 -h db001 -p 3306:3306 --net mybridge --net-alias=db001 -v /db/db001/data:/var/lib/mysql -v /db/db001/log:/var/log/mysql -v /db/db001/conf:/etc/percona-server.conf.d -e MYSQL_ROOT_PASSWORD="root" -d percona:5.7.30 

## orchestrator

  failOver가 발생했을때 처리하는 방법

  slave를 master로 올릴수 있다. 

  오케스트레이터 컨테이너 실행하기
  - docker run -i -t --name orchestrator -h orchestrator --net mybridge --net-alias=orchestrator -p 3000:3000 -d openarkcode/orchestrator:latest

  - docker inspect --format '{{.NetworkSettings.Networks.mybridge.IPAddress}}' db001 : db001 컨테이너의 ip주소 확인 

  orchestrator를 사용할 유저생성 (in db001)
  - reate user orc_client_user@'172.%' identified by'orc_client_password';
  - grant super,process, replication slave, reload on *.* to orc_client_user@'172.%';
  - grant select on mysql.slave_master_info TO orc_client_user@'172.%';

  publicIP:3000/web/cluster 로 들어가면 orchestrator의 웹이 나온다. 


## HA(High Availability test)
  
  시나리오 1

  - db001이 정지되어 db002를 마스터로 올립니다.
  - db001은 개별 마스터가 되어 독립적으로 존재합니다.
  - db001을 db002의 slave로 등록시킵니다. 
    - 방법은 set global read_only = 1;
    - CHANGE MASTER TO MASTER_HOST='db002', MASTER_USER='rep1', MASTER_PASSWORD='rep1', MASTER_AUTO_POSITION=1;
    - start slave; 
