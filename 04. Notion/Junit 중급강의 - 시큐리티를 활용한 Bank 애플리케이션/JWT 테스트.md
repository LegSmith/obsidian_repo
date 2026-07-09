## JwtProcess 테스트

---

- 토큰생성 메서드인 `create()` 는 로그인객체를 파라미터로 필요하기 때문에 로그인 객체를 만들어서 반환되는 JWT 토큰의 앞글자인 `Bearer` 만 검증하면 된다.

    ```java
    class JwtProcessTest {

        @Test
        public void create_test() throws Exception {
            //given
            User user = User.builder().id(1L).role(UserEnum.CUSTOMER).build();
            LoginUser loginUser = new LoginUser(user);

            //when
            String jwtToken = JwtProcess.create(loginUser);
            System.out.println(jwtToken);
            //then
            assertThat(jwtToken.startsWith(JwtVO.TOKEN_PREFIX)).isTrue();
        }
    }
    ```

- 위에서 테스트 했던 JWT 토큰을 복사해서 검증하고 `Id` 와 `Role` 만 검증하면 된다.

    ```java
    class JwtProcessTest {
        @Test
        public void verify_test() throws Exception {
            //given
            String jwtToken = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJiYW5rIiwicm9sZSI6IkNVU1RPTUVSIiwiaWQiOjEsImV4cCI6MTcwMjAxOTgxMX0.H5KaW5an6nfVyG99wcXYOkmVq2w8FLeePh-gp66wIlccCavydg4T8mrvbx14i82Zgd2Ell5hc9kCDOIa5eCHbw";

            //when
            LoginUser loginUser = JwtProcess.verify(jwtToken);

            //then
            assertThat(loginUser.getUser().getId()).isEqualTo(1);
            assertThat(loginUser.getUser().getRole()).isEqualTo(UserEnum.CUSTOMER);
        }
    }
    ```


## JwtAuthenticationFilter 테스트

---

- `JwtAuthenticationFilter` 는 `/api/login` 요청이 오면 동작하는 시큐리티 필터이기 때문에 1건의 데이터 세팅이 필요하다.
- ResultActions 를 이용하여 세팅을하고 ==**응답이 잘 되는지 Http 상태코드를 검증**==하고, ==**jwtToken 이 null**== 이 아닌지, ==PREFIX 값이 Bearer== 이 맞는지 ==**jsonPath 를 통해 응답바디의 데이터를 확인**==해야 한다.
- 실패 테스트는 given 절에 오는 데이터를 잘못된 값을 넣고 테스트해서 401 코드가 응답하는지 검증하면 된다.

    ```java
    import static org.junit.jupiter.api.Assertions.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
    import static shop.metacoding.bank.dto.user.UserReqDto.*;

    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class JwtAuthenticationFilterTest extends DummyObject {

        @Autowired
        private ObjectMapper om;
        @Autowired
        private MockMvc mvc;
        @Autowired
        private UserRepository userRepository;

        @BeforeEach
        void setUp() throws Exception{
            userRepository.save(newUser("ssar", "쌀"));
        }

        @Test
        public void successfulAuthentication_test() throws Exception {
            //given
            LoginReqDto loginReqDto = new LoginReqDto();
            loginReqDto.setUsername("ssar");
            loginReqDto.setPassword("1234");

            String requestBody = om.writeValueAsString(loginReqDto);
            System.out.println("requestBody = " + requestBody);

            //when
            ResultActions resultActions = mvc.perform(
                    post("/api/login").
                    content(requestBody).
                    contentType(MediaType.APPLICATION_JSON));

            String responseBody = resultActions.andReturn().getResponse().getContentAsString();
            String jwtToken = resultActions.andReturn().getResponse().getHeader(JwtVO.HEADER);

            //then
            resultActions.andExpect(status().isOk());
            assertNotNull(jwtToken);
            assertTrue(jwtToken.startsWith(JwtVO.TOKEN_PREFIX));
            resultActions.andExpect(jsonPath("$.data.username").value("ssar"));

        }

    		@Test
        public void unsuccessfulAuthentication_test() throws Exception {
            //given
            LoginReqDto loginReqDto = new LoginReqDto();
            loginReqDto.setUsername("ssar");
            loginReqDto.setPassword("123434");

            String requestBody = om.writeValueAsString(loginReqDto);
            System.out.println("요청 데이터 requestBody = " + requestBody);

            //when
            ResultActions resultActions = mvc.perform(
                    post("/api/login").
                            content(requestBody).
                            contentType(MediaType.APPLICATION_JSON));
            String responseBody = resultActions.andReturn().getResponse().getContentAsString();
            System.out.println("응답 데이터 responseBody : " + responseBody);

            //then
            resultActions.andExpect(status().isUnauthorized());
        }
    }
    ```


## JwtAuthorizationFilter 테스트

---

- authorization_success , fail , admin 으로 3가지로 나누었따.
- `success` 에서는 임의의 ==**LoginUser 객체를 만들고 JWT 토큰을 생성**==하여 인증이 필요한 `/api/s` 로 get 요청을 해서 **==HTTP 상태코드가 제대로 응답하는지 검증==**했다.
- `fail` 테스트에서는 **==아무런 객체를 만들지않고(즉, 로그인을 안한다는 뜻)==** `/api/s` 로 시작하는 주소로 **==get 요청을 할때 401 응답인지 검증==**
- `admin` 테스트에서는 **==customer 권한을 가진 LoginUser 객체==**로 `/api/admin` 으로 요청해서 **==403 응답인지 검증==**하였다.

    ```java
    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class JwtAuthorizationFilterTest {

        @Autowired
        private MockMvc mvc;

        @Test
        public void authorization_success_test() throws Exception {
            //given
            User user = User.builder().id(1L).role(UserEnum.CUSTOMER).build();
            LoginUser loginUser = new LoginUser(user);
            String jwtToken = JwtProcess.create(loginUser);
            System.out.println("jwtToken = " + jwtToken);

            //when
            ResultActions resultActions = mvc.perform(get("/api/s/hello/test").header(JwtVO.HEADER,jwtToken));

            //then
            resultActions.andExpect(status().isNotFound());
        }

        @Test
        public void authorization_fail_test() throws Exception {
            //given

            //when
            ResultActions resultActions = mvc.perform(get("/api/s/hello/test"));

            //then
            resultActions.andExpect(status().isUnauthorized());
        }

        @Test
        public void authorization_admin_test() throws Exception {
            //given
            User user = User.builder().id(1L).role(UserEnum.CUSTOMER).build();
            LoginUser loginUser = new LoginUser(user);
            String jwtToken = JwtProcess.create(loginUser);
            System.out.println("jwtToken = " + jwtToken);

            //when
            ResultActions resultActions = mvc.perform(get("/api/admin/hello/test").header(JwtVO.HEADER,jwtToken));

            //then
            resultActions.andExpect(status().isForbidden());
        }

    }
    ```

## 관련 문서

- 상위 목차: [[Junit 중급강의 - 시큐리티를 활용한 Bank 애플리케이션]]
- 이전 문서: [[CORS 테스트]]
- 다음 문서: [[Long 타입 테스트]]
