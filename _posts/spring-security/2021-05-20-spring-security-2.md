---
layout: post
title: 시큐리티 OAuth login
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

# security
  
  oauth2를 위한 버튼, url 등이 필요하며 응답을 받을 url을 각 페이지에서 설정해야한다. 

  ```
  .and()
    .oauth2Login()
    .loginPage("/login")
    .userInfoEndpoint()
    .userService(principalOauth2UserService);
  ```
  oauth2 사용을 위한 DefaultOAuth2UserService (loadUser메서드)

  Authentication 객체가 가질수있는 2가지 타입
   - UserDetails
   - OAuth2User