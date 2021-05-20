---
layout: post
title: security study
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

# security

  - spring security는 여러가지 기능을 가지고 있다. 
    - authentication, authrize 기능이 대표적이다. 

  그렇다면 어떻게 진행되는것인가?

  시큐리티는 다음과같은 설정과 방법이 필요하다. 

  1. 시큐리티 기능사용을 위한 의존성 등록
  2. 시큐리티 기능을 수행하기에 앞서 설정 (SecurityConfig)
     1. WebSecurityConfigurerAdapter를 상속받는 SecurityConfig
     2. configure()를 오버라이딩 하여 http.request요청이에 다양한 설정을 등록수있다.
     - 권한, 로그인설정 등등
  3. 로그인관련 요청이 들어오면 시큐리티가 로그인을 진행한다. 
     1. 이때 설정한 내용을 토대로 Security ContextHolder에 시큐리티 세션이 만들어진다. 
     2. Security (sesseion => Authentication => UserDetails) 타입으로 구성되어있다. 
     3. 그래서 우리는 UserDetails객체를 이용한 PrincipalDetails,PrincipalDetailService를 만들어야한다. 

  시큐리티는 크게 위와같은 로직으로 동작한다.

  시큐리티 구현을 위한 세부적인 로직(다양한 설정)은 자세히 알아보자 


## 구현 목록
  
  시큐리티 구현을 위해서는 PrincipalDetails, PrincipalDetailService, securityConfig가 필요하다. 

  1. securityConfig는 WebSecurityConfigurerAdapter를 상속받고 오버라이딩을 통해서 기능 구현한다. 
     - @EnableWebSecurity !
  2. PrincipalDetails은 UserDetails상속받고 오버라이딩을 통해 기능 구현한다.
  3. PrincipalDetailService는 Serivce로 등록해서 시큐리티가 login을 동작시킬 로직을 작성한다. 


## PrincipalDetails

  PrincipalDetails에서 우리는 사용할 객체를 콤포지션한다.

  그리고 생성자를 만들어서 사용할 객체를 주입한다. 

  여기에서 우리는 다양한 메서드를 오버라이딩 해야한다.
  ```
  // Authentication 객체에 저장할 수 있는 유일한 타입
  @Data
  public class PrincipalDetails implements UserDetails{

	private User user;

	public PrincipalDetails(User user) {
		super();
		this.user = user;
	}
	
	@Override
	public String getPassword() {
		return user.getPassword();
	}

	@Override
	public String getUsername() {
		return user.getUsername();
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}
	
	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		Collection<GrantedAuthority> collet = new ArrayList<GrantedAuthority>();
		collet.add(()->{ return user.getRole();});
		return collet;
	}
  }
  ```

## PrincipalDetailsService
  PrincipalDetailsService은 DetailsUserSerivce의 구현체이다.

  longin 요청이 오면 자동으로 UserDetailsService 타입으로 IoC되어 있는 loadUserByUsername 함수가 실행된다.

  loadUserByUsername()는 login시에 username이 존재하는지 확인한다. 어디서? 레포지터리에서 

