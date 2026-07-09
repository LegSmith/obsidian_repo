## 계좌 등록 구현

---

- 계좌등록할 때 필요한 건 `계좌번호`와 `비밀번호`, 그리고 `userId` 가 필요하다.
- 계좌번호, 비밀번호는 DTO로 받으면 되고 userId는 유저검증과 계좌주인이 필요하기 때문에 파라미터로 받는다.
- DTO는 클래스단에 두고 추후에 리팩토링할 때 DTO 패키지로 옮겨준다.
- `**동작순서**`

    1. `userId`로 올바른 유저인지 조회한다.
    2. **==계좌번호,비밀번호가 들어있는 DTO로 중복된 계좌가 아닌지 조회==**한다.
    3. 위 조건이 다맞으면 **==계좌를 등록하고 등록한 계좌를 DTO로 응답==**해준다.

    ```jsx
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;

        @Transactional
        public AccountSaveRespDto 계좌등록(AccountSaveReqDto dto, Long userId){
            // User 검증
            User userPS = userRepository.findById(userId).orElseThrow(
                    () -> new CustomApiException("유저를 찾을 수  없습니다.")
            );
            // 중복계좌 검증
            Optional<Account> accountOP = accountRepository.findByNumber(dto.getNumber());
            if(accountOP.isPresent()){
                throw new CustomApiException("계좌 중복!");
            }
            // 계좌 등록
            Account accountPS = accountRepository.save(dto.toEntity(userPS));
            // DTO 응답
            return new AccountSaveRespDto(accountPS);

        }

        @Data
        public static class AccountSaveRespDto{
            private Long id;
            private Long number;
            private Long balance;

            public AccountSaveRespDto(Account account) {
                this.id = account.getId();
                this.number = account.getNumber();
                this.balance = account.getBalance();
            }
        }

        @Data
        public static class AccountSaveReqDto{
            @NotNull
            @Digits(integer = 4, fraction = 4) // @Size 는 String 만
            private Long number;
            @NotNull
            @Digits(integer = 4, fraction = 4)
            private Long password;

            public Account toEntity(User user){
                return Account.builder().
                        number(number).
                        password(password).
                        balance(1000L).
                        user(user).build();
            }
        }

    }
    ```


## 계좌 목록 보기

---

- 계좌목록보기 API 스펙은 fullName 와 가지고있는 계좌목록(id, 계좌번호, 잔액)을 반환한다.
- 이 때 계좌를 가지고 있는 유저를 찾기위해 JPA 쿼리 메서드를 작성한다.

    ```java
    public interface AccountRepository extends JpaRepository<Account, Long> {
        // JPA Query Method ( SELECT * FROM Account a WHERE a.User.userId = :id )
        List<Account> findByUser_id(Long id);
    }
    ```

- 그리고 반환할 DTO를 작성한다. 여기서는 컬렉션이 포함되어 있으므로 2개의 DTO를 이너클래스로 만들었다.
- 우선 `fullName`과 찾은 계좌목록을 필드로 가지고 **==계좌목록은 엔티티이므로 한번더 DTO로 변환==**한다.

    ```java
    @Data
    public static class AccountListRespDto{
        private String fullName;
        private List<AccountDto> accounts = new ArrayList<>();

        public AccountListRespDto(User user, List<Account> accounts) {
            this.fullName = user.getFullName();
            this.accounts = accounts.stream()
                    .map(AccountDto::new)
                    .collect(Collectors.toList());
        }

        @Data
        private static class AccountDto {
            private Long id;
            private Long number;
            private Long balance;

            public AccountDto(Account account) {
                this.id=account.getId();
                this.number = account.getNumber();
                this.balance = account.getBalance();
            }
        }
    }
    ```

- 우선 본인계좌의 `fullName`을 출력할려면 User가 필요하므로 `userId`를 받아 조회를 한다.
- 그다음 계좌목록을 `List<Account>`로 받고 DTO로 변환하여 반환하면 끝이다.

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;

        public AccountListRespDto 계좌목록보기_유저별(Long userId){
            User userPS = userRepository.findById(userId).orElseThrow(
                    () -> new CustomApiException("유저를 찾을 수 없습니다.")
            );

            List<Account> accountListPS = accountRepository.findByUser_id(userId);

            return new AccountListRespDto(userPS,accountListPS);
        }
    }
    ```


## 계좌 삭제

---

- 계좌삭제 로직은 **==계좌번호를 확인하여 유효한 계좌번호==** 인지 확인 후 **==계좌번호 소유자==****==를 확인==**하고 확인이 되면 **==계좌를 삭제==**한다.
- 우선 계좌번호 소유자를 확인하는 메서드는 `Account` 도메인 클래스에 추가해준다.

    ```java
    @NoArgsConstructor
    @Getter
    @EntityListeners(AuditingEntityListener.class)
    @Table(name = "account_tb")
    @Entity
    public class Account {

    		// 생략 ...

        public void checkOwner(Long userId){
            if(!this.user.getId().equals(userId)){
                throw new CustomApiException("계좌 소유자가 아닙니다.");
            }
        }
    }
    ```

- 계좌번호를 확인하고 return 되는 Account 엔티티를 이용하여 계좌소유자를 확인하고 삭제해주면 끝이다.

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;

        @Transactional
        public void 계좌삭제(Long accountNumber, Long userId){
            // 1. 계좌확인
            Account accountPS = accountRepository.findByNumber(accountNumber).orElseThrow(
                    () -> new CustomApiException("계좌를 찾을 수 없습니다.")
            );
            // 2. 계좌 소유자 확인
            accountPS.checkOwner(userId);
            // 3. 계좌 삭제(응답할 데이터가 필요없다.)
            accountRepository.deleteById(accountPS.getId());
        }
    }
    ```


## 계좌 입금

---

- 계좌 입금을 할 때에는 순서가 있다.
    1. 입금 금액이 0원인지 체크
    2. 입금 계좌가 유효한 계좌인지 체크
    3. 입금
    4. 거래내역 남기기
- 거래 내역을 남길 때에는
    - `입금계좌`, `출금계좌`(ATM으로 입금하기 때문에 출금계좌는 null), `입금 후 잔액`(테스트시 필요), `입금 금액`, `입급인지 출금인지 구분`, `보내는 곳`, `받는곳`, `보내는사람의 전화번호` 가 필요
- 입금 로직은 새로운 금액이 save() 되는게 아닌 기존의 잔액(balance)가 추가되기 때문에 update() 가 일어난다. JPA의 더티체킹을 이용하면 된다.

    ```java

    @NoArgsConstructor
    @Getter
    @EntityListeners(AuditingEntityListener.class)
    @Table(name = "account_tb")
    @Entity
    public class Account {

    		// 생략...

        public void deposit(Long amount) {
            balance += amount;
        }
    }
    ```

- `depositAccountPS` 는 **==영속성 컨텍스트에 저장됐기 때문에 영속화된 객체==**이다. 그렇기 떄문에 ==**값을 수정하면 JPA의 더티체킹이 일어나서 Transaction 종료 후 DB에 update 쿼리가 나가게 된다.**==

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;
        private final TransactionRepository transactionRepository;

        @Transactional
        public AccountDepositRespDto 계좌입금(AccountDepositReqDto accountDepositReqDto){
            // 0원인지 체크
            if(accountDepositReqDto.getAmount() <= 0L){
                throw new CustomApiException("0원 이하의 금액을 입금할 수 없습니다.");
            }

            // 입금계좌 확인
            Account depositAccountPS = accountRepository.findByNumber(accountDepositReqDto.getNumber()).orElseThrow(
                    () -> new CustomApiException("계좌를 찾을 수 없습니다.")
            );

            // 입금
            depositAccountPS.deposit(accountDepositReqDto.getAmount());

            // 거래내역 남기기
            Transaction transaction = Transaction.builder()
                    .depositAccount(depositAccountPS)
                    .withdrawAccount(null)
                    .depositAccountBalance(depositAccountPS.getBalance())
                    .withdrawAccount(null)
                    .amount(accountDepositReqDto.getAmount())
                    .gubun(TransactionEnum.DEPOSIT)
                    .sender("ATM")
                    .receiver(accountDepositReqDto.getNumber() + "")
                    .tel(accountDepositReqDto.getTel())
                    .build();
            Transaction transactionPS = transactionRepository.save(transaction);

            return new AccountDepositRespDto(depositAccountPS, transactionPS);
        }
    }
    ```

    - DTO는 받을 때는 유효성 검사를 해서 받으면 된다.
    - DTO를 응답할 때는 계좌 ID, 계좌번호, 거래내역이 출력된다.

        ```java
        @Data
        public static class AccountDepositReqDto{
            @NotNull
            @Digits(integer = 4,fraction = 4)
            private Long number;
            @NotNull
            private Long amount;
            @NotEmpty
            @Pattern(regexp = "^(DEPOSIT)$")
            private String gubun;
            @NotEmpty
            @Pattern(regexp = "^[0-9]{3}[0-9]{4}[0-9]{4}")
            private String tel;
        }

        @Data
        public static class AccountDepositRespDto{
            private Long id; // 계좌 ID
            private Long number; // 계좌번호
            private TransactionDto transaction;

            public AccountDepositRespDto(Account account, Transaction transaction) {
                this.id = account.getId();
                this.number = account.getNumber();
                this.transaction = new TransactionDto(transaction);
            }

            @Data
            public static class TransactionDto{
                private Long id;
                private String gubun;
                private String sender;
                private String receiver;
                private Long amount;
                @JsonIgnore
                private Long depositAccountBalance; // 테스트 용도 컬럼
                private String tel;
                private String createdAt;

                public TransactionDto(Transaction transaction) {
                    this.id = transaction.getId();
                    this.gubun = transaction.getGubun().getValue();
                    this.sender = transaction.getSender();
                    this.receiver = transaction.getReceiver();
                    this.amount = transaction.getAmount();
                    this.depositAccountBalance = transaction.getDepositAccountBalance();
                    this.tel = transaction.getTel();
                    this.createdAt = CustomDateUtil.toStringFormat(transaction.getCreatedAt());
                }
            }
        }
        ```


## 계좌 출금

---

- 계좌출금 로직은 아래와 같다.

    1. 출금액이 0원인지 체크
    2. 출금계좌가 유효한지 체크
    3. 출금 소유자 확인
    4. 출금계좌 비밀번호 확인
    5. 출금계좌 잔액 확인
    6. 출금 후 거래내역 저장

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;
        private final TransactionRepository transactionRepository;

        @Transactional
    		public AccountWithdrawRespDto 계좌출금(AccountWithdrawReqDto accountWithdrawReqDto, Long userId){
    		    // 0원인지 체크
    		    if(accountWithdrawReqDto.getAmount() <= 0L){
    		        throw new CustomApiException("0원 이하의 금액을 입금할 수 없습니다.");
    		    }

    		    // 출금계좌 확인
    		    Account withdrawAccountPS = accountRepository.findByNumber(accountWithdrawReqDto.getNumber()).orElseThrow(
    		            () -> new CustomApiException("계좌를 찾을 수 없습니다.")
    		    );

    		    // 출금 소유자 확인
    		    withdrawAccountPS.checkOwner(userId);

    		    // 출금계좌 비밀번호 확인
    		    withdrawAccountPS.checkSamePassword(accountWithdrawReqDto.getPassword());

    		    // 출금계좌 잔액 확인
    		    withdrawAccountPS.checkBalance(accountWithdrawReqDto.getAmount());

    		    // 출금하기
    		    withdrawAccountPS.withdraw(accountWithdrawReqDto.getAmount());

    		    // 거래내역 남기기(내 계좌 -> ATM 으로 출금
    		    Transaction transaction = Transaction.builder()
    		            .withdrawAccount(withdrawAccountPS)
    		            .depositAccount(null)
    		            .withdrawAccountBalance(withdrawAccountPS.getBalance())
    		            .depositAccountBalance(null)
    		            .amount(accountWithdrawReqDto.getAmount())
    		            .gubun(TransactionEnum.WITHDRAW)
    		            .sender(withdrawAccountPS.getNumber()+"")
    		            .receiver("ATM")
    		            .tel(null)
    		            .build();

    		    Transaction transactionPS = transactionRepository.save(transaction);

    		    return new AccountWithdrawRespDto(withdrawAccountPS,transactionPS);
    		}
    }
    ```

- DTO 는 입금하기() 에서 사용한 DTO를 그대로 복붙하여 잔액을 표시하는 필드만 추가해준다.(재사용을 하지 않은 이유는 API 스펙이 바뀔 경우 수정이 필요해지기 때문)

    ```java
    @Data
    public static class AccountWithdrawReqDto{
        @NotNull
        @Digits(integer = 4,fraction = 4)
        private Long number;
        @NotNull
        @Digits(integer = 4,fraction = 4)
        private Long password;
        @NotNull
        private Long amount;
        @NotEmpty
        @Pattern(regexp = "^(WITHDRAW)$")
        private String gubun;
    }

    @Data
    public static class AccountWithdrawRespDto{
        private Long id; // 계좌 ID
        private Long number; // 계좌번호
        private Long balance; // 잔액
        private TransactionDto transaction;

        public AccountWithdrawRespDto(Account account, Transaction transaction) {
            this.id = account.getId();
            this.number = account.getNumber();
            this.balance = account.getBalance();
            this.transaction = new TransactionDto(transaction);
        }

        @Data
        public static class TransactionDto{
            private Long id;
            private String gubun;
            private String sender;
            private String receiver;
            private Long amount;
            private String createdAt;

            public TransactionDto(Transaction transaction) {
                this.id = transaction.getId();
                this.gubun = transaction.getGubun().getValue();
                this.sender = transaction.getSender();
                this.receiver = transaction.getReceiver();
                this.amount = transaction.getAmount();
                this.createdAt = CustomDateUtil.toStringFormat(transaction.getCreatedAt());
            }
        }
    }
    ```


## 계좌 이체

---

- 계좌이체는 아래와 같은 로직이 필요하다
    1. 출금계좌와 입금계좌가 동일한지 체크
    2. 이체금액이 0원인지 체크
    3. 출금계좌가 유효한지 체크
    4. 입금계좌가 유효한지 체크
    5. 출금 소유자 확인
    6. 출금계좌 비밀번호 확인
    7. 이체하기(출금금액과 출금계좌 잔액을 체크 → 출금)
    8. 이체하기(계좌이체금액을 입금계좌에 추가)
    9. 거래내역 남기기
- 우선 DTO 는 아래와 같다.

    ```java
    @Data
    public static class AccountTransferReqDto {
        @NotNull
        @Digits(integer = 4, fraction = 4)
        private Long withdrawNumber;
        @NotNull
        @Digits(integer = 4, fraction = 4)
        private Long depositNumber;
        @NotNull
        @Digits(integer = 4, fraction = 4)
        private Long withdrawPassword;
        @NotNull
        private Long amount;
        @NotEmpty
        @Pattern(regexp = "^(TRANSFER)$")
        private String gubun;
    }

    @Data
    public static class AccountTransferRespDto{
        private Long id; // 계좌 ID
        private Long number; // 계좌번호
        private Long balance; // 잔액
        private TransactionDto transaction;

        public AccountTransferRespDto(Account account, Transaction transaction) {
            this.id = account.getId();
            this.number = account.getNumber();
            this.balance = account.getBalance(); // 출금계좌 잔액
            this.transaction = new TransactionDto(transaction);
        }

        @Data
        public static class TransactionDto{
            private Long id;
            private String gubun;
            private String sender;
            private String receiver;
            private Long amount;
            @JsonIgnore
            private Long depositAccountBalance;
            private String createdAt;

            public TransactionDto(Transaction transaction) {
                this.id = transaction.getId();
                this.gubun = transaction.getGubun().getValue();
                this.sender = transaction.getSender();
                this.receiver = transaction.getReceiver();
                this.amount = transaction.getAmount();
                this.createdAt = CustomDateUtil.toStringFormat(transaction.getCreatedAt());
            }
        }
    }
    ```

- 계좌출금과 비슷한 코드이다.

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class AccountService {
        private final AccountRepository accountRepository;
        private final UserRepository userRepository;
        private final TransactionRepository transactionRepository;

        @Transactional
        public AccountTransferRespDto 계좌이체(AccountTransferReqDto accountTransferReqDto, Long userId){
            // 출금계좌와 입금계좌가 동일하면 안됨
            if(accountTransferReqDto.getWithdrawNumber().equals(accountTransferReqDto.getDepositNumber())){
                throw new CustomApiException("입금계좌와 출금계좌가 동일합니다.");
            }

            // 0원인지 체크
            if(accountTransferReqDto.getAmount() <= 0L){
                throw new CustomApiException("0원 이하의 금액을 입금할 수 없습니다.");
            }

            // 출금계좌 확인
            Account withdrawAccountPS = accountRepository.findByNumber(accountTransferReqDto.getWithdrawNumber()).orElseThrow(
                    () -> new CustomApiException("출급계좌를 찾을 수 없습니다.")
            );

            // 입금계좌 확인
            Account depositAccountPS = accountRepository.findByNumber(accountTransferReqDto.getDepositNumber()).orElseThrow(
                    () -> new CustomApiException("입금계좌를 찾을 수 없습니다.")
            );

            // 출금 소유자 확인
            withdrawAccountPS.checkOwner(userId);

            // 출금계좌 비밀번호 확인
            withdrawAccountPS.checkSamePassword(accountTransferReqDto.getWithdrawPassword());

            // 이체하기(출금금액과 잔액을 확인 -> 출금 -> 이체)
            withdrawAccountPS.withdraw(accountTransferReqDto.getAmount());
            depositAccountPS.deposit(accountTransferReqDto.getAmount());

            // 거래내역 남기기(내 계좌 -> ATM 으로 출금
            Transaction transaction = Transaction.builder()
                    .withdrawAccount(withdrawAccountPS)
                    .depositAccount(depositAccountPS)
                    .withdrawAccountBalance(withdrawAccountPS.getBalance())
                    .depositAccountBalance(depositAccountPS.getBalance())
                    .amount(accountTransferReqDto.getAmount())
                    .gubun(TransactionEnum.TRANSFER)
                    .sender(accountTransferReqDto.getWithdrawNumber()+"")
                    .receiver(accountTransferReqDto.getDepositNumber()+"")
                    .build();

            Transaction transactionPS = transactionRepository.save(transaction);

            return new AccountTransferRespDto(withdrawAccountPS,transactionPS);
        }
    }
    ```


## 계좌 상세보기 서비스 구현

---

- 계좌 정보를 조회하는 쿼리 1개 , 거래내역을 조회하는 쿼리 1개 를 각각 실행하여 DTO 로 데이터를 통합시키면 된다.

    ```java
    public AccountDetailRespDto 계좌상세보기(Long number, Long userId, Integer page) {
        // 1. 구분값, 페이지고정
        String gubun = "ALL";

        Account accountPS = accountRepository.findByNumber(number).orElseThrow(
                () -> new CustomApiException("계좌를 찾을 수 없습니다.")
        );

        accountPS.checkOwner(userId);

        // 입출금 목록보기
        List<Transaction> transactionListPS = transactionRepository.findTransactionList(accountPS.getId(), gubun, page);

        return new AccountDetailRespDto(accountPS, transactionListPS);
    }
    ```

## 관련 문서

- 상위 목차: [[어플리케이션 구현]]
- 이전 문서: [[AccountController 테스트]]
- 다음 문서: [[AccountService 테스트]]
