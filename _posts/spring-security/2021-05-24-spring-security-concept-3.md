---
layout: post
title: 시큐리티 원리 - 3
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study

## Authentication

  인증은 다음과 같이 구성되어있다.   
  그리고 인증을 구성하기 위해서 사용되는 토큰이 존재한다. 

  - Authentication
    - Credentiails
    - Principal
    - Details
    - Authorities

  여러개의 토큰을 사용해서 인증을 구성한다. 
  - UsernamePasswordAuthentication Token
  - RunAsUser Token
  - TestingAuthentication Token
  - AnonymouseAuthenication Token
  - RememberMeAuthentication Token


## AuthenticationProvider
  Authentication을 검증하여 Principal을 제공합니다. 

  ```
  public class StudentManager implements AuthenticationProvider, InitializingBean {

    private HashMap<String, Student> studentDB = new HashMap<>();

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        StudentAuthenticationToken token = (StudentAuthenticationToken) authentication;
        if(studentDB.containsKey(token.getCredentials())){
            Student student = studentDB.get(token.getCredentials());
            return StudentAuthenticationToken.builder()
                    .principal(student)
                    .details(student.getUsername())
                    .authenticated(true)
                    .authorities(student.getRole())
                    .build();
        }
        return null;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication == StudentAuthenticationToken.class;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Set.of(
                new Student("hong", "홍길동", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT"))),
                new Student("kang", "강아지", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT"))),
                new Student("rang", "호랑이", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT")))
        ).forEach(s->
            studentDB.put(s.getId(), s)
        );
    }   
    }

  ```

  authenticationProvider를 커스텀해서 사용할수있다. 이를 만들기 위해서 student, StudentAuthenticationToken이 필요하다.

  student는 사용자의 데이터를 받을 클래스,StudentAuthenticationToken 정보를 담을 클래스이다.   
  StudentAuthenticationToken은 즉 userDetails가 되는것이다. 

  이를 구현하고 securityConfig에서 사용하면된다. 그리고 
  ```
   public class CustomLoginFilter extends UsernamePasswordAuthenticationFilter {

    public CustomLoginFilter(AuthenticationManager authenticationManager){
        super(authenticationManager);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        String username = obtainUsername(request);
        username = (username != null) ? username : "";
        username = username.trim();
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        String type = request.getParameter("type");
        if(type == null || !type.equals("teacher")){
            // student
            StudentAuthenticationToken token = StudentAuthenticationToken.builder()
                    .credentials(username).build();
            return this.getAuthenticationManager().authenticate(token);
        }else{
            // teacher
            TeacherAuthenticationToken token = TeacherAuthenticationToken.builder()
                    .credentials(username).build();
            return this.getAuthenticationManager().authenticate(token);
        }
    }
    }
  ```
  아래와 같이 filter 또한 직접 커스텀해서 사용가능하다. 


  
  