# (+) 스프링 시큐리티 + JWT

## 개요

JWT 를 많이 사용하는데 보통 백엔드가 스프링으로 구성된 곳에서는 시큐리티 설정을 커스텀하게 변경해서 인증 처리를 하고 있다.(나의 경험상) 아주 간단한 형태로 스프링 시큐리티에서 약간의 추가 설정을 통해 JWT 를 이용한 인증처리를 하는 데모를 구현했다.



## 예제 코드

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {

    private static final String[] IGNORING_PATTERN = {};

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (webSecurity) -> webSecurity.ignoring().requestMatchers(IGNORING_PATTERN);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.authorizeHttpRequests(authorizationManagerRequestMatcherRegistry -> authorizationManagerRequestMatcherRegistry.anyRequest().authenticated());
        JsonWebTokenAuthenticationFilter jsonWebTokenAuthenticationFilter = new JsonWebTokenAuthenticationFilter();
        CustomAuthenticationEntryPoint customAuthenticationEntryPoint = new CustomAuthenticationEntryPoint();
        httpSecurity.addFilterBefore(jsonWebTokenAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling(httpSecurityExceptionHandlingConfigurer -> httpSecurityExceptionHandlingConfigurer.authenticationEntryPoint(customAuthenticationEntryPoint));
        return httpSecurity.build();
    }

}

```

버전이 올라가면서 자꾸 설정하는 부분이 바뀌고 자주 쓰던 것들이 Deprecated 되고 있다.

인증을 위해서 JsonWebTokenAuthenticationFilter 라는 필터를 만들어서 스프링 시큐리티 설정 내에서 필터로 추가를 해줬다. 또 에러 처리를 위해서 CustomAuthenticationEntryPoint 를 만들어서 exceptionHandling 을 하도록 설정해줬다.

```java
public class JsonWebTokenAuthenticationFilter extends OncePerRequestFilter {

    // for test
    private final List<String> VALID_TOKENS = List.of("validToken1", "validToken2", "validToken3");

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = request.getHeader(HttpHeaders.AUTHORIZATION);

        try {
            if (isInValidToken(token)) {
                throw new RuntimeException();
            }

            this.setSecurityUser(token);
        } catch (Exception exception) {
            // 아래 값은 모두 enum 혹은 상수 등으로 클린코드 원칙을 지켜줘야한다. 지금은 그냥 테스트용으로 가볍게 만든 것.
            request.setAttribute(CustomAuthenticationEntryPoint.SECURITY_ERROR_CODE_KEY, "INVALID_TOKEN");
        } finally {
            filterChain.doFilter(request, response);
        }
    }

    private boolean isInValidToken(String token) {
        return (token.isEmpty() || !this.VALID_TOKENS.contains(token));
    }

    private void setSecurityUser(String token) {
        // token 을 파싱해서 나온 값으로 식별한 userId 를 repository 를 통해 DB 조회해서 나온 user 이며 아래는 테스트용 코드
        /*
        String userId = this.tokenManager.decodeToken(token);
        User user = this.userRepository.findById(userId);
        **/
        User user = new User("userId");

        UserDetails userDetails = new org.springframework.security.core.userdetails.User(
                user.getId(),
                "",
                Collections.emptyList());
        Authentication authentication = new UserAuthenticationToken(userDetails, userDetails.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(authentication);
    }
}

```

```java
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    public static String SECURITY_ERROR_CODE_KEY = "SECURITY_ERROR_CODE_KEY";

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        String message = request.getAttribute(SECURITY_ERROR_CODE_KEY).toString();
        this.sendResponse(response, message);
    }

    private void sendResponse(HttpServletResponse response, String message) throws IOException {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(message);
    }

}

```



## 테스트

```java
@RestController
public class TestController {

    @GetMapping("/index")
    public String index() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println(authentication.getPrincipal().toString());
        return "index";
    }

}
```

```
GET http://localhost:8080/index
Accept: application/json
Authorization: validToken1

###
```

위와 같이 시도하면 아래와 같이 응답과 로그가 돌아온다.

```
HTTP/1.1 200 
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json
Content-Length: 5
Date: Thu, 15 Jun 2023 11:52:39 GMT
Keep-Alive: timeout=60
Connection: keep-alive

index
```

```
org.springframework.security.core.userdetails.User [Username=userId, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]]
```

이상한 토큰 값으로 시도하면 아래와 같은 응답을 받게 된다.

```
http://localhost:8080/index

HTTP/1.1 401 
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Content-Length: 13
Date: Thu, 15 Jun 2023 11:54:14 GMT
Keep-Alive: timeout=60
Connection: keep-alive

INVALID_TOKEN
```

설정한 상태코드와 body 값 그대로이다.
