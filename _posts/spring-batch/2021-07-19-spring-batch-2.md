---
layout: post
title: security batch-2
subtitle: "spring, batch"
categories: spring
tags: security
comments: true
---
> spring study

## spring batch Task & Chunk


## 1. Task & Chunk 

  - 배치를 처리할 수 있는 방법은 크게 2가지
  - Tasklet을 사용한 Task 기반처리
    - 배치 처리 과정이 비교적 쉬운 경우 쉽게 사용
    - 대량 처리를 하는 경우 더 복잡
    - 하나의 큰 덩어리를 여러 덩어리로 나누어 처리하기 부적합
  - Chunk를 사용한 chunk 기반 처리
    - ItemReader, ItemProcessor, ItemWrtier의 관계 이해 필요
    - 대량처리를 하는 경우 Tasklet보다 비교적 쉽게 구현
    - 예를 들면 10,000개의 데이터 중 1000개씩 10개의 덩어리로 수행
      -  이를 Tasklet으로 처리하면 10000개를 한번에 처리하거나, 수동으로 1000개씩 분할



## 2. jobScope, stepScope 

  - jobScope는 job이 생성될고 소멸될때의 사이클을 가진다. 
  - stepScope도 step이 생성될고 소멸될때의 사이클을 가진다. 


  - 이들은 빈자체가 생성주기를 갖게 되고 범위를 가지게 되면서 스레드 세잎 해진다. 


## 3. ItemReader

  - 배치 대상 데이터를 읽기 위한 설정 
    - 파일,DB,네트워크 등에서 읽기 위함
  - step에 itemReader는 필수 
  - 기본제공되는 ItemReader 구현체
    - file,jdbc,jpa,hibernate,kafka
  - ItemReader 구현체가 없으면 직접개발
  - ItemStream은 ExecutionContext로 read, write정보를 저장
  - CustomItemReader예제 참고


## 4. ItemWriter

  - ItemWriter 구현체가 없으면 직접개발 
  - csv, jdbc, jpa등이 사용됨 
  - jdbc
    - JdbcBatchItemWriter를 이용해 insert/update/delete가능
    - 단건 처리가 아니기 때문에 비교적 높은 성능 
  - jpa
    ```
    @Bean
    public Step jpaItemWriterStep() throws Exception {
        return stepBuilderFactory.get("jpaItemWriterStep")
                .<Person, Person>chunk(10)
                .reader(itemReader())
                .writer(jpaItemWriter())
                .build();
    }

    private ItemWriter<Person> jpaItemWriter() throws Exception {
        JpaItemWriter<Person> itemWriter = new JpaItemWriterBuilder<Person>()
                .entityManagerFactory(entityManagerFactory)
              //.usePersist(true)
                .build();

        itemWriter.afterPropertiesSet();
        return itemWriter;
    }
    ```
    
    jpaItemWriter는 위와같이 실행된다. 

    JPA는 변경감지가 되면 merge를 진행하는데 이를 방지하기 위한 메소드가 .usePersist이다. 

    itemReader값을 가져오고 이 데이터가 id 혹은 변경감지 대상이 되지 않는다면 .usePersist를 사용하지 않아도 되긴한다.

## 5. ItemProcessor

  - itemReader, itemWriter 의 기능을 할수있지만 명확한 책임을 부여하기 위해서 각각의 기능을 수행하는것이 좋다. 
  - ItemProcessor는 추가적인 필터링 기능을 수행함이 좋다. 

  - CompositeItemProcessorBuilder()를 통해서 여러개의 processor를 사용할 수 있다. 
    - delegates(param1, param2,..) 등을 통해서 여러개의 함수를 사용한다. 


## 6. ItemWriter

  - ItemReader()가 CSV 파일을 읽었을때
  - JpaWriterBuilder()를 통해서 Writer를 생성할수있다. 
  - 그리고 entityManagerFactory()로 영속화 한다음 Jpa에 넣을수있는것이다. 
