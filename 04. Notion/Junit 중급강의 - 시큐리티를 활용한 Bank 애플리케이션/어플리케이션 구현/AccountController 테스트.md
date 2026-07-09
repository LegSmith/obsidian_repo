## 계좌등록 컨트롤러 테스트

---

- 계좌등록 컨트롤러를 테스트할 때는 **==클라이언트가 로그인을 하여 인증에 통과했다는 가정==**하에 테스트를 해야 한다.
- 하지만 테스트를 할때 JWT 토큰을 만들거나 헤더에 `Authorization` 에 담아야 하는데 그럴 필요 없이 `@WithUserDetails()` 어노테이션을 쓰면 간편해진다.
    - `@WithUserDetails()`
        - `value` : DB 에서 value 값을 조회하여 시큐리티 세션에 담아준다.
        - `setupBefore = TestExecutionEvent.TEST_METHOD` : @BeforeEach 로 실행대는 메서드 이전에 실행
        - `setupBefore = TestExecutionEvent.TEST_EXECUTION` : 해당 테스트 메서드 실행 이전에 실행
- 하지만 위 어노테이션을 사용할려면 JWT 인증필터인 `BasicAuthenticationFilter` 에서 로직을 **==HTTP 요청 헤더에 JWT 토큰이 없으면 그냥 doFilter() 를 타도록 로직==**을 짜야한다.

    ```java
    public class JwtAuthorizationFilter extends BasicAuthenticationFilter {

        public JwtAuthorizationFilter(AuthenticationManager authenticationManager) {
            super(authenticationManager);
        }

        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
            if (isHeaderVerify(request,response)){
    						// 생략 ...
            }
            chain.doFilter(request,response);
        }
    }
    ```

- 그래서 `saveAccount_test()` 실행 전에 **==DB 에 있는 객체를 조회하여 시큐리티 세션에 담아준다.==** 그러면 로그인한 상태에서 Account 를 등록할 수 있다.
- given 절에 DTO를 생성해 requestBody 로 만들어주고, when 절에서 `ResultActions` 를 이용하여 `/api/s/account` 로 계좌등록 요청을 하면 된다.
- 그리고 실제 응답으로 오는 HTTP 상태코드를 검증하면 된다.

    ```java
    @Transactional
    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class AccountControllerTest extends DummyObject {

        @Autowired
        private MockMvc mvc;

        @Autowired
        private UserRepository userRepository;

        @Autowired
        private ObjectMapper om;

        @BeforeEach
        public void setUp(){
            userRepository.save(newUser("cristiano","ronaldo"));
        }

        @WithUserDetails(value = "cristiano", setupBefore = TestExecutionEvent.TEST_EXECUTION)
        @Test
        public void saveAccount_test() throws Exception {
            //given
            AccountSaveReqDto dto = new AccountSaveReqDto();
            dto.setNumber(9999L);
            dto.setPassword(1234L);

            String requestBody = om.writeValueAsString(dto);
            System.out.println("requestBody = " + requestBody);

            //when
            ResultActions resultActions = mvc.perform(post("/api/s/account").content(requestBody).contentType(MediaType.APPLICATION_JSON));

            String responseBody = resultActions.andReturn().getResponse().getContentAsString();
            System.out.println("responseBody = " + responseBody);

            //then
            resultActions.andExpect(MockMvcResultMatchers.status().isCreated());
        }
    }
    ```


## 계좌 목록보기 컨트롤러 테스트

---

```java
@Transactional
@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
class AccountControllerTest extends DummyObject {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ObjectMapper om;

    @BeforeEach
    public void setUp(){
        userRepository.save(newUser("cristiano","ronaldo"));
    }

    // TODO Junit 중급강의 종료 후 코드 비교하기
    @WithUserDetails(value = "cristiano", setupBefore = TestExecutionEvent.TEST_EXECUTION)
    @Test
    public void findUserAccount_test() throws Exception {
        //given

        //when
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/api/s/account/login-user"));
        String responseBody = resultActions.andReturn().getResponse().getContentAsString();
        System.out.println("responseBody = " + responseBody);

        //then
        resultActions.andExpect(MockMvcResultMatchers.status().isOk());

    }
}
```

## 계좌 삭제하기 컨트롤러 테스트

---

- 계좌 삭제후 삭제한 계좌번호를 조회하여 제대로 예외가 터지는지 검증

    ```java
    @Transactional
    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class AccountControllerTest extends DummyObject {

        @Autowired
        private MockMvc mvc;

        @Autowired
        private UserRepository userRepository;

        @Autowired
        private AccountRepository accountRepository;

        @Autowired
        private EntityManager em;

        @Autowired
        private ObjectMapper om;

        @BeforeEach
        public void setUp() {
            User cristiano = userRepository.save(newUser("cristiano", "ronaldo"));
            User messi = userRepository.save(newUser("messi", "lionel"));
            accountRepository.save(newAccount(1111L, cristiano));
            accountRepository.save(newAccount(2222L, messi));
            em.clear();
        }

        /**
         * <h2>Test 시 영속화컨텍스트를 초기화 해야 하는 이유!!</h2>
         * <li>테스트시에는 insert 한 것들이 영속화컨텍스트에 남아 있다.</li>
         * <li>개발 모드와 동일한 환경에으로 테스트 할려면 영속화 컨텍스트를 비워야 한다.</li>
         *
         * @throws Exception
         */
        @WithUserDetails(value = "cristiano", setupBefore = TestExecutionEvent.TEST_EXECUTION)
        @Test
        public void deleteAccount_test() throws Exception {
            //given
            Long number = 1111L;

            //when
            ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.delete("/api/s/account/" + number));
            String responseBody = resultActions.andReturn().getResponse().getContentAsString();
            System.out.println("responseBody = " + responseBody);

            //then
            assertThrows(CustomApiException.class, () -> accountRepository.findByNumber(number).orElseThrow(
                                                            () -> new CustomApiException("계좌를 찾을 수 없습니다.")));

        }
    }
    ```


## 계좌 입금 컨트롤러 테스트

---

- `Controller` 테스트는 거의 같다. 상태코드만 잘 검증해주면 된다.

    ```java
    @Transactional
    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class AccountControllerTest extends DummyObject {

        @Autowired
        private MockMvc mvc;

        @Autowired
        private UserRepository userRepository;

        @Autowired
        private AccountRepository accountRepository;

        @Autowired
        private EntityManager em;

        @Autowired
        private ObjectMapper om;

        @BeforeEach
        public void setUp() {
            User cristiano = userRepository.save(newUser("cristiano", "ronaldo"));
            User messi = userRepository.save(newUser("messi", "lionel"));
            accountRepository.save(newAccount(1111L, cristiano));
            accountRepository.save(newAccount(2222L, messi));
            em.clear();
        }

        @Test
        public void depositAccount_test() throws Exception {
            //given
            AccountDepositReqDto accountDepositReqDto = new AccountDepositReqDto();
            accountDepositReqDto.setNumber(1111L);
            accountDepositReqDto.setAmount(100L);
            accountDepositReqDto.setGubun("DEPOSIT");
            accountDepositReqDto.setTel("01040163427");

            String requestBody = om.writeValueAsString(accountDepositReqDto);
            System.out.println("requestBody = " + requestBody);

            //when
            ResultActions resultActions = mvc.perform
                    (MockMvcRequestBuilders.post("/api/account/deposit").content(requestBody).contentType(MediaType.APPLICATION_JSON));

            String responseBody = resultActions.andReturn().getResponse().getContentAsString();
            System.out.println("responseBody = " + responseBody);

            //then
            resultActions.andExpect(MockMvcResultMatchers.status().isCreated());
        }
    }
    ```


## 계좌 출금 컨트롤러 테스트

---

```java
@Transactional
@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
class AccountControllerTest extends DummyObject {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private EntityManager em;

    @Autowired
    private ObjectMapper om;

    @BeforeEach
    public void setUp() {
        User cristiano = userRepository.save(newUser("cristiano", "ronaldo"));
        User messi = userRepository.save(newUser("messi", "lionel"));
        accountRepository.save(newAccount(1111L, cristiano));
        accountRepository.save(newAccount(2222L, messi));
        em.clear();
    }

    @WithUserDetails(value = "cristiano", setupBefore = TestExecutionEvent.TEST_EXECUTION)
    @Test
    public void withdrawAccount_test() throws Exception {
        //given
        AccountWithdrawReqDto reqDto = new AccountWithdrawReqDto();
        reqDto.setNumber(1111L);
        reqDto.setPassword(3427L);
        reqDto.setAmount(100L);
        reqDto.setGubun("WITHDRAW");

        String requestBody = om.writeValueAsString(reqDto);
        System.out.println("responseBody = " + requestBody);

        //when
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.
                post("/api/s/account/withdraw").content(requestBody).contentType(MediaType.APPLICATION_JSON));
        String responseBody = resultActions.andReturn().getResponse().getContentAsString();
        System.out.println("responseBody = " + responseBody);

        //then
        resultActions.andExpect(MockMvcResultMatchers.status().isCreated());
    }
}
```

## 계좌 이체 컨트롤러 테스트

---

```java
@Transactional
@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
class AccountControllerTest extends DummyObject {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private EntityManager em;

    @Autowired
    private ObjectMapper om;

    @BeforeEach
    public void setUp() {
        User cristiano = userRepository.save(newUser("cristiano", "ronaldo"));
        User messi = userRepository.save(newUser("messi", "lionel"));
        accountRepository.save(newAccount(1111L, cristiano));
        accountRepository.save(newAccount(2222L, messi));
        em.clear();
    }

    @WithUserDetails(value = "cristiano", setupBefore = TestExecutionEvent.TEST_EXECUTION)
    @Test
    public void transferAccount_test() throws Exception {
        //given
        AccountTransferReqDto reqDto = new AccountTransferReqDto();
        reqDto.setWithdrawNumber(1111L);
        reqDto.setDepositNumber(2222L);
        reqDto.setWithdrawPassword(3427L);
        reqDto.setAmount(100L);
        reqDto.setGubun("TRANSFER");

        String requestBody = om.writeValueAsString(reqDto);
        System.out.println("requestBody = " + requestBody);

        //when
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.
                post("/api/s/account/transfer").content(requestBody).contentType(MediaType.APPLICATION_JSON));
        String responseBody = resultActions.andReturn().getResponse().getContentAsString();
        System.out.println("responseBody = " + responseBody);

        //then
        resultActions.andExpect(MockMvcResultMatchers.status().isCreated());
    }
}
```

## 계좌 상세보기 컨틀로러 테스트

---

```java

@Sql("classpath:db/teardown.sql")
@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
class AccountControllerTest extends DummyObject {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private TransactionRepository transactionRepository;

    @Autowired
    private EntityManager em;

    @Autowired
    private ObjectMapper om;

    @BeforeEach
    public void setUp() {
        dataSetting();
        em.clear();
    }

		@WithUserDetails(value = "ssar", setupBefore = TestExecutionEvent.TEST_EXECUTION)
    @Test
    public void findDetailAccount_test() throws Exception {
        //given
        Long number = 1111L;
        String page = "0"; // 실제 컨트롤러에서 쿼리파라미터로 들어오는 값들은 전부 String 이기 때문.

        //when
        ResultActions resultActions =
                mvc.perform(MockMvcRequestBuilders.get("/api/s/account/" + number)
                        .param("page", page));
        String responseBody = resultActions.andReturn().getResponse().getContentAsString();
        System.out.println("responseBody = " + responseBody);

        //then
        resultActions.andExpect(MockMvcResultMatchers.jsonPath("$.data.transactions[0].balance").value(900L));
        resultActions.andExpect(MockMvcResultMatchers.jsonPath("$.data.transactions[1].balance").value(800L));
        resultActions.andExpect(MockMvcResultMatchers.jsonPath("$.data.transactions[2].balance").value(700L));
        resultActions.andExpect(MockMvcResultMatchers.jsonPath("$.data.transactions[3].balance").value(800L));
    }

    private void dataSetting() {
        User ssar = userRepository.save(newUser("ssar", "쌀"));
        User cos = userRepository.save(newUser("cos", "코스,"));
        User love = userRepository.save(newUser("love", "러브"));
        User admin = userRepository.save(newUser("admin", "관리자"));

        Account ssarAccount1 = accountRepository.save(newAccount(1111L, ssar));
        Account cosAccount = accountRepository.save(newAccount(2222L, cos));
        Account loveAccount = accountRepository.save(newAccount(3333L, love));
        Account ssarAccount2 = accountRepository.save(newAccount(4444L, ssar));

        Transaction withdrawTransaction1 = transactionRepository
                .save(newWithdrawTransaction(ssarAccount1, accountRepository));
        Transaction depositTransaction1 = transactionRepository
                .save(newDepositTransaction(cosAccount, accountRepository));
        Transaction transferTransaction1 = transactionRepository
                .save(newTransferTransaction(ssarAccount1, cosAccount, accountRepository));
        Transaction transferTransaction2 = transactionRepository
                .save(newTransferTransaction(ssarAccount1, loveAccount, accountRepository));
        Transaction transferTransaction3 = transactionRepository
                .save(newTransferTransaction(cosAccount, ssarAccount1, accountRepository));
    }
}
```

## 관련 문서

- 상위 목차: [[어플리케이션 구현]]
- 이전 문서: [[AccountController 구현]]
- 다음 문서: [[AccountService 구현]]
