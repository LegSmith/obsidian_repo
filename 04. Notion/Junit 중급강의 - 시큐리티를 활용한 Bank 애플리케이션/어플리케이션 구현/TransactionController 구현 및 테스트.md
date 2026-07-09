## 입출금내역 컨트롤러 및 테스트

---

- ResponseEntity를 return 하는 다른 방식이다. 가독성이 더 좋다.
    
    ```java
    @RequestMapping("/api")
    @RequiredArgsConstructor
    @RestController
    public class TransactionController {
        private final TransactionService transactionService;
    
        @GetMapping("/s/account/{number}/transaction")
        public ResponseEntity<?> findTransactionList(@PathVariable Long number,
                                                     @RequestParam(value = "gubun", defaultValue = "ALL") String gubun,
                                                     @RequestParam(value = "page", defaultValue = "0") Integer page,
                                                     @AuthenticationPrincipal LoginUser loginUser){
    
            TransactionListRespDto transactionListRespDto =
                    transactionService.입출금목록보기(loginUser.getUser().getId(), number, gubun, page);
    
    //        return new ResponseEntity<>(new ResponseDto<>(1,"입출금목록보기 성공",transactionListRespDto), HttpStatus.OK);
            return ResponseEntity.ok().body(new ResponseDto<>(1,"입출금목록보기 성공",transactionListRespDto));
    
        }
    }
    ```
    
- 반환되는 JSON 코드에 있는 값을 검증하면 된다.
    
    ```java
    @Sql("classpath:db/teardown.sql")
    @ActiveProfiles("test")
    @AutoConfigureMockMvc
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    class TransactionControllerTest extends DummyObject {
        @Autowired
        private MockMvc mvc;
    
        @Autowired
        private UserRepository userRepository;
    
        @Autowired
        private TransactionRepository transactionRepository;
    
        @Autowired
        private AccountRepository accountRepository;
    
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
        public void findTransactionList_test() throws Exception {
            //given
            Long number = 1111L;
            String gubun = "ALL";
            String page = "0"; // 실제 컨트롤러에서 쿼리파라미터로 들어오는 값들은 전부 String 이기 때문.
    
            //when
            ResultActions resultActions =
                    mvc.perform(MockMvcRequestBuilders.get("/api/s/account/" + number + "/transaction").param("gubun", gubun).param("page", page));
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