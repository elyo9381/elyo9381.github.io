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