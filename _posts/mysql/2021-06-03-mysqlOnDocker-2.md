---
layout: post
title: mysql on Docker with centos - 2
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> mySQL on Docker 


## proxyLayer 
  
  구성한 master/slave 구조에서 마스터에 문제 발생시 오케스트라를 이용해서 slave1을 마스터로 올리고 하위 slave를 새로운 마스터에 붙일수있다. 

  오케스트라를 통해서 자동으로 failOver가 발생하고 이를 처리한다. 

  하지만 오케스트라에서 failover가 진행되지만 클라이언트의 request를 자동으로 처리하지는 않는다. 

  그렇다면 문제 발생시에 오케스트라에서 failover를 처리해주지만 클라이언트 단의 request를 새로운 마스터로 자동으로 처리해주는 것은 무엇이 있을까??

  이는 바로 proxyLayer 구성하는 것이다. 
  
  - mkdir -p /db/proxysql/data /db/proxysql/conf
  - chmod 777 /db/proxysql /db/proxysql/data /db/proxysql/conf
  
  권한설정 및 필요한 디렉터리를 만든다.   
  proxysql/conf로 이동하여서 proxysql.conf를 생성한다.
  
  proxysql.cnf 파일 설정

  ```
    datadir="/var/lib/proxysql"
    admin_variables=
    {
    admin_credentials="admin:admin;radmin:radmin"
    mysql_ifaces="0.0.0.0:6032"
    }
    mysql_variables=
    {
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033"
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.5.30"
    connect_timeout_server=3000

    ^G Get He^O WriteO^R Read F^Y Prev P^K Cut Te^C Cur Pos
    GNU nano 2.0.6     File: proxysql.cnf

    connect_timeout_server=3000
    monitor_username="monitor"
    monitor_password="monitor"
    monitor_history=600000
    monitor_connect_interval=60000
    monitor_ping_interval=10000
    monitor_read_only_interval=1500
    monitor_read_only_timeout=500
    ping_interval_server_msec=120000
    ping_timeout_server=500
    commands_stats=true
    sessions_sort=true
    connect_retries_on_failure=10
    }

  ```   


  docker container생성

  - docker run -i -t --name proxysql -h proxysql --net mybridge --net-alias=proxysql -p 16032:6032 -p 16033:6033 -v /db/proxysql/data:/var/lib/proxysql -v /db/proxysql/conf/proxysql.cnf:/etc/proxysql.cnf -d proxysql/proxysql

  - mysql -h127.0.0.1 -P16032 -uradmin -pradmin --prompt "ProxySQL admin" : 로그인 하는 방법
  

## proxysql test
  

  - db0001에 테스트용 디비를 생성한다. 
  - create database testdb default character set utf8;
  
  - 어플리케이션에서 사용할 유저를 만든다. 
  - create user appuser@'%' identified by 'apppass';

  - testdb에 읽고 쓸 수 있도록 권한을 준다.
  - grant select, insert, update, delete on testdb.* to appuser@'%';

  - 프록시 시퀄에서 사용할 유저를 만든다. 
  - create user  'monitor'@'%' identified by 'monitor';

  - 프록시 시퀄에서 사용할 권한을 준다.
  - grant REplication client on *.* to 'monitor'@'%';
  - flush privileges;

  - proxysql서버에 로그인하여 각각의 replication을 등록해준다. 
  - insert into mysql_servers(hostgroup_id, hostname, port) values(10,'db001',3306);
  - insert into mysql_servers(hostgroup_id, hostname, port) values(20,'db001',3306);
  - insert into mysql_servers(hostgroup_id, hostname, port) values(20,'db002',3306);
  - insert into mysql_servers(hostgroup_id, hostname, port) values(20,'db003',3306);
  - insert into mysql_replication_hostgroups values(10,20,'read_only','');
  - load mysql servers to runtime;
  - save mysql servers to disk;
  - insert into mysql_users(username,password,default_hostgroup,transaction_persistent) values ('appuser','apppass',10,0);
  - LOAD MYSQL USERS TO RUNTIME;
  - SAVE MYSQL USERS TO DISK;
  - insert into mysql_query_rules(rule_id,active,match_pattern,destination_hostgroup) values (1,1,'^SELECT.*FOR UPDATE$',10);
  - insert into mysql_query_rules(rule_id,active,match_pattern,destination_hostgroup) values (2,1,'^SELECT',20);
  - load mysql QUERY RULES to runtime;
  - save mysql QUERY RULES to disk;

  `insert test`

  - docker exec -it -uroot db001 /bin/bash
  - use testdb;
  - create table insert_test(hostname varchar(5) not null, insert_time datetime not null);
   

  insert test shall   
  ```
    #!/bin/bash

    while true;
    do

    mysql -uappuser -papppass -h172.31.10.19 -P16033 -N -e "insert into testdb.insert_test select @@hostname,now()" 2>&1| grep -v "Warning"

    sleep 1

    done
  ```

  - truncate table testdb.insert_test;
 

## 모니터링 (mysql)
  

  prometheus통해서 모니터링을 진행하고  grafana를 통해서 대시보드로 시각화 한다. 

  prometheus로 모니터링 항목 수집을 위해 exporter가 존재해야한다.   

  prometheus가 풀형태로 진행한다. 

  exporter를 어떻게 만들어야할까요? 도커이미지를 직접 만들거나 또는 만들어진 도커 파일을 찾아야한다.   

  `도커 이미지를 직접 만들것입니다. `
  
  ```
    Dokerfile
    Percona-Server-client-57-5.7.30-33.1.el7.x86_64.rpm
    Percona-Server-server-57-5.7.30-33.1.el7.x86_64.rpm
    Percona-Server-shared-57-5.7.30-33.1.el7.x86_64.rpm
    Percona-Server-shared-compat-57-5.7.30-33.1.el7.x86_64.rpm
    mysqld_exporter-0.12.1.linux-amd64.tar.gz
    node_exporter-1.0.1.linux-amd64.tar.gz
    ps-entry.sh
    start_mysqld_exporter.sh
    start_node_exporter.sh
    ```   
    파일이 하나의 폴더에 필요하다.   
    docker build -t mysql57:0.0 ./ : 명령어를 통해서 도커이미지를 만들수있다. 



    ```
    FROM centos:7
    COPY ["Percona-Server-client-57-5.7.30-33.1.el7.x86_64.rpm",\
        "Percona-Server-server-57-5.7.30-33.1.el7.x86_64.rpm", \
        "Percona-Server-shared-57-5.7.30-33.1.el7.x86_64.rpm", \
        "Percona-Server-shared-compat-57-5.7.30-33.1.el7.x86_64.rpm", \
        "node_exporter-1.0.1.linux-amd64.tar.gz", \
        "mysqld_exporter-0.12.1.linux-amd64.tar.gz", \
        "start_node_exporter.sh", \
        "start_mysqld_exporter.sh", \
        ".my.cnf","/tmp/"]
    USER root
    RUN groupadd -g 1001 mysql
    RUN useradd -u 1001 -r -g 1001 mysql
    RUN yum install -y perl.x86_64 \
        libaio.x86_64 \
        numactl-libs.x86_64 \
        net-tools.x86_64 \
        sudo.x86_64 \
        openssl.x86_64
    WORKDIR /tmp/
    RUN rpm -ivh Percona-Server-shared-57-5.7.30-33.1.el7.x86_64.rpm \
        Percona-Server-shared-compat-57-5.7.30-33.1.el7.x86_64.rpm \
        Percona-Server-client-57-5.7.30-33.1.el7.x86_64.rpm \
        Percona-Server-server-57-5.7.30-33.1.el7.x86_64.rpm
    RUN mkdir -p /opt/exporters/ && \
        tar -xzvf ./node_exporter-1.0.1.linux-amd64.tar.gz -C /opt/exporters && \
        tar -xzvf ./mysqld_exporter-0.12.1.linux-amd64.tar.gz -C /opt/exporters
    WORKDIR /opt/exporters/
    RUN mv node_exporter-1.0.1.linux-amd64 node_exporter && \
        mv mysqld_exporter-0.12.1.linux-amd64 mysqld_exporter && \
        mv /tmp/start_node_exporter.sh /opt/exporters/node_exporter/ && \
        mv /tmp/start_mysqld_exporter.sh /opt/exporters/mysqld_exporter/ && \
        mv /tmp/.my.cnf /opt/exporters/mysqld_exporter/ && \
        chmod o+x /opt/exporters/node_exporter/start_node_exporter.sh && \
        chmod o+x /opt/exporters/mysqld_exporter/start_mysqld_exporter.sh && \
        rm -rf /tmp/*.rpm && \
        /usr/bin/install -m 0775 -o mysql -g mysql -d /var/lib/mysql \
        /var/run/mysqld /docker-entrypoint-initdb.d
    VOLUME ["/var/lib/mysql", "/var/log/mysql","/etc/percona-server.conf.d"]
    COPY ps-entry.sh /tmp/docker-entrypoint.sh
    RUN chmod +x /tmp/docker-entrypoint.sh
    ENTRYPOINT ["/tmp/docker-entrypoint.sh"]
    USER mysql
    EXPOSE 3306
    CMD ["mysqld"]
  ```   
  도커파일은 다음과 같이 구성되어있다.   
  상세한 분석은 다음시간에 알아보자 

  생성한 도커 이미지를 컨테이너로 만들어보자 
  - docker run -i -t --name mydb -e MYSQL_ROOT_PASSWORD="root" -d mysql57:0.0
  
  접속
  - docker exec -it -uroot mydb /bin/bash

## 도커파일 기반 mysql (master,slave prometheus 구성)

  우선 기존의 도커 컨테이너를 모두 삭제한다.
  - docker stop mydb db001 db002 db003
  - docker rm mydb db001 db002 db003

  mysql 그룹과 유저를 각각 생성 해준다. 그리고 chown을 통해서 오너를 변경한다. 
  - groupadd -g 1001 mysql
  - useradd -u 1001 -r -g 1001 mysql
  - chown -R mysql:mysql /db/db001 /db/db002 /db/db003

  새로운 도커 컨테이너 생성
  - docker run -i -t --name db003 -h db003 -p 3308:3306 --net mybridge --net-alias=db003 -v /db/db003/data:/var/lib/mysql -v /db/db003/log:/var/log/mysql -v /db/db003/conf:/etc/percona-server.conf.d -e MYSQL_ROOT_PASSWORD="root" -d mysql57:0.0   
  
  같은 방법으로 002,003 도 실행

  프로메테우스 설정을 위한 디렉터리 생성
  - mkdir -p /db/prom001 /db/prom001/data /db/prom001/conf
  - chmod 777 /db/prom001 /db/prom001/data /db/prom001/conf

  /db/prom001/conf에 promethus.yml 설정파일 만들었다.

  promethus 실행
  - docker run -i -t --name prom001 -h prom001 --net mybridge --net-alias=prom001 -p 9090:9090 -v /db/prom001/data:/data -v /db/prom001/conf:/etc/prometheus -d prom/prometheus-linux-amd64

  promethus를 이용할 유저 생성 (db001)에서 생성
  - create user 'exporter'@'localhost' identified by 'exporter123' WITH MAX_USER_CONNECTIONS 3;
  - grant PROCESS, REPLICATION CLIENT, select on *.* to 'exporter'@'localhost';


  exporters 실행
  - docker exec db001 sh /opt/exporters/node_exporter/start_node_exporter.sh
  - docker exec db001 sh /opt/exporters/mysqld_exporter/start_mysqld_exporter.sh

  db001,db002,db003 모두 exporter를 실행한다.   
  그러면 모니터링을 위한 exporter가 준비 완료 되었다.   

  - http://{도커.ip}/graph 에서 확인할 수 있다. 

## 그라파나 설정

  - docker run -i -t --name grafana -h grafana -p 13000:3000 --net mybridge --net-alias=grafana -d grafana/grafana
  - http://{도커.ip}/13000 에 들어가서 직접 대시보드를 볼수있다.
    - id : admin , pwd : admin
  - grafana의 설정에서 prometheus를 설정할수있으며 `https://github.com/percona/grafana-dashboards/blob/master/dashboards/MySQL_Overview.json` 에서 json파일을 복사하여 웹상에서 import하면 유용하게 대시보드를 사용할 수 있다.


## doker compose

  여러 컨테이너를 한방에 배포하기 

  도커 컴포즈 설치   
  - curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

  권한 설정 
  - chmod +x /usr/local/bin/docker-compose

  실행 확인 
  - docker-compose -v

  기존의 컨테이너 및 host의 data 공유 폴더 삭제 

  도커 컴포즈를 이용한 여러 컨테이너 다움
  - docker-compose up -d
  - 그후 shallscript를 이용해서 각각의 설정을 해줘야함 (유저생성, 권한 생성 등)

## Scalability

  도커 호스트에 리소스가 부족해지면 어떻게 해야할까?? 

  가장 쉽게 생각 할 수 있는것은 바로 scale up이다. 

  - scale up  : 컴퓨터의 리소스를 업그레이드 시키는 방법 (cpu,ram)등을 더 높은 사양으로 교체한다. 

  리소스를 늘릴수 없다면 ??  scale out이다. 

  - scale out : 비슷한 컴퓨터 리소스를 구성하여 확장하는 시스템이다. 물론 자동으로 확장되는것은 아니고 다양한 설정이 필요하다. 

  도커에서는 도커Swarm을 지원한다.

  - docker Swarm : 여러대의 도커 호스트를 하나의 도커호스트처럼 사용 할 수 있게 해준다. 
  - Swarm mode : 매니저,워커 노드로 구성된 docker culster이다. 

## docker hub에 custom image 등록

  - docker login
  - docker tag mysql57:0.0 elyo9381/mysql57:0.0
  - docker push elyo9381/mysql57:0.0
  
  - docker service create --name db001 -hostname db001 -p 3306:3306 -mount type=bind,source=/db/db001/data,target=/var/lib/mysql -mount type=bind,source=/db/db001/log,target=/var/log/mysql -mount type=bind,source=/db/db001/conf,target=/etc/percona-server.conf.d -e MYSQL_ROOT_PASSWORD="root" --with-registry-auth elyo9381/mysql57:0.0 
  
  위의 docker service는 swarm mode시에 사용될 custom image를 허브에 올린 image를 사용하여 도커 런 한다.

  --with-registry-auth 는 


## 백업과 복구

  swarm을 이용해서 복구를 진행할것인데 이는 swarm mode를 테스트 실습 해보고 스크랩 하겠다.
  


  









  

