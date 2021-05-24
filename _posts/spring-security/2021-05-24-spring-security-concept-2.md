---
layout: post
title: 시큐리티 원리 - 2
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

# 폼 로그인


## DefaultLoginPageGeneratingFilter
  
  - GET /login 을 처리
  - 별도의 로그인 페이지 설정을 하지 않으면 제공되는 필터
  - 기본 로그인 폼을 제공
  - OAuth2 / OpenID / Saml2 로그인과도 같이 사용할 수 있음.

## UsernamePasswordAuthenticationFilter

  - POST /login 을 처리. processingUrl 을 변경하면 주소를 바꿀 수 있음.
  - form 인증을 처리해주는 필터로 스프링 시큐리티에서 가장 일반적으로 쓰임.
  - 주요 설정 정보 
    - filterProcessingUrl : 로그인을 처리해 줄 URL (POST)
    - username parameter : POST에 username에 대한 값을 넘겨줄 인자의 이름
    - password parameter : POST에 password에 대한 값을 넘겨줄 인자의 이름
    - authenticationDetailSource : Authentication 객체의 details 에 들어갈 정보를 직접 만들어 줌.
    ```
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        String username = obtainUsername(request);
        username = (username != null) ? username : "";
        username = username.trim();
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }

    ```

## DefaultLogoutPageGeneratingFilter
  - GET /logout 을 처리
  - POST /logout 을 요청할 수 있는 UI 를 제공
  - DefaultLoginPageGeneratingFilter 를 사용하는 경우에 같이 제공됨.

## LogoutFilter
  - POST /logout 을 처리. processiongUrl 을 변경하면 바꿀 수 있음.



## 실습 예제

  - HomeController 
  - SecurityConfig
  - RequestInfo
  - CustomAuthDetails
  
  위와 같은 컨드롤러 및 컨피그를 설정하여 실습을 진행함 
  컨트롤러에는 로그인, 로그아웃, 홈, 엑세스디나인, 유저 , 어드민을 갈수있는 url이 설정되어있다.   
  페이지는 간단하게 string만 출력한다.   

  유저는 SecurityContig에서 간단하게 설정하였다. 
  ```
   @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .inMemoryAuthentication()
                .withUser(
                        User.withDefaultPasswordEncoder()
                                .username("user1")
                                .password("1111")
                                .roles("USER")
                ).withUser(
                User.withDefaultPasswordEncoder()
                        .username("admin")
                        .password("2222")
                        .roles("ADMIN")
        );

    }
  ```

  request를 위한 필터 설정은 SecurityConfig의 configure(HttpSecurity http)에서 설정하였다.

  ```
   @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests(request->{
                    request
                            .antMatchers("/").permitAll()
                            .anyRequest().authenticated()
                            ;
                })
                .formLogin(
                        login->login.loginPage("/login")
                        .permitAll()
                        .defaultSuccessUrl("/",false)
                        .failureUrl("/login-error")
                        .authenticationDetailsSource(customAuthDetails)
                )
                .logout(logout->logout.logoutSuccessUrl("/"))
                .exceptionHandling(exception->exception.accessDeniedPage("/access-denied"))
                ;
    }
  ```

  위의 코드에서 각 유저의 로그인을 설정하였고 커스텀한 Details를 설정하였다.   
  `커스텀 Details`는 아래와 같이 구성되어있고 ip,sessionId,로그인시각을 기록한다. 
  ```
  @Component
    public class CustomAuthDetails implements AuthenticationDetailsSource<HttpServletRequest,RequestInfo> {

        @Override
        public RequestInfo buildDetails(HttpServletRequest request) {
            return RequestInfo.builder()
                    .remoteIp(request.getRemoteAddr())
                    .sessionId(request.getSession().getId())
                    .loginTime(LocalDateTime.now())
                    .build();
        }
    }
  ```
  

  



