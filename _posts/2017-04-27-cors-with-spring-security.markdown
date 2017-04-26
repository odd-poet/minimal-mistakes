---
layout: single
title: "CORS 설정과 Spring Security"
date: 2017-04-27T02:52:33+09:00
tags: ['CORS', 'spring security']
---

API 서버와 Web FrontEnd 서버를 나누어서 구성할 때 CORS(*Cross Origin Resource Sharing*) 설정을 해야하는데,
*spring-security*를 적용 중이라면 servlet filter로 CORS를 적용하면 원치 않은 결과를 얻을 수 있다.

*spring-security*를 사용 중일 때 *CORS*를 적용하는 방법을 정리한다.

<!-- more -->

```java
@EnableWebSecurity
@Configuation
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
  @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          /*... 중략 ...*/
            .authorizeRequests()
            .requestMatchers(CorsUtils::isPreFlightRequest).permitAll() // - (1)
            /* 중략 */
            .anyRequest().authenticated().and()
            .cors().and(); // - (2)
    }

    @Bean
   public CorsConfigurationSource corsConfigurationSource() {
       CorsConfiguration configuration = new CorsConfiguration();
       // - (3)
       configuration.addAllowedOrigin("*");
       configuration.addAllowedMethod("*");
       configuration.addAllowedHeader("*");
       configuration.setAllowCredentials(true);
       configuration.setMaxAge(3600L);
       UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
       source.registerCorsConfiguration("/**", configuration);
       return source;
   }
}
```

위 코드에서 `(1)` 부분이 중요하다.

*CORS preflight* 요청은 인증처리를 하지 않겠다는 것인데,
CORS semantic 상으로 CORS prefight에는 `Authorization` 헤더를 줄 이유가 없으므로
*CORS preflight* 요청에 대해서는 `401` 응답을 하면 안된다.

이 부분의 설정이 없으면 spring-security 설정에 의해서
*CORS preflight* 요청이 정상적으로 처리되지 못해서 실제 CORS 요청이 정상적으로 이루어지지 않는다.
(*CORS preflight*에 대해서는 아래에서 설명한다)

`(2)` 부분은 spring-security에서 cors를 적용한다는 설정이다.
인증 성공 여부와 무관하게 `Origin` 헤더가 있는 모든 요청에 대해 CORS 헤더를 포함한 응답을 해준다.

`(3)`은 CORS 정책설정이다.
입맛 대로 설정하자.


### CORS preflight

사실 기본적인 내용들은 [모질라의 HTTP 접근제어(CORS)](https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS) 문서에 잘 설명되어 있다.

상기 문서에 설명되어 있듯이 브라우저는 현재 웹페이지 이외의 사이트에 xhr 요청할 때 **CORS preflight** 라는 요청을 보낸다.
이 것은 실제 해당 서버에 CORS 정책을 확인하기 위한 요청이며 `OPTIONS` 메소드를 사용하여 요청을 보낸다
(`GET /hello` 요청에 대해 `OPTIONS /hello` 요청을 *preflight*로 보낸다).

이 `OPTIONS` 요청은 웹페이지의 javascript에 의한 명시적인 요청이 아니라, 브라우저가 몰래(?) 보내는 요청이다.
이 요청의 응답으로 적절한 CORS 접근 권한을 얻지 못하면 브라우저는 그 사이트에 대한 xhr 요청을 모두 무시한다. (실제 서버응답을 javascript로 돌려주지 않는다)

*CORS*와 관련된 응답헤더들은 아래와 같다.

- `Access-Contorl-Allow-Origin`
  - CORS 요청을 허용할 사이트 (e.g. `https://oddpoet.net`)
- `Access-Contorl-Allow-Method`
  - CORS 요청을 허용할 Http Method들 (e.g. `GET,PUT,POST`)
- `Access-Contorl-Allow-Headers`
  - 특정 헤더를 가진 경우에만 CORS 요청을 허용할 경우
- `Access-Contorl-Allow-Credencial`
  - 자격증명과 함께 요청을 할 수 있는지 여부.
  - 해당 서버에서 `Authorization`로 사용자 인증도 서비스할 것이라면 `true`로 응답해야 할 것이다.
- `Access-Contorl-Max-Age`
  - preflight 요청의 캐시 시간.

References
- http://stackoverflow.com/questions/36968963/how-to-configure-cors-in-a-spring-boot-spring-security-application/37610988#37610988
- http://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html
- http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cors.html
