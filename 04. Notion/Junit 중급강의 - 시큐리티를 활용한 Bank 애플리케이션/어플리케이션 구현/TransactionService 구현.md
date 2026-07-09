## 입출금내역 조회하기

---

- 계좌가 유효한지 체크한다.
- 입출금을 할려면 로그인한 상태이기 때문에 로그인한 사용자가 계좌주인인지 체크한다.(로직 자체가 자기 계좌의 입출금내역을 조회하는 로직이기 때문)
    
    ```java
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    @Service
    public class TransactionService {
        private final TransactionRepository transactionRepository;
        private final AccountRepository accountRepository;
    
        public TransactionListRespDto 입출금목록보기(Long userId, Long accountNumber, String gubun, int page) {
            Account accountPS = accountRepository.findByNumber(accountNumber).orElseThrow(
                    () -> new CustomApiException("계좌가 유효하지 않습니다!")
            ); // 유효한 계좌인치 확인
    
            accountPS.checkOwner(userId); // 올바른 사용자가 요청했는지 확인
    
            List<Transaction> transactionListPS =
                    transactionRepository.findTransactionList(accountPS.getId(), gubun, page);
    
            return new TransactionListRespDto(accountPS, transactionListPS);
        }
    }
    ```
    
- DTO는 아래와 같다.
    
    ```java
    public class TransactionRespDto {
    
        @Data
        public static class TransactionListRespDto {
            private List<TransactionDto> transactions = new ArrayList<>();
    
            public TransactionListRespDto(Account account, List<Transaction> transactions) {
                this.transactions =
                        transactions.stream()
                                .map((t) -> new TransactionDto(t, account.getNumber()))
                                .collect(Collectors.toList());
            }
    
            @Data
            public static class TransactionDto {
                private Long id;
                private String gubun;
                private Long amount;
                private String sender;
                private String receiver;
                private String tel;
                private String createdAt;
                private Long balance;
    
                public TransactionDto(Transaction transaction, Long accountNumber) {
                    this.id = transaction.getId();
                    this.gubun = transaction.getGubun().getValue();
                    this.amount = transaction.getAmount();
                    this.sender = transaction.getSender();
                    this.receiver = transaction.getReceiver();
                    this.tel = transaction.getTel() == null ? "없음" : transaction.getTel();
                    this.createdAt = CustomDateUtil.toStringFormat(transaction.getCreatedAt());
    
                    if (transaction.getDepositAccount() == null) {
                        this.balance = transaction.getWithdrawAccountBalance();
                    } else if (transaction.getWithdrawAccount() == null) {
                        this.balance = transaction.getDepositAccountBalance();
                    } else {
                        if (accountNumber.longValue() == transaction.getDepositAccount().getNumber()) {
                            this.balance = transaction.getDepositAccountBalance();
                        } else {
                            this.balance = transaction.getWithdrawAccountBalance();
                        }
                    }
                }
            }
        }
    }
    ```