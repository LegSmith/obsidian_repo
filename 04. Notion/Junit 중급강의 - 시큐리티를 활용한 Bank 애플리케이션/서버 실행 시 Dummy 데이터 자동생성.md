### Dummy 데이터 Init

- 서버 시작할 때마다 회원가입이 번거롭기 때문에 User 객체를 save하는 로직을 자동으로 실행한다.
    
    ```java
    @Configuration
    public class DummyDevInit extends DummyObject{
    
        @Profile("dev") // dev 모드에서만 실행
        @Bean
        CommandLineRunner init(UserRepository userRepository){
            return (args) -> {
                // 서버 실행시  무조건 실행.
                userRepository.save(newUser("cristiano", "ronaldo"));
            };
        }
    }
    ```