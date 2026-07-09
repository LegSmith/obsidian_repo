암호화는 스프링 시큐리티가 제공하는 `BCryptPasswordEncoder` 클래스를 사용하자 !

- `BCryptPasswordEncoder` 클래스는 PasswordEncoder 인터페이스 구현체이다.
- encode() 메서드로 비밀번호를 암호화할 수 있다.

### 스프링 컨테이너 등록

- 스프링 시큐리티 설정파일로 가서 `@Bean` 어노테이션으로 등록
    
    ```java
    @Configuration
    @EnableWebSecurity // 스프링 시큐리티 필터가 스프링 필터체인에 등록이 된다.
    public class SecurityConfig{
    
        @Bean
        public BCryptPasswordEncoder encodePwd(){
            return new BCryptPasswordEncoder();
        }
    		
    		//생략 ..
    }
    ```
    

### encode() 사용

- 실제로는 Service 단에서 트랜잭션이 걸린채로 비밀번호를 암호화하고 DB에 저장하지만 예제를 간단히 하기 위해 컨트롤러에 구현하였다.
- Repository 는 스프링 데이터 JPA 인터페이스를 구현하여 사용
- `BCryptPasswordEncoder` 를 주입받아서 `encode()` 메서드에 암호화할 비밀번호를 파라미터로 받으면 암호화된 비밀번호로 변환한다.
    
    ```java
    @Slf4j
    @Controller
    @RequiredArgsConstructor
    public class IndexController {
    
        private final UserRepository userRepository;
        private final BCryptPasswordEncoder bCryptPasswordEncoder;
    
        @ResponseBody
        @PostMapping("/join")
        public String join(@ModelAttribute User user){
            user.setRole(UserRole.ROLE_USER);
            user.setPassword(bCryptPasswordEncoder.encode(user.getPassword()));
            userRepository.save(user);
    
            return "join";
        }
    }
    ```