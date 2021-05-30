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

  http에서 header에 username:password 값이 묻어가기 때문에 보안에 매우 취약합니다. 반드시 https 프로토콜에서 사용할 것을 권장하고 있습니다.

  최조 로그인시에만 인증을 처리하고, 이후에는 session에 의존합니다. 또 RememberMe를 설정한 경우, 쿠기가 브러우저에 저장되기 때문에 세션이 만료된 이후라도 브라우저 기반의 앱에서는 장시간 서비스를 로그인 페이지를 거치지 않고 이용할 수 있습니다. 
  

  테스트 방법
  ```
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    public class BasicTokenTest {

        @LocalServerPort
        int port;

        private RestTemplate restTemplate = new RestTemplate();

        @DisplayName("1. Basic Token Test")
        @Test
        void test_1(){

            String url = format("http://localhost:%d%s", port, "/greeting");
            HttpHeaders headers = new HttpHeaders();
            headers.add(HttpHeaders.AUTHORIZATION, "basic "+ Base64.getEncoder().encodeToString("user1:1111".getBytes()));
            HttpEntity entity = new HttpEntity("", headers);

            ResponseEntity<String> response = restTemplate.exchange(url,
                    HttpMethod.GET, entity, String.class);

            assertEquals("Hello jongwon", response.getBody());
        }

    }
  ```

## Session with Basic Authentication 

Basic Authentication을 이용해서 세선과 클라이언트 로그인을 진행 할 수 있다. 

스프링 인증 처리는 세션과는 별도로 동작하도록 설계되어 있습니다. 그래서 session을 사용하건 사용하지 않건 같은 authentication과 authenticationProvider를 사용할 수 있습니다. 

서버의 세션정책과 스프링의 인정 체계가 서로 맞물려 작동하려면 `SecurityContextPersistenceFilter`와 `RememberMeAuthenticationFilter`등과 여러 인증을 보조해주는 다른 필터들의 도움을 받아야 합니다. 


## **SecurityContextPersistenceFilter**

SecurityContextRepository에 저장된 SecurityContext를 request의 localThread에 넣어주었다가 뺐는 역할을 한다. doFilter 메소드를 따라가보면 알수 있다. 세션에 SecurityContext를 보관했다가 다음 request에서 넣어줍니다.
    
```
SecurityContextPersistenceFilter -> SecurityContextRepository
SecurityContextPersistenceFilter -> SecurityContextHolder

HttpSessionSecurityContextRepository -> SecurityContextRepository
```

## RememberMeAuthenticationFilter

  인증 정보를 세션관리하는 경우 , 세션 timeout이 발생하게 되면, remember-me 쿠키를 이용해 로그인을 기억했다 자동으로 재로그인 시켜주는 기능입니다.

  ```
  RememberMeAuthenticationFilter -> RememberMeServices
  
  AbstrackRememberMeService -> RememberMeService

  TokenBasedRememberMeService -> AbstrackRememberMeService
  PersistenceTokenBasedRememberMeService -> AbstrackRememberMeService

  PersistenceTokenBasedRememberMeService -> PersistenceTokenRepository
  ```

  PersistenceTokenRepository에 username, series, token,last_used 정보가 들어있습니다. 

  **토큰기반의 TokenBasedRememberMeService은 다음과 같은 특징이 존재합니다.**
   - 포맷 : 아이디 : 만료시간 : Md5Hex(아이디:만료시간:비밀번호:인증키)
   - 만약 User가 password를 바꾼다면 토큰을 쓸 수 없게 됩니다. 
   - 기본 유효기간은 14일 이고 설정에서 바꿀 수 있습니다. 
   - 약점 : 탈취된 토큰은 비밀번호를 바꾸지 않는한 유효기간동안 만능키가 됩니다. 
  
  그렇기 때문에 토큰기반의 탈취시 문제의 소지가 있습니다. 

  **PersistenceTokenBasedRememberMeServices**
   - 포멧 : series:token
   - 토큰에 username이 노출되지 않고, 만료시간도 노출되지 않습니다. 만료시간은 서버에서 정하고 노출하지 않고 서버는 로그인 시간만 저장합니다.
   - series 값이 키가 된다. 일종의 채널이라고 보면 편리하다.
   - 대신 재로그인이 될 때마다 token 값을 갱신해 줍니다. 그래서 토큰이 탈취되어 다른 사용자가 다른 장소에서 로그인을 했다면 정상 사용자가 다시 로그인 할 때, CookieTheftException 이 발생하게 되고, 서버는 해당 사용자로 발급된 모든 remember-me 쿠키값들을 삭제하고 재로그인을 요청하게 됩니다.
   - InmemoryTokenRepository 는 서버가 재시작하면 등록된 토큰들이 사라집니다. 따라서 자동로그인을 설정했더라도 다시 로그인을 해야 합니다. 재시작 후에도 토큰을 남기고 싶다면 JdbcTokenRepository를 사용하거나 이와 유사한 방법으로 토큰을 관리해야 합니다.
   - 로그아웃하게 다른 곳에 묻혀놓은 remember-me 쿠키값도 쓸모가 없게 됩니다. 만약 다른 곳에서 remember-me로 로그인한 쿠키를 살려놓고 싶다면, series 로 삭제하도록 logout 을 수정해야 합니다.



  
    

