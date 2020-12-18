---
layout: post
title:  spring security
categories: security
description: spring security 的简单使用
keywords: security
---

  Spring Security是基于spring的应用程序提供声明式安全保护的安全性框架，它提供了完整的安全性解决方案，能够在web请求级别和方法调用级别
  处理身份证验证和授权．它充分使用了依赖注入和面向切面的技术。


# spring security 具体功能讲解

## UserDetails

   该接口是提供用户信息的核心接口。该接口实现仅仅存储用户的信息。后续会将该接口提供的用户信息封装到认证对象中去。
   
   该接口是实现Spring Security 认证信息的核心接口。其中 getUsername 方法为 UserDetails 接口 的方法，
   
   这个方法返回 username，也可以是其他的用户信息，例如手机号、邮箱等。
   
   getAuthorities() 方法返回的是该用户设置的权限信息
   
   
   Role 类实现了 GrantedAuthority 接口，并重写 getAuthority() 方法。权限点可以为任何字符串，不一定非要用角色名。
   
   所有的Authentication实现类都保存了一个GrantedAuthority列表，其表示用户所具有的权限。
   
   GrantedAuthority是通过AuthenticationManager设置到Authentication对象中的，然后AccessDecisionManager将从Authentication中
   
   获取用户所具有的GrantedAuthority来鉴定用户是否具有访问对应资源的权限。
   
## UserDetailsService

   Service 层需要实现 UserDetailsService 接口，该接口是根据用户名获取该用户的所有信息， 包括用户信息和权限点。
   
   loadUserByUsername 返回 UserDetails

## FilterInvocationSecurityMetadataSource

   FilterInvocationSecurityMetadataSource 的作用是用来储存请求与权限的对应关系。

   FilterInvocationSecurityMetadataSource接口有3个方法：

   1.boolean supports(Class<?> clazz)：指示该类是否能够为指定的方法调用或Web请求提供ConfigAttributes。
   
   2.Collection<ConfigAttribute> getAllConfigAttributes()：Spring容器启动时自动调用, 一般把所有请求与权限的对应关系也要在这个方法里初始化, 保存在一个属性变量里。
   
   3.Collection<ConfigAttribute> getAttributes(Object object)：当接收到一个http请求时, filterSecurityInterceptor会调用的方法. 参数object是一个包含url信息的HttpServletRequest实例. 
   
   这个方法要返回请求该url所需要的所有权限集合。

## AccessDecisionManager 

   AccessDecisionManager是由AbstractSecurityInterceptor调用的，它负责鉴定用户是否有访问对应资源（方法或URL）的权限。
   
    /**
        * 通过传递的参数来决定用户是否有访问对应受保护对象的权限
        *
        * @param auth 包含了当前的用户信息，包括拥有的权限。这里的权限来源就是前面登录时UserDetailsService中设置的authorities。
        * @param o  就是FilterInvocation对象，可以得到request等web资源
        * @param cas configAttributes是本次访问需要的权限
        */
       @Override
       public void decide(Authentication auth, Object o, Collection<ConfigAttribute> cas){
           Iterator<ConfigAttribute> iterator = cas.iterator();
           while (iterator.hasNext()) {
               ConfigAttribute ca = iterator.next();
               //当前请求需要的权限
               String needRole = ca.getAttribute();
               if ("ROLE_LOGIN".equals(needRole)) {
                   if (auth instanceof AnonymousAuthenticationToken) {
                       //throw new BadCredentialsException("未登录");
                       return;
                   } else
                       return;
               }
               //当前用户所具有的权限
               Collection<? extends GrantedAuthority> authorities = auth.getAuthorities();
               for (GrantedAuthority authority : authorities) {
                   if (authority.getAuthority().equals(needRole)) {
                       return;
                   }
               }
           }
           throw new AccessDeniedException("权限不足!");
       }
       @Override
       public boolean supports(ConfigAttribute configAttribute) {
           return true;
       }
       @Override
       public boolean supports(Class<?> aClass) {
           return true;
       }

## AbstractSecurityInterceptor 

  AbstractSecurityInterceptor 是一个实现了对受保护对象的访问进行拦截的抽象类

  AbstractSecurityInterceptor的机制可以分为几个步骤：
  
  1.查找与当前请求关联的“配置属性（简单的理解就是权限）”
  
  2.将 安全对象（方法调用或Web请求）、当前身份验证、配置属性 提交给决策器（AccessDecisionManager）
  
  3.（可选）更改调用所根据的身份验证
  
  4.允许继续进行安全对象调用(假设授予了访问权)
  
  5.在调用返回之后，如果配置了AfterInvocationManager。如果调用引发异常，则不会调用AfterInvocationManager。

 *AbstractSecurityInterceptor中的方法说明*
 
  beforeInvocation()方法实现了对访问受保护对象的权限校验，内部用到了AccessDecisionManager和AuthenticationManager；
  
  finallyInvocation()方法用于实现受保护对象请求完毕后的一些清理工作，主要是如果在beforeInvocation()中改变了SecurityContext，则在finallyInvocation()中需要将其恢复为原来的SecurityContext，该方法的调用应当包含在子类请求受保护资源时的finally语句块中。
  
  afterInvocation()方法实现了对返回结果的处理，在注入了AfterInvocationManager的情况下默认会调用其decide()方法。
  
  了解了AbstractSecurityInterceptor，就应该明白了，我们自定义MyFilterSecurityInterceptor就是想使用我们之前自定义的 AccessDecisionManager 和 securityMetadataSource。
  
## WebSecurityConfigurerAdapter

  用于指定特定的Web安全设置
  
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Autowired
      private MyUserDetailsService userService;
      
      @Autowired
      public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
  
          //校验用户
          auth.userDetailsService( userService ).passwordEncoder( new PasswordEncoder() {
              //对密码进行加密
              @Override
              public String encode(CharSequence charSequence) {
                  System.out.println(charSequence.toString());
                  return DigestUtils.md5DigestAsHex(charSequence.toString().getBytes());
              }
              //对密码进行判断匹配
              @Override
              public boolean matches(CharSequence charSequence, String s) {
                  String encode = DigestUtils.md5DigestAsHex(charSequence.toString().getBytes());
                  boolean res = s.equals( encode );
                  return res;
              }
          } );
      }
  
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http.authorizeRequests()
                  .antMatchers("/","index","/login","/login-error","/401","/css/**","/js/**").permitAll()
                  .anyRequest().authenticated()
                  .and()
                  .formLogin().loginPage( "/login" ).failureUrl( "/login-error" )
                  .and()
                  .exceptionHandling().accessDeniedPage( "/401" );
          http.logout().logoutSuccessUrl( "/" );
      }
    }


