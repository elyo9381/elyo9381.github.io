---
layout: post
title: 시큐리티 원리 - 4
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

## baseAuthenticationFilter

  - 기본적으로 로그인 페이지를 사용할 수 없는 상황에서 사용합니다.
    - SPA 페이지
    - 브라우저 기반의 모바일 앱

   
  - 사용방법
  ```
  public class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
      http
              .httpBasic()
              ;
  }
  }
  ```

  
