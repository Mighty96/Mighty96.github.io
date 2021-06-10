---
title: "[Spring Security] 인증/인가 실패에 따른 리다이렉트"

categories:
  - TIL
tags:
  - Spring
  - Spring Boot
  - Spring Security
toc: true
toc_sticky: true
---

# 개요

스프링 프로젝트 도중, 아직 회원가입하지 않은 회원과 회원가입은 했으나 권한이 없는 사용자가 각각 다른 경로로 리다이렉트됐으면 좋겠다고 생각했다.

전자는 `인증(Authentication)`, 후자는 `인가(Authorization)`에 관한 내용이었다.

__인증__

- 사용자가 해당 도메인에 등록되어 있는지 확인하는 과정이라고 할 수 있다.
- 일반적인 사이트에서는 회원가입 및 로그인이 되었는지 확인하는 과정일 것이다.

__인가__

- 인증이 완료된 사용자가 해당 리소스에 접근할 권한이 있는지 확인하는 과정이라고 할 수 있다.
- 일반적인 사이트에서는 일반 USER가 ADMIN의 관리페이지에 접근할 수 없는 점을 예로 들 수 있다.

이러한 `인증`과 `인가`에 차이를 두고 처리하고 싶었다.

# AuthenticationEntryPoint

```java
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        log.error("가입되지 않은 사용자 접근");
        response.sendRedirect("/signin");
    }
}
```

`Spring Security`에서 인증되지 않은 사용자의 리소스에 대한 접근 처리는 `AuthenticationEntryPoint`가 담당한다.

인터페이스이기 때문에 `CustomAuthenticationEntryPoint`라는 이름으로 상속받아서 나만의 메서드를 구성했다.

내가 확인할 수 있도록 `인증`과 관련된 로그를 남기고 사용자를 `/signin`이라는 로그인페이지로 리다이렉트시키도록 처리했다.

# AccessDeniedHandler

```java
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        log.error("권한없는 사용자의 접근");
        response.sendRedirect("/signup_auth");
    }
}
```

`Spring Security`에서 인증되었으나 권한이 없는 사용자의 리소스에 대한 접근 처리는 `AccessDeniedHandler`가 담당한다.

마찬가지로 인터페이스이기 떄문에 `CustomAccessDeniedHandler`라는 이름으로 상속받아서 메서드를 구성했다.

현재 프로젝트의 경우 이메일 인증이 되기 전에는 유저의 권한이 `GUEST`로 제한되고 추가인증이 완료되고 나서야 `USER`라는 권한을 얻게된다.

따라서 `GUEST`가 접근한 리소스에 대한 권한이 없을 경우 `인가`와 관련된 로그를 남기고 사용자를 `/signup_auth`라는 추가인증을 진행할 수 있는 페이지로 이동시킨다.

# 구현체 적용

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                
              .exceptionHandling()
                  .accessDeniedHandler(new CustomAccessDeniedHandler())
                  .authenticationEntryPoint(new CustomAuthenticationEntryPoint());
```

`WebSecurityConfigurerAdapter`를 상속받은 `SecurityConfig`클래스에서 설정을 적용했다.  

`exceptionHandling()`을 통하여 기존 만들었던 두가지 커스텀 클래스들을 주입시켰다.