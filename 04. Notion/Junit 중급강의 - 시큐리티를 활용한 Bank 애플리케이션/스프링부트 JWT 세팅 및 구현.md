## 시큐리티 로그인과 회원가입 세팅

---

- 우선 UserDetailsService 인터페이스 구현체를 만들어 로그인 요청이 왔을 때 동작하는 코드
- username 으로 인증이 안될 경우 `InternalAuthenticationServiceException` 예외를 터뜨려야 한다. 왜냐하면 시큐리티가 동작할 때는 제어권이 개발자에게 없기 때문에 `CustomException` 같은 처리를 못한다.

    ```java
    @Service
    @RequiredArgsConstructor
    public class LoginService implements UserDetailsService {

        private final UserRepository userRepository;

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            User userPS = userRepository.findByUsername(username).orElseThrow(
                    () -> new InternalAuthenticationServiceException("인증 실패")
            ); // 시큐리티로 들어왔을 때는 제어권이 개발자에게 없기 때문에 InternalAuthenticationServiceException 예외 터뜨려야함

            return new LoginUser(userPS);
        }
    }
    ```

- `UserDetails` 인터페이스 구현체를 생성해 세션객체를 저장하는 코드

    ```java
    @RequiredArgsConstructor
    @Getter
    public class LoginUser implements UserDetails {

        private final User user;


        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            Collection<GrantedAuthority> authorities = new ArrayList<>();
            authorities.add(()->"ROLE_"+user.getRole());
            return authorities;
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
    }
    ```


## JWT 생성,검증 코드

---

- JWT 토큰을 이용할 때 고정되는 값이나 HS256 알고리즘을 사용할 때 서버만 아는 시크릿 값이 필요하기 때문에 인터페이스로 만들어 사용

    ```java
    public interface JwtVO {
        public static final String SECRET = "메타코딩"; // HS256 알고리즘을 위한 서버만 알고있는 키 값
        public static final int EXPIRATION_TIME = 1000 * 60 * 60 * 24 * 7; // 만료시간(ms)
        public static final String TOKEN_PREFIX = "Bearer "; // Protocol 이다
        public static final String HEADER = "Authorization";
    }
    ```

- 그리고 JWT 토큰을 생성하는 메서드와 검증하는 메서드가 들어있는 클래스를 생성

    ```java
    @Slf4j
    public class JwtProcess {

        /**
         * <h2>JWT 토큰 생성 메서드</h2>
         * @return
         */
        public static String create(LoginUser loginUser){
            String jwtToken = JWT.create()
                    .withSubject("bank")
                    .withExpiresAt(new Date(System.currentTimeMillis()+JwtVO.EXPIRATION_TIME))
                    .withClaim("id", loginUser.getUser().getId())
                    .withClaim("role", loginUser.getUser().getRole() + "")
                    .sign(Algorithm.HMAC512(JwtVO.SECRET));
            return JwtVO.TOKEN_PREFIX+jwtToken;
        }

        /**
         * <h2>JWT 토큰 검증 메서드</h2>
         * <li>return 되는 LoginUser 객체를 강제로 시큐리티 세션에 직접 주입</li>
         */
        public static LoginUser verify(String token){
            DecodedJWT decodedJWT = JWT.require(Algorithm.HMAC512(JwtVO.SECRET)).build().verify(token);

            Long id = decodedJWT.getClaim("id").asLong();
            String role = decodedJWT.getClaim("role").asString();
            User user = User.builder().
                            id(id).
                            role(UserEnum.valueOf(role)).
                            build();
            LoginUser loginUser = new LoginUser(user);
            return loginUser;

        }

    }
    ```


## JWT 필터 구현 및 등록

---

- TODO : 추후에 코드 리뷰시 작성

```java
public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private AuthenticationManager authenticationManager;

    public JwtAuthenticationFilter(AuthenticationManager authenticationManager) {
        super(authenticationManager);
        setFilterProcessesUrl("/api/login"); // 디폴트였던 /login 요청이 아닌 /api/login 요청으로 변경
        this.authenticationManager = authenticationManager;
    }

    // POST : /login 요청 시 동작
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        try {
            ObjectMapper om = new ObjectMapper();
            LoginReqDto loginReqDto = om.readValue(request.getInputStream(), LoginReqDto.class);

            // 강제 로그인을 위한 토큰
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(loginReqDto.getUsername(),loginReqDto.getPassword());

            // UserDetailsService 클래스의 loadUserByUsername() 호출(강제로그인)
            // 강제 로그인을 하는 이유는 시큐리티 설정파일에서 설정했던 권한체크의 도움을 받기 위해
            // 이 세션의 유효기간은 request -> response 하면 끝.(오직 권한체크만을 위한 세션)
            Authentication authentication = authenticationManager.authenticate(authenticationToken);

            return authentication;
        }catch (Exception e){
            // SecurityConfig 에 authenticationEntryPoint 에 걸리고,
            throw new InternalAuthenticationServiceException(e.getMessage());
        }
    }

    // attemptAuthentication() 가 잘 작동하면 호출
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        LoginUser loginUser = (LoginUser) authResult.getPrincipal();
        String jwtToken = JwtProcess.create(loginUser);

        response.addHeader(JwtVO.HEADER, jwtToken);

        LoginRespDto loginRespDto = new LoginRespDto(loginUser.getUser());

        CustomResponseUtil.success(response,loginRespDto);
    }
}
```

- 필터 등록은 Security 가 권장하는 방법으로 한다.
- 생성자로 필요한 `AuthenticationManager` 는 `getSharedObject()` 메서드를 호출하여 `AuthenticationManager` 타입을 매개변수로 넣어서 강제로 생성한 뒤 JWT 필터 생성자에 주입한다.

    ```java
    @Slf4j
    @Configuration
    public class SecurityConfig {

    		// 생략 ...

        // 공식 문서에 따라 모든 Filter 는 이 메서드에서 등록해야 한다.
        public class CustomSecurityFilterManager extends AbstractHttpConfigurer<CustomSecurityFilterManager,HttpSecurity>{
            @Override
            public void configure(HttpSecurity builder) throws Exception {
                // 강제로 AuthenticationManager 를 만든다.
                AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);
                builder.addFilter(new JwtAuthenticationFilter(authenticationManager));
                super.configure(builder);
            }
        }

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    				// 생략 ...

            // Filter 적용
            http.apply(new CustomSecurityFilterManager());

    				// 생략 ...

            return http.build();
        }
    		// 생략 ...
    }
    ```

- 로그인 실패 시 동작하는 `unsuccessfulAuthentication()` 도 JWT 필터에서 오버라이딩 해줘야 한다.

    ```java
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        CustomResponseUtil.unAuthentication(response,"로그인 실패 !");
    }
    ```


## JWT 검증 필터 구현 및 등록

---

시큐리티에서 모든 요청이 오면 동작하는 필터는 `BasicAuthenticationFilter` 이다. 이 필터에서 JWT 토큰검증을 하면된다.

- 가장 먼저 **==클라이언트가 로그인이 성공한 상태에서 JWT 토큰을 HTTP 요청 헤더에 담은 상태==**로 `/api/admin` 요청을 했다고 가정하자.
    - `/api/admin` : SecurityConfig 파일에서 해당요청이 오면 ADMIN 권한을 가지고 있는 요청만 허가하도록 설정했다.
- `isHeaderVerify()` 메서드는 HTTP 요청 헤더에 JWT 토큰을 가지고 있는 키가 비어있지 않고 `Bearer` 로 시작하는 지 체크하는 메서드이다.
- 만약 요청헤더에 JWT 토큰이 있다면 아래와 같은 로직이 돈다.
    1. `Bearer` 문자열을 JWT 토큰에서 잘라내어 순수 JWT 토큰을 가져온다.
    2. `verify()` 메서드에서 JWT 토큰을 검증한다.

        - 토큰검증 메서드인 verify() 는 토큰 검증 후 id와 role 만 가져와 LoginUser 객체를 반환한다.( id 와 role 만 가지는 이유는 권한체크만 하면 되기 때문이다.)

        ```java
        public static LoginUser verify(String token){
            DecodedJWT decodedJWT = JWT.require(Algorithm.HMAC512(JwtVO.SECRET)).build().verify(token);

            Long id = decodedJWT.getClaim("id").asLong();
            String role = decodedJWT.getClaim("role").asString();
            User user = User.builder().
                            id(id).
                            role(UserEnum.valueOf(role)).
                            build();
            LoginUser loginUser = new LoginUser(user);
            return loginUser;

        }
        ```

    3. 그리고 시큐리티 세션에 강제주입하기 위해 Authentication 객체를 만든다.
    4. 강제주입을 하고 인가가 완료된다. 그다음 doFilter() 로 여러 필터통과후 Controller 로 진입한다. 이 때 시큐리티 세션에 강제 주입한 객체의 role 을 보고 /api/admin 요청을 수행한다.
- **==즉, 오직 권한체크만을 위해 시큐리티 세션에 id와 role 만 가지는 객체를 저장한 것이다.==**

    ```java
    public class JwtAuthorizationFilter extends BasicAuthenticationFilter {

        public JwtAuthorizationFilter(AuthenticationManager authenticationManager) {
            super(authenticationManager);
        }

        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
            if (isHeaderVerify(request,response)){
                // JWT 토큰이 존재함
                String token = request.getHeader(JwtVO.HEADER).replace(JwtVO.TOKEN_PREFIX, "");
                LoginUser loginUser = JwtProcess.verify(token);

                // 임시 세션(UserDetails 타입 or username, 비밀번호, 유저의 권한을 파라미터로 받는다)
                Authentication authentication =
                        new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());

                // 강제 주입
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
            chain.doFilter(request,response);
        }

        private boolean isHeaderVerify(HttpServletRequest request, HttpServletResponse response){
            String header = request.getHeader(JwtVO.HEADER);

            return header != null && header.startsWith(JwtVO.TOKEN_PREFIX);
        }
    }
    ```


## 시큐리티 + JWT 개념잡기

---

![[Untitled 15.png|Untitled 15.png]]

## 관련 문서

- 상위 목차: [[Junit 중급강의 - 시큐리티를 활용한 Bank 애플리케이션]]
- 이전 문서: [[서버 실행 시 Dummy 데이터 자동생성]]
- 다음 문서: [[시큐리티 세팅]]
