---
layout: post
title: 시큐리티 원리 - 5
subtitle: "spring, security"
categories: spring
tags: security
comments: true
---
> spring study



## 권한의 이해 

  Authorization은 filter를 거친후에 url단에서 권한을 검사할수있으며 , Global Method 권한 위원회에 의해서 컨드롤러,서비스, 레포지터리단을 애노테이션기반으로 검사할수있다.

  SecurityFilterChain 당 한개의 filterSecurityIntercetor를 둘 수 있고, 각 SecurityInterceptor당 한개의 AccessDecisionManager를 둘 수 있습니다. 반면 Method 권한 판정을 global 한 권한 위원회를 둡니다. 

  
  ```
  SecurityInterceptor -> AccessDecisionManager -> AccessDecisionVoter -> pass / AccessDeny
  ```
  과정으로 진행된다. 
  
## voter
  이 과정에서 기존의 방법으로는 voter의 방법이 사용되고 더 나은 방법으로는 expression 방법이 존재한다. 

  voter는 MethodSecurityConfiguration에 GlobalMethodSecurityConfiguration을 구현체로 사용해서 accesDecisionManager()를 구현해야한다.   
  accesDecisionManager()에서 decision을 결정할수있다.   
  PreInvocationAuthorizationAdviceVoter, RoleVoter, AuthenticatedVoter 에서 voter가 작동하고 커스텀한 voter또한 등록하여 사용할수있다. 

  MethodSecurityExpressionHandler를 통해서 permissionEvaluator를 핸들러에 추가할수있고 
  핸들러의 타입은 DefaultMethodSecurityExpressionHandler을 가지며 이를 생성하기 위해서는 new CustomMethodSecurityExpressionRoot(authentication, invocation);을 이용해야 가능하다. 
  
  그리고 난후 handler.setPermissionEvaluator(permissionEvaluator); 클래스에 다양한 검사를 넣을수있다.


  ```
  @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        DefaultMethodSecurityExpressionHandler handler = new DefaultMethodSecurityExpressionHandler(){
            @Override
            protected MethodSecurityExpressionOperations createSecurityExpressionRoot(Authentication authentication, MethodInvocation invocation) {
                CustomMethodSecurityExpressionRoot root = new CustomMethodSecurityExpressionRoot(authentication, invocation);
                root.setPermissionEvaluator(getPermissionEvaluator());
                return root;
            }
        };
        handler.setPermissionEvaluator(permissionEvaluator);
        return handler;
    }
  ```

## ExpressionVoter와 PIIAVoter
  WebExpressionVoter, PreInvocationAuthoriztionAdviceVoter는 SpEL 방식으로 동작한다. 
  

  ```
  @PostAuthorize("hasPermission(#paperId, 'paper', 'read')")
  @PostAuthorize("returnObject.studentIds.contains(principal.username)")
  ```

  `hasPermission`메서드를 통해서 검사가 가능하며 SpEL을 사용해야한다. #,@등으로 SpEL을 진행할수있다. 
  returnObject을 통해서 Expression을 통해서 검사가 진행된다. Expression은 MethodSecurityExpressionOperations에서 메서드를 재정의를 통해서 사용할 수 있다.
  
  AccessDecisionManager에서 AbstractAccessDecisionManager에서 AccessDecisionVoter를 통해서 voter가 진행되고 이를 통해서 Attribute가 진행되고 post,webExpress,preInvocation 등에서 postAuthorize,PostFilter등을 사용할 수 있다. 

  


