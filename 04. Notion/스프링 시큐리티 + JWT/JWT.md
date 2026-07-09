**==JWT : JSON Web Token==  
**_JWT 는 암호의 목적이 아닌 서명의 목적이다 !!_

## JWT(JSON Web Token)이란 ?

---

- 서버와 클라이언트 간 ==**정보를 JSON 객체로 안전하게 전송하기 위한 간결하고 독립적인 개방형 표준(RFC 7519)**==이다.
- JWT 는 `HMAC` 알고리즘 또는 `RSA` 또는 `ECDSA` 를 사용한다.
- JWT 는 ==**서명된 토큰에 중점**==을 둔다.

## JWT 구조

---

- JWT 는 `.` 으로 구분된다.

    - `Header`
    - `Paylod`
    - `Signature`
    - `xxxxxx.yyyyyyy.zzzzz` → 이런식으로 만들어진다.

    ### Haeder

    - 헤더는 일반적으로 토큰유형과 사용되는 서명 알고리즘(`HMAC SHA256` 또는 `RSA`)으로 구성된다.

        ```json
        {
        	"alg" : "HS256",
        	"typ" : "JWT"
        }
        ```

    - 그리고 이 JSON 이 `Base64Url` 로 인코딩되어 JWT 의 첫 번째 부분을 구성한다.

    ### Paylod

    - JWT의 두 번째 부분은 클레임을 포함하는 페이로드이다. 클레임은 엔티티 및 추가 데이터에 대한 설명이다.
    - 클레임에는 registered 클레임 , public 클레임, private 클레임이 있다.

        - registered 클레임에는 `iss(발행자)` , `exp(만료시간)`, `sub(주제)` , `aud(청중)` , `nbj` , `iat(생성된 시간)` , `jti(JWT ID)` 입니다.
        - public 클레임 : JWT 를 사용하는 사람들이 원하는 대로 정의할 수 있습니다.
        - private 클레임 : 사용에 동의한 당사자 간에 정보를 공유하기 위해 생성된 맞춤 클레임

        ```json
        {
        	"sub" : "12345678",
        	"name" : "John Doe",
        	"admin" : true
        }
        ```

    - 이 JSON 도 Base64Url 로 인코딩되어 JWT 의 두 번째 부분을 구성

    ### Signature

    - Base64Url 로 인코딩된 Header , Paylod , Secret키값을 더해 Header 에 지정된 알고리즘으로 암호화를 한다.
    - 만약 HMAC SHA256 알고리즘을 사용한다면 아래와 같은 방식으로 생성된다.

        ```json
        HMACSHA256(
        	base64UrlEncode(header) + "." +
          base64UrlEncode(payload),
          secret
        )
        ```


    ### 최종 JWT

    - Header , Paylod , Signature 가 각각 Base64Url 로 인코딩 되고 `.` 으로 구분된다.

        ![[Untitled 49.png|Untitled 49.png]]


## 임시 토큰 만들어서 테스트

---

- Postman 툴로 테스트를 한다.
- 우선 테스트를 하기위한 `/token` URL 을 가진 컨트롤러를 만들자.

    ```java
    @PostMapping("/token")
    public String token(){
        return "<h1>Token</h1>";
    }
    ```

- 임의의 필터를 만들고 여기서 로직을 짠다.
- 우선 ==**Servlet 이 제공하는 Filter 는 확장성을 위해 HttpServle이 아닌 Servlet 타입의 request , response 를 받기 때문에 다운 캐스팅**==을 한다.
- 그리고 토큰은 HTTP 헤더에 Authorization 에 Value 값으로 담기때문에 postman 에서 임의로 값을 지정한다.

    ![[Untitled 1 35.png|Untitled 1 35.png]]

- 그리고 필터에서 `Authorization` 값을 비교하여 **==임의로 정한값(cos)가 아니면 컨트롤러에 진입하지 못하게막고 임의로 정한값이 맞다면 컨트롤러로 정상흐름을 하게 만들면 된다.==**

    ```java
    public class MyFilter3 implements Filter {
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest req = (HttpServletRequest) request;
            HttpServletResponse res = (HttpServletResponse) response;


            // 토큰이 만들어 졌다고 가정하자 -> 토크 : 코스
            // 실제로는 ID,PW 정상적으로 들어와서 로그인이 완료되면 JWT 토큰을 만들어주고 그걸 응답해준다.
            // 요청할 때 마다 HTTP header 에 Authorization 에 Value 값으로 JWT 토큰을 가지고 요청한다.
            // 서버에서는 JWT 토큰을 받으면 받은 JWT 토큰을 서버에서 만든 JWT 토큰인지 검증만 하면 된다 !!(RSA, HS256)
            if(req.getMethod().equals("POST")){
                String headerAuth = req.getHeader("Authorization");
                System.out.println("headerAuth = " + headerAuth);

                if(headerAuth.equals("cos")){
                    System.out.println("필터 정상 흐름");
                    chain.doFilter(req,res);
                }else {
                    PrintWriter writer = res.getWriter();
                    writer.println("인증안됨");
                }
            }
        }
    }
    ```

- 마지막으로 Security 설정파일에서 Security 필터보다 위에서 만든 필터를 우선으로 실행하게 설정하면 된다.

    ```java
    @Configuration
    @EnableWebSecurity
    @RequiredArgsConstructor
    public class SecurityConfig {

        private final CorsFilter corsFilter;

        @Bean
        SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

            http.addFilterBefore(new MyFilter3(),BasicAuthenticationFilter.class); //시큐리티 진입전에 임시토큰 검사

            http.csrf().disable();

    				// 생략...

    }
    ```


정리  
1. 실제로 ID와 PW를 클라이언트에서 서버로 요청을 하면 ID,PW를 검증한다.  
2. 정상적인 사용자로 판단하고 로그인이 완료되면 서버에서 JWT 토큰을 만들어서 응답해준다.  
3. 다음 요청부터는 클라이언트는 HTTP 헤더에 Authorization 에 Value값으로 JWT 토큰을 담아서 요청한다.  
4. 서버에서는 JWT 토큰을 검증만 해주면 된다.

## JWT 를 위한 로그인 시도

---

- `http://localhost:8080/login` 요청이 오면 `PrincipalDeatilsService` 가 실행되면서 `loadUserByUsername()` 메서드에서 로그인 로직을 실행 했다.
- 그리고 로그인이 완료되면 `PrincipalDeatils`에 담아서 세션에 저장했다.

    ```java
    @Service
    @RequiredArgsConstructor
    @Slf4j
    public class PrincipalDetailsService implements UserDetailsService {

        private final UserRepository userRepository;

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            log.debug("PrincipalDetailsService 진입");
            User userEntity = userRepository.findByUsername(username);
            return new PrincipalDetails(userEntity);
        }
    }
    ```

- 하지만 `SecurityConfig` 에서 `formLogin`을 disable 해서 `/login` 요청이 와도 호출이 안된다 !!
- **==즉, JWT를 사용하면 Security 에서 제공하는 필터를 사용해서 로그인시도를 해야한다 !!==**


### JWT 를 위한 로그인

- Security 에서 제공하는 `UsernamePasswordAuthenticationFilter` 을 상속받아서 임의의 필터를 만들어준다.
- 이 때 `AuthenticationManager` 를 주입받아야한다.

    - `AuthenticationManager` 로 로그인시도를 하면 **==PrincipalDetailsService 를 호출하면서 loadUserByUsername을 호출해 주기 때문에 주입받아서 사용해야 한다 !!==**

    ```java
    @RequiredArgsConstructor
    public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
        private final AuthenticationManager authenticationManager;

        // /login 요청을 하면 로그인 시도를 위해서 실행되는 핫무
        @Override
        public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
            System.out.println("JwtAuthenticationFilter : 로그인 시도중");
            return super.attemptAuthentication(request, response);
      }
    ```

- Security 파일에서 `.addFilter(new JwtAuthenticationFilter(authenticationManager));` 로 필터를 등록해주면 된다.


## JWT를 위한 강제 로그인 진행

---

- JWT로 로그인을 하기 위해 Security 필터 중 `UsernamePasswordAuthenticationFilter`를 상속받아서 사용하였다.
- `JwtAuthenticationFilter` 필터는 `/login` 요청이 오면 가장 먼저 `JwtAuthenticationFilter()` 메서드가 실행된다. 여기서는 로그인 인증을 구현하면 된다.

    - 우선 formLogin 이 아니기 때문에 웹 브라우저, react, 휴대폰 앱이랑 통신할 수 있다. 그렇기에 JSON 통신을 기반으로 로직을 짜야한다.
    - JSON 을 파싱하기 위해 ObjectMapper 를 사용한다.
    - 우선 **==ObjectMapper 를 이용하여 User 엔티티를 만들고 User의 username, password 를 뽑아내서 로그인 정보를 UsernamePasswordAuthenticationToken 토큰을 만든다(JWT가 아니다 !!)==**
    - 만든 토큰은 `authenticationManager` 에 담게되면 `PrincipalDetailsService` 에 `loadUserByUsername()` 메서드를 호출하여 로그인완료를 한다.
    - 그리고 **==authenticate 를 return 하면 그 때 session 에 저장==**한다.

    ```java
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        System.out.println("JwtAuthenticationFilter : 로그인 시도중");

        try {
            // 1. username, password 받아서
            ObjectMapper om = new ObjectMapper();
            User user = om.readValue(request.getInputStream(), User.class);

            // 2. 정상유저인지 authenticationManager 로 로그인 시도를 한다.
            // Token 생성
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword());

            // Token 을 AuthenticationManager 에 담는다 !
            // 이 때, PrincipalDetailsService 에 loadUserByUsername() 호출 !
            // authenticate 에 로그인정보가 담긴다.
            Authentication authenticate = authenticationManager.authenticate(authenticationToken);

            PrincipalDetails principalDetails = (PrincipalDetails) authenticate.getPrincipal();
            System.out.println("principalDetails.getUser() = " + principalDetails.getUser());

            // return 이 될때 세션에 저장된다.
            return authenticate;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

- 그리고 로그인인증이 완료되면 `successfulAuthentication()` 가 호출된다. **==이 메서드에서 JWT 토큰을 만들어서 클라이언트에게 응답==**해주면 된다 !!

    ```java
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        System.out.println("successfulAuthentication 실행됨 : 로그인 인증 완료");
        super.successfulAuthentication(request, response, chain, authResult);
    }
    ```


## JWT 토큰 생성

---

- 로그인 인증이 정상적으로 확인대면 Security 는 `UsernamePasswordAuthenticationFilter` 클래스의 `successfulAuthentication()` 메서드를 호출한다.
- `successfulAuthentication()` 메서드에서 JWT 토큰을 생성하여 ResponseHeader 에 담아 응답하면 된다 !
- 우선 파라미터인 Authentication 객체는 세션에 담긴 유저정보를 갖고오기 위해 사용한다. PrincipalDetails로 다운캐스팅하여 사용
- JWT토큰은 빌더패턴으로 create() 메서드를 호출하여 생성하면 된다.
    - `Subject()` 는 주제가 들어가므로 뭘로하든 크게 상관없다.
    - `ExpiresAt()` 은 만료시간이므로 ==**현재시간에서 ms 단위로 추가**==해주면된다(짧게 하는것이 좋다 탈취 위험때문에)
    - `Claim` 은 비공개 Claim 이므로 간단하게 id랑 username 만 넣는다.
    - `sign` 은 알고리즘 방식을 정하면 된다. ==**HMAC512 알고리즘은 서버만 아는 고유의값을 지정**==해주면 된다.
- 응답헤더에 `Authorization` 이라는 키값에 `jwtToken` 을 넣어주면 끝이다 !!

    ```java
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                                            FilterChain chain, Authentication authResult) throws IOException, ServletException {
        PrincipalDetails principalDetails = (PrincipalDetails) authResult.getPrincipal();

        System.out.println("successfulAuthentication 실행됨 : 로그인 인증 완료");

        // RSA 방식이 아닌 Hash 암호방식
        String jwtToken = JWT.create()
                .withSubject("cos 토큰") // 크게 상관없다하네요
                .withExpiresAt(new Date(System.currentTimeMillis() + (60000*10))) // 만료시간(10분)
                .withClaim("id", principalDetails.getUser().getId()) // withClaim 은 비공개 Claim 이니까 아무거나 넣어도댐 !
                .withClaim("username", principalDetails.getUser().getUsername())
                .sign(Algorithm.HMAC512("cos")); // 내 서버만 아는 고유한 값

        response.addHeader("Authorization","Bearer "+jwtToken);
    }
    ```

- postman 으로 확인하면 응답헤더에 잘 들어간게 확인된다. 이후 통신부터는 항상 헤더에 JWT가 들어가있다.

    ![[Untitled 2 23.png|Untitled 2 23.png]]


**==이후로는 JWT 를 가지고 요청,응답하기 떄문에 서버는 JWT 토큰이 유효한 토큰인지 필터를 만들어 판단해주면 된다 !!==**

## JWT 토큰 유효한지 검사

---

- `Spring Security` 가 가지고 있는 필터 중 `BasicAuthenticationFilter` 는 ==**인증이나 권한이 필요한 요청이 들어오면 호출**==된다.
- `BasicAuthenticationFilter` 로 JWT 토큰이 유효한지 체크하면 된다.
- 우선 SecurityConfig 에 필터를 추가하고 아래와 같이 userRepository 도 받아온다.
    - `.addFilter(newJwtAuthorizationFilter(authenticationManager,userRepository));`
- 코드의 가독성을 위해 인터페이스를 하나 생성했다.

    ```java
    public interface JwtProperties {
        String SECRET = "cos"; // 서버만 아는 고유의 값
        int EXPIRATION_TIME = 60000*10; // 10분
        String TOKEN_PREFIX = "Bearer ";
        String HEADER_STRING = "Authorization";
    }
    ```

- doFilterInternal 에서 로직을 짜주면 된다.

    1. 요청헤더에서 JWT 토큰값을 가져온다.
        1. **==Header 가 비어있다면 로그아웃이 아닌 doFilter 를 타게해서 시큐리티의 다음 필터를 타게하고 시큐리티는 JWT 토큰이 없는것을 확인하면 알아서 사용자를 서버내부로 못오게 한다.==**
    2. 요청헤더에 있는 JWT 토큰은 "Beareer ****" 이렇게 특정 단어와 결합해 있다. 그래서 순수한 토큰값만 꺼내온다.
    3. 꺼내온 토큰값을 서명을 해야한다.
        1. `require()` 메서드에 알고리즘과 서버만 아는 고유의 값을 갖고오고 빌더 패턴으로 ==**verify()로 뽑아온 JWT토큰을 넣고 서명**==을 한다.
        2. 서명이 되면 getClaim() 으로 JWT 토큰을 만들때 설정한 username 을 갖고온다.
    4. username 이 비어있지 않다면 서명이 제대로 된것이다.
    5. userRepository 로 username 을 통해 User 엔티티를 갖고오고 PrincipalDetails() 에 넣는다.
    6. Authentication 객체를 강제로 만들어서 PrincipalDetails 를 넣고 password 는 null 그리고 권한을 넣어준다.
    7. 그리고 ==**강제로 시큐리티 세션에 접근하여 PrincipalDetails 를 넣는다.**== → why? **==시큐리티가 권한처리를 할때 시큐리티 세션을 이용하기 때문 !!==**

    ```java
    public class JwtAuthorizationFilter extends BasicAuthenticationFilter {

        private UserRepository userRepository;

        public JwtAuthorizationFilter(AuthenticationManager authenticationManager, UserRepository userRepository) {
            super(authenticationManager);
            this.userRepository = userRepository;

        }

        // 인증이나 권한이 필요한 주소요청이 있을 때 해당 필터를 타게 된다.
        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {

            // 여기서 헤더값을 확인하면 된다 !!
            String jwtHeader = request.getHeader(JwtProperties.HEADER_STRING);
            System.out.println("jwtHeader = " + jwtHeader);

            // Header 가 있는지 확인
            if (jwtHeader == null || !jwtHeader.startsWith(JwtProperties.TOKEN_PREFIX)) {
                chain.doFilter(request, response);
                return;
            }

            // JWT 토큰 검증
            String jwtToken = request.getHeader(JwtProperties.HEADER_STRING).replace(JwtProperties.TOKEN_PREFIX, "");
            String username = JWT.require(Algorithm.HMAC512("cos")).build().
                    verify(jwtToken).
                    getClaim("username").asString();

            // 서명이 제대로 됐다.
            if(username != null){
                User userEntity = userRepository.findByUsername(username);

                PrincipalDetails principalDetails = new PrincipalDetails(userEntity);

                // JWT 토큰 서명을 통해서 정상이면 Authentication 객체를 만들어준다.
                Authentication authentication =
                        new UsernamePasswordAuthenticationToken(principalDetails,null,principalDetails.getAuthorities());

                // 강제로 시큐리티 세션에 접근하여 Authentication 객체를 저장.
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
            chain.doFilter(request, response);
        }
    }
    ```

## 관련 문서

- 상위 목차: [[스프링 시큐리티 + JWT]]
- 다음 문서: [[로그인 동작 원리와 구현]]
