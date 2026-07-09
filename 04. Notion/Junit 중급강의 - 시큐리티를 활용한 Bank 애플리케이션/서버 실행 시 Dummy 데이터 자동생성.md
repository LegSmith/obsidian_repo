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

## 관련 문서

- 상위 목차: [[Junit 중급강의 - 시큐리티를 활용한 Bank 애플리케이션]]
- 이전 문서: [[동적쿼리 레파지토리 테스트]]
- 다음 문서: [[스프링부트 JWT 세팅 및 구현]]
