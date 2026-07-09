## 구글 로그인 - 준비단계

- 구글에 google api 콘솔을 검색후 들어간다.

    > [!info] Google Cloud console  
    >  
    > [https://console.cloud.google.com/apis/library?hl=ko&pli=1](https://console.cloud.google.com/apis/library?hl=ko&pli=1)  

- 프로젝트가 없다면 만들고 좌측에 API 및 서비스 배너를 클릭후 OAuth 동의 화면으로 이동

    ![[Untitled 48.png|Untitled 48.png]]

- 필수로 입력해야 하는것들만 입력 후 저장

    ![[Untitled 1 34.png|Untitled 1 34.png]]

- 사용자 인증 정보 메뉴를 들어와서 +사용자 인증 정보 만들기 클릭

    ![[Untitled 2 22.png|Untitled 2 22.png]]

- 승인된 리다이렉션 URI 는 고정이다. 구글 로그인이 성공하면 인증코드를 아래 URI 로 받는다

    ![[Untitled 3 14.png|Untitled 3 14.png]]

- 아이디 비번은 노출안되게 잘 주의하자 !

    클라이언트 아이디 :
    클라이언트 비밀번호 : 

    ![[Untitled 4 12.png|Untitled 4 12.png]]


## 페이스북 로그인 - 세팅+yml 파일 설정

- 로그인 or 회원가입

    > [!info] Meta for Developers  
    > Facebook for Developers와 사용자를 연결하는 코드 인공 지능, 비즈니스 도구, 게임, 오픈 소스, 게시, 소셜 하드웨어, 소셜 통합, 가상 현실 등 다양한 주제를 둘러보세요.  
    > [https://developers.facebook.com/?locale=ko_KR&biz_login_source=bizweb_unified_login_fb_login_button](https://developers.facebook.com/?locale=ko_KR&biz_login_source=bizweb_unified_login_fb_login_button)  

- 우측 상단에 '내 앱' 클릭

    ![[Untitled 5 11.png|Untitled 5 11.png]]

- 앱만들기 클릭

    ![[Untitled 6 9.png|Untitled 6 9.png]]

- 기타 클릭

    ![[Untitled 7 6.png|Untitled 7 6.png]]

- 빈칸 채우고 앱만들기 클릭

    ![[Untitled 8 3.png|Untitled 8 3.png]]

- 페이스북 로그인에 설정 클릭

    ![[Untitled 9 3.png|Untitled 9 3.png]]

- 웹을 누르고 프로젝트 URL 을 입력

    ![[Untitled 10 3.png|Untitled 10 3.png]]

- 앱 설정 → 기본 설정에서 앱 ID 와 앱 시크릿 코드를 가져온다.

    앱 아이디 : ``

    앱 시크릿 코드 : ``

- application.yml 파일에 가서 구글 로그인때와 같은 구조로 만든뒤 scope 만 변경해준다.(문서참고)

    > [!info] Web - Facebook Login - Documentation - Meta for Developers  
    > Facebook Login with the Facebook SDK for JavaScript enables people to sign into your web page with their Facebook credentials.  
    > [https://developers.facebook.com/docs/facebook-login/web/#loginbutton](https://developers.facebook.com/docs/facebook-login/web/#loginbutton)  


## 네이버 로그인 - 세팅 + yml 파일 설정

- 네이버 개발자 센터로 이동

    > [!info] NAVER Developers  
    > 네이버 오픈 API들을 활용해 개발자들이 다양한 애플리케이션을 개발할 수 있도록 API 가이드와 SDK를 제공합니다.  
    > [https://developers.naver.com/main/](https://developers.naver.com/main/)  

- 상단 메뉴에 Application - > 애플리케이션 등록 이동

    ![[Untitled 11 2.png|Untitled 11 2.png]]

- 애플리케이션 이름과 사용 API 는 네이버 로그인을 고른뒤 애플리케이션으로 가져올 정보를 클릭한다. 환경은 PC웹

    ![[Untitled 12 2.png|Untitled 12 2.png]]

- 서비스 URL 은 애플리케이션 URL 을 적고 CallBack URL 은 application.yml 파일에서 적은 `redirect-uri` 와 똑같이 적어주면 된다.

    ![[Untitled 13 2.png|Untitled 13 2.png]]

- 등록을 하면 클라이언트 ID 와 클라이언트 Secret 이 나온다.
    - Client ID : 
    - Client Secret : 
- naver 는 구글이나 페이스북과는 달리 provider가 제공되지 않기 때문에 코드가 좀 달라진다.


---

OAuth2 로그인 동작 흐름  
1. `/loginForm` 에서 `/oauth2/authorization/google` 경로가 담긴 a태그를 만들어 구글로 이동  
2. 구글 로그인차에서 구글계정으로 로그인  
3. 로그인 완료 후 인증 Code를 구글로부터 받는다.(`OAuth -Client` 라이브러리가 받아준다)  
4. 인증코드를 가지고 구글에 `AccessToken` 을 요청한다.  
5. `UserRequest` 정보를 받는다  
6. 정보로 회원프로필을 받는다(`loadUser` 메서드가 로직을 처리한다)

## OAuth 로그인 후 처리

---

- 우선 OAuth 로그인 처리를 위한 시큐리티 세팅과 `application.yml` 파일 세팅이 필요하다

    ```yaml
    spring:
    	security:
    	  oauth2:
          client:
            registration:
              google:
                client-id: 
                client-secret: 
                scope:
                  - email
                  - profile
    ```

- `principalOauth2UserService` 클래스에서 로그인 후처리 로직이 담긴다.

    ```java
    @Configuration
    @EnableWebSecurity // 스프링 시큐리티 필터가 스프링 필터체인에 등록이 된다.
    @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true) // secured 어노테이션 활성화, preAuthorize,postAuthorize 어노테이션 활성화
    public class SecurityConfig {

        @Autowired
        PrincipalOauth2UserService principalOauth2UserService;

        @Bean
        SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http.csrf().disable();
            http.authorizeRequests()
    						//생략 ...
                .oauth2Login()
                .loginPage("/loginForm")
                // 로그인이 완료되면 후처리가 필요하다. (엑세스 토큰 + 사용자프로필 정보)를 받는다.
                .userInfoEndpoint()
                .userService(principalOauth2UserService);

            return http.build();
        }
    }
    ```

- 파라미터로 넘어오는 OAuth2UserRequest 에 사용자 정보가 담긴다.

    ```java
    @Service
    public class PrincipalOauth2UserService extends DefaultOAuth2UserService {

        // 구글 로그인 후 후처리가 여기서 일어난다.
        @Override
        public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
            System.out.println("userRequest.getClientRegistration() = " + userRequest.getClientRegistration());
            System.out.println("userRequest.getAccessToken() = " + userRequest.getAccessToken());
            System.out.println("super.loadUser(userRequest).getAttributes() = " + super.loadUser(userRequest).getAttributes());
            return super.loadUser(userRequest);
        }
    }
    ```

    - `getClientRegistration()` 의 정보는 아래와 같다.

        ```bash
        userRequest.getClientRegistration() 
        	= ClientRegistration{
        		registrationId='google', 
        		clientId='303612447473-dkg9tgb5ip9ekf27n93u0qm8vbgl6diq.apps.googleusercontent.com', 
        		clientSecret='비밀번호', 
        		clientAuthenticationMethod=org.springframework.security.oauth2.core.ClientAuthenticationMethod@4fcef9d3, 
        		authorizationGrantType=org.springframework.security.oauth2.core.AuthorizationGrantType@5da5e9f3, 
        		redirectUri='{baseUrl}/{action}/oauth2/code/{registrationId}', 
        		scopes=[email, profile], 
        		providerDetails=org.springframework.security.oauth2.client.registration.ClientRegistration$ProviderDetails@1cd5b7e9, 
        		clientName='Google'}
        ```

    - `getAttributes()` 는 사용자 정보를 가져온다.

        - `Map<String, Obejct>` 타입으로 데이터가 저장되어 있다.

        ```bash
        super.loadUser(userRequest).getAttributes() = 
        	{
        		sub=114824701293627045410, // google의 PK(Primary Key)
        		name=Cristiano, 
        		given_name=Cristiano, 
        		picture=https://lh3.googleusercontent.com/a/ACg8ocJYZSxHEHsth5CjKFVeqm1mLKreq11wJmAIAjwU-Bu6nQ=s96-c, 
        		email=a01040163427@gmail.com, 
        		email_verified=true, 
        		locale=ko
        }
        ```


## Authentication 객체로 로그인정보를 가져오기

---

==**로그인을 하면 UserDetails 객체에 유저정보가 담기고 Authentication 객체로 감싸져서 세션에 저장**==된다. 그래서 컨트롤러에서는 `Authentication` 객체로 사용자 정보를 다루면 된다.

- Authentication 객체에서 `getPrincipal()`로 불러오면 Object 타입이기 때문에 PrincipalDetails 로 타입을 바꾸면 PrincipalDetails 객체를 만들 수 있다.
    - `PrincipalDetails` 는 `UserDetails` 의 구현체다.

        ```java
        @GetMapping("/test/login")
        public @ResponseBody String loginTest(Authentication authentication){
            System.out.println("===========/test/login============");
            PrincipalDetails principalDetails = (PrincipalDetails)authentication.getPrincipal();
            System.out.println("principalDetails.getUser() = " + principalDetails.getUser());
            System.out.println("==================================");

            return "세션 정보 확인하기";
        }
        ```

- `@AuthenticationPrincipal` 어노테이션을 사용하면 바로 PrincipalDetails 타입으로 파라미터를 받을 수 있다.

    ```java
    @GetMapping("/test/login")
        public @ResponseBody String loginTest(@AuthenticationPrincipal PrincipalDetails userDetails){
            System.out.println("userDetails.getUsername() = " + userDetails.getUser());
            return "세션 정보 확인하기";
        }
    ```

    ### OAuth2 로그인 정보

    - **==OAuth2 로 로그인을하면 UserDetails 의 구현체인 PrinciaplDetails 로 캐스팅을 하면 안된다 !!==**
    - OAuth2User 타입으로 받아야 한다.
    - OAuth2User 의 객체정ㅂ도를 담아오는 함수는 `getAttributes()` 이다.

        ```java
        @GetMapping("/test/oauth/login")
        public @ResponseBody String oauthLoginTest(@AuthenticationPrincipal OAuth2User oAuth2User){
            System.out.println("===========/test/oauth/login============");
            System.out.println("principalDetails.getUser() = " + oAuth2User.getAttributes());
            return "OAuth 세션 정보 확인하기";
        }
        ```


즉, Security Session 에는 **==Authentication 객체로 감싸져 있는 사용자 정보가 있는데 이 객체 안에 들어올 수 있는 타입은 UserDetails 와 OAuth2User 타입이다.==**


### 문제점 !!

- 컨트롤러에서는 일반로그인(UserDetails) 인지 SNS 로그인(OAuth2User) 인지 구분하기 어렵다 !!!!
- **==PrincipalDetails 클래스에 UserDetails 와 OAuth2User 인터페이스를 받아서 하나의 구현체로 만들면 된다 !!!==**

## PrincipalDetails 에서 UserDetails 와 OAuth2User를 받는다.

---

- `OAuth2User` 의 메서드를 오버라이딩한다.
- 컨트롤러에서 OAuth2 로그인과 일반로그인을 구현하기위해 생성자로 구분한다.
- OAuth2 가 AccessToken 으로 사용자 정보를 받아오는 `getAttributes()` 는 `Map<String, Object>` 타입이기 떄문에 같은 타입의 필드를 만들고, **==User 와 Map<String, Object> 를 받는 생성자==**를 만든다.

    ```java
    @Data
    public class PrincipalDetails implements UserDetails, OAuth2User {

        private User user;
        private Map<String, Object> attributes;

        // 일반 로그인
        public PrincipalDetails(User user) {
            this.user = user;
        }

        // OAuth 로그인
        public PrincipalDetails(User user, Map<String, Object> attributes) {
            this.user = user;
            this.attributes = attributes;
        }

        // OAuth2User Overriding
        @Override
        public Map<String, Object> getAttributes() {
            return attributes;
        }

        @Override
        public String getName() {
            return null;
        }

    		//생략...
    }
    ```

- 이제 OAuth2 로그인 후 후처리가 일어나는 loadUser() 메서드를 짜면 된다.
- 여기서는 oAuth2User 가 사용자의 정보를 담고 있다. 사용자의 정보로 User엔티티에 필드값을 채워넣는다.
- 이미 회원가입을 했으면 User 엔티티를 save 하고 PrincipalDetails의 OAuth2 전용 생성자를 return 해주면 `@AuthenticationPrincipal` 가 생성된다(정확히는 메서드가 종료된 후 생성)

    ```java
    @RequiredArgsConstructor
    @Service
    public class PrincipalOauth2UserService extends DefaultOAuth2UserService {

        private final UserRepository userRepository;

        // 구글 로그인 후 후처리가 여기서 일어난다.
        @Override
        public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
            System.out.println("userRequest.getClientRegistration() = " + userRequest.getClientRegistration());
            System.out.println("userRequest.getAccessToken() = " + userRequest.getAccessToken());

            OAuth2User oAuth2User = super.loadUser(userRequest);
            System.out.println("oAuth2User.getAttributes() = " + oAuth2User.getAttributes());


            //회원가입 로직
            String provider = userRequest.getClientRegistration().getClientId();
            String providerId = oAuth2User.getAttribute("sub");
            String username = provider + "_" + providerId;
            BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
            String password = bCryptPasswordEncoder.encode("겟인데어");
            String email = oAuth2User.getAttribute("email");
            String role = "ROLE_USER";

            // 회원가입 여부
            User userEntity = userRepository.findByUsername(username);
            if(userEntity == null){
                userEntity = User.builder()
                        .username(username)
                        .password(password)
                        .email(email)
                        .role(role)
                        .provider(provider)
                        .providerId(providerId)
                        .build();
                userRepository.save(userEntity);
            }else{

            }

            return new PrincipalDetails(userEntity,oAuth2User.getAttributes());
        }
    }
    ```


## 페이스북 로그인 구현

---

**==페이스북과 구글에서 넘어오는 유저정보의 키값들은 조금씩 다르다.==**

- `getClientRegistrantion()` 의 키는 똑같다.

    ```bash
    userRequest.getClientRegistration() = ClientRegistration{
    	registrationId='facebook', 
    	clientId='691566239611692', 
    	clientSecret='31fb0f2781e8bf1b7ee355c63254d8d5', 
    	clientAuthenticationMethod=org.springframework.security.oauth2.core.ClientAuthenticationMethod@86baaa5b, 
    	authorizationGrantType=org.springframework.security.oauth2.core.AuthorizationGrantType@5da5e9f3, 
    	redirectUri='{baseUrl}/{action}/oauth2/code/{registrationId}', 
    	scopes=[email, public_profile], 
    	providerDetails=org.springframework.security.oauth2.client.registration.ClientRegistration$ProviderDetails@12ab0c9d, 
    	clientName='Facebook'}
    ```

- `getAttributes()` 의 내용이 다르다. 구글에서는 usb 라는 키값의 id가 있지만 페이스북은 id라는 키값의 id가 들어가있다.

    ```bash
    oAuth2User.getAttributes() = {
    	id=2612025632293940, 
    	name=최재영, 
    	email=chlwodud0327@naver.com
    }
    ```

- 해결방법은 인터페이스를 만들어서 google, facebook, naver 로그인 때 사용할 구현체를 만들어서 sns 마다 알맞은 키값으로 User객체를 강제 회원가입 시키면 된다.
- 먼저 ==**OAuth2UserInfo 라는 인터페이스를 만들고 각 sns 마다 scope가 다르기 때문에 구분을 위한 메서드를 만들어 놓는다.**==

    ```java
    public interface OAuth2UserInfo {
        String getProviderId();
        String getProvider();
        String getEmail();
        String getName();
    }
    ```

- 먼저 구글전용 구현제

    - attributes 필드는 OAuth2 로그인을 할때 유저정보가 담겨서 오는 OAuth2User 객체의 getAttributes() 를 가져오기 위한 필드다.
    - 그리고 구글의 id값은 sub 라는 키의 담아서 오기 때문에 구분하면 된다.

    ```java
    public class GoogleUserInfo implements OAuth2UserInfo{

        private Map<String, Object> attributes; // getAttributes() 를 받는다.

        public GoogleUserInfo(Map<String, Object> attributes) {
            this.attributes = attributes;
        }

        @Override
        public String getProviderId() {
            return (String)attributes.get("sub");
        }

        @Override
        public String getProvider() {
            return "google";
        }

        @Override
        public String getEmail() {
            return (String)attributes.get("email");
        }

        @Override
        public String getName() {
            return (String)attributes.get("name");
        }
    }
    ```

- 페이스북 전용 구현체

    - 페이스북의 id값은 id라는 키의 담아서 오기때문에 구분해주면 된다.

    ```java
    public class FacebookUserInfo implements OAuth2UserInfo{

        private Map<String, Object> attributes; // getAttributes() 를 받는다.

        public FacebookUserInfo(Map<String, Object> attributes) {
            this.attributes = attributes;
        }

        @Override
        public String getProviderId() {
            return (String)attributes.get("id");
        }

        @Override
        public String getProvider() {
            return "facebook";
        }

        @Override
        public String getEmail() {
            return (String)attributes.get("email");
        }

        @Override
        public String getName() {
            return (String)attributes.get("name");
        }
    }
    ```

- 이제 OAuth2 로그인 로직을 담당하는 PrincipalOauth2UserService 코드만 수정해주면 된다.

    ```java
    @RequiredArgsConstructor
    @Service
    public class PrincipalOauth2UserService extends DefaultOAuth2UserService {

        private final UserRepository userRepository;

        // 구글 로그인 후 후처리가 여기서 일어난다.
        @Override
        public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
            System.out.println("userRequest.getClientRegistration() = " + userRequest.getClientRegistration());
            System.out.println("userRequest.getAccessToken() = " + userRequest.getAccessToken());

            OAuth2User oAuth2User = super.loadUser(userRequest);
            System.out.println("oAuth2User.getAttributes() = " + oAuth2User.getAttributes());

            OAuth2UserInfo oAuth2UserInfo = null;
            //회원가입 로직
            if(userRequest.getClientRegistration().getRegistrationId().equals("google")){
                System.out.println("구글 로그인 요청");
                oAuth2UserInfo = new GoogleUserInfo(oAuth2User.getAttributes());
            }else if(userRequest.getClientRegistration().getRegistrationId().equals("facebook")){
                System.out.println("페이스북 로그인 요청");
                oAuth2UserInfo = new FacebookUserInfo(oAuth2User.getAttributes());
            }else{
                System.out.println("우리는 구글과 페이스북만 지원해요");
            }

            String provider = oAuth2UserInfo.getProvider();
            String providerId = oAuth2UserInfo.getProviderId();
            String username = provider + "_" + providerId;
            BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
            String password = bCryptPasswordEncoder.encode("겟인데어");
            String email = oAuth2UserInfo.getEmail();
            String role = "ROLE_USER";

            // 회원가입 여부
            User userEntity = userRepository.findByUsername(username);
            if(userEntity == null){
                userEntity = User.builder()
                        .username(username)
                        .password(password)
                        .email(email)
                        .role(role)
                        .provider(provider)
                        .providerId(providerId)
                        .build();
                userRepository.save(userEntity);
            }else{

            }

            return new PrincipalDetails(userEntity,oAuth2User.getAttributes());
        }
    }
    ```


## 네이버 로그인 구현

---

OAuth2-Client 는 Provider(구글, 페이스북, 트위터 등등)을 지원한다. 하지만 네이버나 카카오는 우리나라에서 주로사용하는 플랫폼이므로 OAuth2-Client가 어떤 데이터를 넘겨주는지 알 수 없다. 그렇기 떄문에 네이버는 Provider가 없다.

- application.yml 에서도 네이버의 provider를 따로 만들어줘야 한다.

    ```yaml
    spring:
      security:
        oauth2:
          client:
            registration:
              naver:
                client-id: OuXxM8IJRud2Ib4y9yZR
                client-secret: 3r2BNA03H5
                scope:
                  - name
                  - email
                client-name: Naver
                authorization-grant-type: authorization_code
                redirect-uri: http://localhost:8080/login/oauth2/code/naver

            provider:
              naver:
                authorization-uri: https://nid.naver.com/oauth2.0/authorize
                token-uri: https://nid.naver.com/oauth2.0/token
                user-info-uri: https://openapi.naver.com/v1/nid/me
                user-name-attribute: response # 회원정보를 JSON 으로 받는데 response 라는 키값으로 네이버가 리턴해줌.
    ```

- 네이버에서 넘어오는 getAttributes() 는 많이 다르다.

    - response 안에 사용자 정보가 담아져서 온다.

    ```bash
    oAuth2User.getAttributes() = {
    	resultcode=00, 
    	message=success, 
    	response={
    		id=68BzVGkdLw-rT9uRw9tVoOrn44cpOBecKj28gCycwms, 
    		email=chlwodud0327@naver.com, 
    		name=최재영
    	}
    }
    ```

- Map 안에 또 Map 으로 담겨져 있기 때문에 SNS를 분기하여 구현체를 생성할때 한번더 (Map)으로 감싼 객체에 response 부분만 빼와서 구현체에 attributes 에 보내줄 수 있다.

    ```java
    OAuth2UserInfo oAuth2UserInfo = null;
    //회원가입 로직
    if(userRequest.getClientRegistration().getRegistrationId().equals("google")){
        System.out.println("구글 로그인 요청");
        oAuth2UserInfo = new GoogleUserInfo(oAuth2User.getAttributes());
    }else if(userRequest.getClientRegistration().getRegistrationId().equals("facebook")){
        System.out.println("페이스북 로그인 요청");
        oAuth2UserInfo = new FacebookUserInfo(oAuth2User.getAttributes());
    }else if(userRequest.getClientRegistration().getRegistrationId().equals("naver")){
        System.out.println("네이버 로그인 요청");
        oAuth2UserInfo = new NaverUserInfo((Map)oAuth2User.getAttributes().get("response"));
    }
    ```

- 그렇게 되면 OAuth2UserInfo 인터페이스의 네이버전용 구현체에서 attribute 에 response 데이터가 담아지게 된다.

    ```java
    public class NaverUserInfo implements OAuth2UserInfo{

        // 	response={id=68BzVGkdLw-rT9uRw9tVoOrn44cpOBecKj28gCycwms,email=chlwodud0327@naver.com,name=최재영}
        private Map<String, Object> attributes;


        public NaverUserInfo(Map<String, Object> attributes) {
            this.attributes = attributes;
        }

        @Override
        public String getProviderId() {
            return (String)attributes.get("id");
        }

        @Override
        public String getProvider() {
            return "naver";
        }

        @Override
        public String getEmail() {
            return (String)attributes.get("email");
        }

        @Override
        public String getName() {
            return (String)attributes.get("name");
        }
    }
    ```


## 개념정리

---

OAuth2-Client 는 Provider(구글, 페이스북, 트위터 등등)이 있다. 단, 네이버나 카카오는 없다.

## 관련 문서

- 상위 목차: [[스프링 시큐리티 + JWT]]
- 이전 문서: [[비밀번호 암호화 해서 DB 에 저장하기]]
- 다음 문서: [[스프링 시큐리티 설정파일]]
