## 계좌 등록 테스트

---

- `Service` 테스트는 **==직접 DB와 연결없이 Mock 환경에서 테스트==**하면 된다.
- 실제 `AccountService` 를 띄워놓고 `@Mock` 어노테이션으로 `가짜 UserRepository` 와 `가짜 AccountRepository` 를 `AccountService`에 주입한다.
- 그리고 Service 코드에서는 **==계좌등록을 했을때 반환하는 DTO가 제대로 DTO로 전환되는지 검증==**한다.
    - 잘 변환됐는지 확인하기 때문에 `ObjectMapper` 를 주입받는다(이 때 ==**진짜 객체를 주입**==해야 하기 때문에 `@Spy` 어노테이션 사용)
- `계좌등록()` 메서드는 내부에서 `userRepository` 로 `findById()` , `accountRepository` 로 `findByNumber()`, `save()` 의 3가지 동작을 하기 때문에 stub 을 3번 동작하여 `Mokito.when()` 메서드로 가설을 3개 세워야 한다.
- 그리고 `계좌등록()` 으로 반환되는 DTO를 검증하면 된다.

    ```java
    @ExtendWith(MockitoExtension.class)
    class AccountServiceTest extends DummyObject {

        @InjectMocks
        private AccountService accountService;

        @Mock
        private UserRepository userRepository;
        @Mock
        private AccountRepository accountRepository;

        @Spy // 진짜 객체를 InjectMock 에 주입
        private ObjectMapper om;

        @Test
        public void 계좌등록_test() throws Exception {
            //given
            Long userId = 1L;

            AccountSaveReqDto dto = new AccountSaveReqDto();
            dto.setNumber(4016L);
            dto.setPassword(3427L);

            // stub 1
            User cristiano = newMockUser(userId, "cristiano", "ronaldo");
            Mockito.when(userRepository.findById(Mockito.any())).thenReturn(Optional.of(cristiano));

            // stub 2
            Mockito.when(accountRepository.findByNumber(Mockito.any())).thenReturn(Optional.empty());

            // stub 3
            Account cristianoAccount = newMockAccount(userId, 4016L, 1000L, cristiano);
            Mockito.when(accountRepository.save(Mockito.any())).thenReturn(cristianoAccount);

            //when
            AccountSaveRespDto accountSaveRespDto = accountService.계좌등록(dto, cristiano.getId());
            String responseBody = om.writeValueAsString(accountSaveRespDto);

            //then
            assertThat(accountSaveRespDto.getNumber()).isEqualTo(4016L);
        }
    }
    ```


## 계좌 목록 보기 테스트

---

- 강의종료 후 코드비교

    ```java

    @ExtendWith(MockitoExtension.class)
    class AccountServiceTest extends DummyObject {

        @InjectMocks
        private AccountService accountService;

        @Mock
        private UserRepository userRepository;
        @Mock
        private AccountRepository accountRepository;

        @Spy // 진짜 객체를 InjectMock 에 주입
        private ObjectMapper om;

        @Test
    		public void 계좌목록보기_유저별_test() throws Exception {
    		    //given
    		    Long userId = 1L;

    		    //stub 1
    		    User cristiano = newMockUser(userId, "cristiano", "ronaldo");
    		    Mockito.when(userRepository.findById(userId)).thenReturn(Optional.of(cristiano));

    		    //stub 2
    		    Account cristianoAccount1 = newMockAccount(userId, 4016L, 1000L, cristiano);
    		    Account cristianoAccount2 = newMockAccount(userId, 3427L, 1000L, cristiano);
    		    List<Account> accounts = new ArrayList<>();
    		    accounts.add(cristianoAccount1);
    		    accounts.add(cristianoAccount2);
    		    Mockito.when(accountRepository.findByUser_id(userId)).thenReturn(accounts);

    		    //when
    		    AccountListRespDto accountListRespDto = accountService.계좌목록보기_유저별(userId);

    		    //then
    		    assertThat(accountListRespDto.getAccounts().size()).isEqualTo(2);
    		}
    }
    ```


## 계좌 삭제 테스트

---

- 계좌삭제 메서드를 보면 return 값이 따로없다. 그렇기에 given 데이터를 잘못 줘서 예외가 터지는 경우를 검증하면 된다.

    ```java

    @ExtendWith(MockitoExtension.class)
    class AccountServiceTest extends DummyObject {

        @InjectMocks
        private AccountService accountService;

        @Mock
        private UserRepository userRepository;
        @Mock
        private AccountRepository accountRepository;

        @Spy // 진짜 객체를 InjectMock 에 주입
        private ObjectMapper om;

        @Test
        public void 계좌삭제_test() throws Exception {
            //given
            Long number = 1111L;
            Long userId = 2L;

            //stub
            User cristiano = newMockUser(1L, "cristiano", "ronaldo");
            Account cristianoAccount = newMockAccount(1L, 1111L, 1000L, cristiano);
            Mockito.when(accountRepository.findByNumber(Mockito.any())).thenReturn(Optional.of(cristianoAccount));

            //when
            assertThrows(CustomApiException.class, () -> accountService.계좌삭제(number, userId));
        }
    }
    ```


## 계좌 입금 테스트

---

- `계좌임금()` 은 `AccountRepository.findByNumber()` 와 `TransactionRepository.save()` 2개의 stub이 필요하다.
- 여기서 문제는 하나의 `MockUser` 와 하나의 `MockAccount` 객체를 **==2개의 stub 에서 사용하면 데이터가 꼬여질 수 있다==**.
    - `stub1` 에서 잔액이 증가한 account 객체가 stub2에 사용된다고 해보자.
    - `stub2`는 실제 계좌입금() 메서드가 동작하므로 잔액이 증가한 account 객체를 받으면 한번 더 잔액이 증가한다.
- `stub` 을 만들 때 마다 `Mock` 객체를 만들어서 데이터가 안꼬이게 해주자.

    ```java


    @ExtendWith(MockitoExtension.class)
    class AccountServiceTest extends DummyObject {

        @InjectMocks
        private AccountService accountService;

        @Mock
        private UserRepository userRepository;
        @Mock
        private AccountRepository accountRepository;

        @Spy // 진짜 객체를 InjectMock 에 주입
        private ObjectMapper om;

        @Test
        public void 계좌입금_test() throws Exception {
            //given
            AccountDepositReqDto accountDepositReqDto = new AccountDepositReqDto();
            accountDepositReqDto.setNumber(1111L);
            accountDepositReqDto.setAmount(100L);
            accountDepositReqDto.setGubun("DEPOSIT");
            accountDepositReqDto.setTel("01040163427");

            // stub 1
            User cristiano = newMockUser(1L, "cristiano", "ronaldo");
            Account account = newMockAccount(1L, 1111L, 1000L, cristiano);
            Mockito.when(accountRepository.findByNumber(Mockito.any())).thenReturn(Optional.of(account));

            // stub 2
            Account account2 = newMockAccount(1L, 1111L, 1000L, cristiano);
            Transaction transaction = newMockDepositTransaction(1L, account2);
            Mockito.when(transactionRepository.save(Mockito.any())).thenReturn(transaction);


            //when
            AccountDepositRespDto accountDepositRespDto = accountService.계좌입금(accountDepositReqDto);
            String responseBody = om.writeValueAsString(accountDepositRespDto);
            System.out.println("responseBody = " + responseBody);

            //then
            assertThat(accountDepositRespDto.getTransaction().getDepositAccountBalance()).isEqualTo(1100L);
            assertThat(account.getBalance()).isEqualTo(1100L);
        }
    }
    }

    ```


## 계좌 출금 테스트

---

이제부터 Mock을 이용한 테스트가 아닌 서비스단에 순수 로직을 테스트한다

- 일부러 잘못된 값을 넣어 오류가 제대로 일어나는지 검증했다.

    ```java
    @Test
    @DisplayName("Mock 을 이용하지 않고 순수한 로직 테스트")
    public void 계좌출금_test() throws Exception {
        //given
        Long failAmount = 0L;
        Long failUserId = 2L;
        Long failPassword = 4016L;
        Long withdrawAmount = 2000L;

        User cristiano = newMockUser(1L, "cristiano", "ronaldo");
        Account cristianoAccount = newMockAccount(1L, 1111L, 1000L, cristiano);

        //when

        //then
        assertThrows(CustomApiException.class, () -> {
            if (failAmount <= 0L) {
                throw new CustomApiException("0원 이하의 금액을 출금할 수 없습니다.");
            }
        });
        assertThrows(CustomApiException.class, () -> cristianoAccount.checkOwner(failUserId));
        assertThrows(CustomApiException.class, () -> cristianoAccount.checkSamePassword(failPassword));
        assertThrows(CustomApiException.class, () -> cristianoAccount.withdraw(withdrawAmount));

    }
    ```


## 계좌이체 테스트

---

- Mockito 가 꼭 필요한 경우가 아니라면 내부 로직이 동작하는지 테스트하는게 좋다

    ```java
    @Test
    @DisplayName("Mockito 가 아닌 로직 테스트")
    public void 계좌이체_test() throws Exception {
        //given
        AccountTransferReqDto reqDto = new AccountTransferReqDto();
        reqDto.setWithdrawNumber(1111L);
        reqDto.setDepositNumber(2222L);
        reqDto.setWithdrawPassword(3427L);
        reqDto.setAmount(100L);
        reqDto.setGubun("TRANSFER");

        User cristiano = newMockUser(1L,"cristiano","ronaldo");
        User messi = newMockUser(2L,"messi","lionel");
        Account withdrawAccount = newMockAccount(1L,1111L,1000L,cristiano);
        Account depositAccount = newMockAccount(2L,2222L,1000L,messi);

        //when
        // 출금계좌와 입금계좌가 동일한지 체크
        if(reqDto.getWithdrawNumber().equals(reqDto.getDepositNumber())){
            throw new CustomApiException("입금계좌와 출금계좌가 동일합니다");
        }
        // 이체금액이 0원인지 체크
        if(reqDto.getAmount() <= 0L){
            throw new CustomApiException("0원 이하의 금액을 이체할 수 없습니다.");
        }
        // 출금 소유자 확인
        withdrawAccount.checkOwner(1L);
        // 출금계좌 비밀번호 확인
        withdrawAccount.checkSamePassword(reqDto.getWithdrawPassword());
        // 이체하기
        withdrawAccount.withdraw(reqDto.getAmount());
        depositAccount.deposit(reqDto.getAmount());

        //then
        assertThat(withdrawAccount.getBalance()).isEqualTo(900L);
    }
    ```

## 관련 문서

- 상위 목차: [[어플리케이션 구현]]
- 이전 문서: [[AccountService 구현]]
- 다음 문서: [[TransactionController 구현 및 테스트]]
