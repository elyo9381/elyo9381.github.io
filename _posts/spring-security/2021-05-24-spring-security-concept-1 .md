---
layout: post
title: 시큐리티 원리 - 1
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

# security
  
  spring은 웹서버로 많이 사용된다.   
  이때 클라이언트로 부터 request를 받고 서버(spring)은 response를 준다.

  서버가 reqeust를 스프링을 request응 다양한 filter로 확인을 하고 필터에 해당하지 않으면 에러를 발송한다.   
  이때 스프링이서 필터들을 관리하고 다양한 필터가 설정되는곳이 스프링 시큐리티이다. 

  그렇다면 어떠한 필터가 있을까?

  - HeaderWriterFilter : Http 해더를 검사한다. 써야 할 건 잘 써있는지, 필요한 해더를 더해줘야 할 건 없는가?
  - CorsFilter : 허가된 사이트나 클라이언트의 요청인가?
  - CsrfFilter : 내가 내보낸 리소스에서 올라온 요청인가?
  - LogoutFilter : 지금 로그아웃하겠다고 하는건가?
  - UsernamePasswordAuthenticationFilter : username / password 로 로그인을 하려고 하는가? 만약 로그인이면 여기서 처리하고 가야 할 페이지로 보내 줄께.
  - ConcurrentSessionFilter : 여거저기서 로그인 하는걸 허용할 것인가?
  - BearerTokenAuthenticationFilter : Authorization 해더에 Bearer 토큰이 오면 인증 처리 해줄께.
  - BasicAuthenticationFilter : Authorization 해더에 Basic 토큰을 주면 검사해서 인증처리 해줄께.
  - RequestCacheAwareFilter : 방금 요청한 request 이력이 다음에 필요할 수 있으니 캐시에 담아놓을께.
  - SecurityContextHolderAwareRequestFilter : 보안 관련 Servlet 3 스펙을 지원하기 위한 필터라고 한다.(?)
  - RememberMeAuthenticationFilter : 아직 Authentication 인증이 안된 경우라면 RememberMe 쿠키를 검사해서 인증 처리해줄께
  - AnonymousAuthenticationFilter : 아직도 인증이 안되었으면 너는 Anonymous 사용자야
  - SessionManagementFilter : 서버에서 지정한 세션정책을 검사할께.
  - ExcpetionTranslationFilter : 나 이후에 인증이나 권한 예외가 발생하면 내가 잡아서 처리해 줄께
  - FilterSecurityInterceptor : 여기까지 살아서 왔다면 인증이 있다는 거니, 니가 들어가려고 하는 request 에 들어갈 자격이 있는지 그리고 리턴한 결과를 너에게 보내줘도 되는건지 마지막으로 내가 점검해 줄께
  - 그 밖에... OAuth2 나 Saml2, Cas, X509 등에 관한 필터들도 있습니다.
  
  등의 다양한 필터가 존재한다. 


## authentication
  
  필터와 인증제공자를 통해서 SecurityContextHolder에 인증이 만들어진다. 

  - SecurityContextHolder : 인증보관함 보관소
    - Authentication : 인증
      - Principal(UserDetails) : 인증대상
        - GrantedAuthority : 권한

  - AuthenticationManager : 인증관리자
    - ProviderManager : 인증 제공 관리자 
      - AuthenticationProvider : 인증제공자
  
  security는 위와 같이 구성되어있다고 볼수있다.
  
  - 로그인을 위한 토큰을 제공하는 필터는 다음과 같다. 
    - UsernamePasswordAuthenticationFilter : 폼 로그인 -> UsernamePasswordAuthenticationToken
    - RememberMeAuthenticationFilter : remember-me 쿠키 로그인 -> RememberMeAuthenticationToken
    - AnonymousAuthenticationFilter : 로그인하지 않았다는 것을 인증함 -> AnonymousAuthenticationToken
    - SecurityContextPersistenceFilter : 기존 로그인을 유지함(기본적으로 session 을 이용함)
    - BearerTokenAuthenticationFilter : JWT 로그인
    - BasicAuthenticationFilter : ajax 로그인 -> UsernamePasswordAuthenticationToken
    - OAuth2LoginAuthenticationFilter : 소셜 로그인 -> OAuth2LoginAuthenticationToken, OAuth2AuthenticationToken
    - OpenIDAuthenticationFilter : OpenID 로그인
    - Saml2WebSsoAuthenticationFilter : SAML2 로그인
  - Authentication을 제공하는 인증제공자는 여러개가 동시에 존재할 수있고, 인증 방식에 따라 providerManager도 복수로 존재할수있다. 

  - Authentication 은 인터페이스로 아래와 같은 정보들을 갖고 있습니다.
    - Set<GrantedAuthority> authorities : 인증된 권한 정보
    - principal : 인증 대상에 관한 정보. 주로 UserDetails 객체가 옴
    - credentials : 인증 확인을 위한 정보. 주로 비밀번호가 오지만, 인증 후에는 보안을 위해 삭제함.
    - details : 그 밖에 필요한 정보. IP, 세션정보, 기타 인증요청에서 사용했던 정보들.
    - boolean authenticated : 인증이 되었는지를 체크함.
    
   