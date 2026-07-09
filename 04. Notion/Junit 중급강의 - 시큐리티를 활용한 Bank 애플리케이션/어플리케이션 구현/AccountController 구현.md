## 계좌등록 컨트롤러

---

- DTO를 받을 때는 `@Valid` 로 유효성 검사를한다.
    - _**유효성 검사는 AOP로 구현했기 때문에 유효성검사 실패시 동작하는 로직은 컨트롤러에 없다.**_
- `userId`는 현재 로그인되어 있는 시큐리티 세션에서 가져오는 `@AuthenticationPricipal` 어노테이션을 이용한다.

    ```java
    @RestController
    @RequestMapping("/api")
    @RequiredArgsConstructor
    public class AccountController {
        private final AccountService accountService;

        @PostMapping("/s/account")
        public ResponseEntity<?> saveAccount(@RequestBody @Valid AccountSaveReqDto dto,
                                             BindingResult bindingResult,
                                             @AuthenticationPrincipal LoginUser loginUser){
            AccountSaveRespDto accountSaveRespDto = accountService.계좌등록(dto, loginUser.getUser().getId());

            return new ResponseEntity<>(new ResponseDto<>(1,"계좌등록 성공",accountSaveRespDto), HttpStatus.CREATED);

        }
    }
    ```

    ![[Untitled 60.png|Untitled 60.png]]


## 계좌 목록보기 컨트롤러

---

- REST API 형식으로는 `/s/account/{id}` 로 하는게 맞다. 하지만 그렇게 되면 **==로그인한 유저가 타 유저의 계좌목록을 요청할 수 있기 때문에 권한검증 로직이 필요==**해진다.
- 순수하게 **==컨트롤러에서는 계좌목록만 보여주는 로직만 작성하기 위해 로그인한 유저가 본인계좌만 조회할 수 있게 설계==**하였다.

    ```java
    @RestController
    @RequestMapping("/api")
    @RequiredArgsConstructor
    public class AccountController {
        private final AccountService accountService;

        @GetMapping("/s/account/login-user")
        public ResponseEntity<?> findUserAccount(@AuthenticationPrincipal LoginUser loginUser){
            AccountListRespDto accountListRespDto = accountService.계좌목록보기_유저별(loginUser.getUser().getId());

            return new ResponseEntity<>(new ResponseDto<>(1,"유저별 계좌목록보기 성공",accountListRespDto),HttpStatus.OK);
        }
    }
    ```


## 계좌 삭제하기 컨트롤러

---

- `@DeleteMapping` 은 HTTP Body 데이터가 없으므로 `null` 로 전송해주면 된다.

    ```java
    @RestController
    @RequestMapping("/api")
    @RequiredArgsConstructor
    public class AccountController {
        private final AccountService accountService;

    		@DeleteMapping("/s/account/{number}")
        public ResponseEntity<?> deleteAccount(@PathVariable Long number, @AuthenticationPrincipal LoginUser loginUser){
            accountService.계좌삭제(number,loginUser.getUser().getId());
            return new ResponseEntity<>(new ResponseDto<>(1,"계좌삭제 성공",null),HttpStatus.OK);
        }
    }
    ```


## 계좌 입금 컨트롤러

---

- Junit 이 아닌 Postman 으로 테스트 해보기 위해서 DummyDevInit 클래스에 user 2건, account 2건을 서버 실행 시 생성하도록 설정함.

    ```java
    @Configuration
    public class DummyDevInit extends DummyObject{

        @Profile("dev") // dev 모드에서만 실행
        @Bean
        CommandLineRunner init(UserRepository userRepository, AccountRepository accountRepository){
            return (args) -> {
                // 서버 실행시  무조건 실행.
                User cristiano = userRepository.save(newUser("cristiano", "ronaldo"));
                User messi = userRepository.save(newUser("messi", "lionel"));

                accountRepository.save(newAccount(1111L,cristiano));
                accountRepository.save(newAccount(2222L,messi));
            };
        }
    }
    ```

- 단순히 `@Valid` 로 DTO를 받을 때 유효성 검사를 하고 JSON 통신을 위해 `@RequestBody` 를 붙혀주었다.

    ```java

    @RestController
    @RequestMapping("/api")
    @RequiredArgsConstructor
    public class AccountController {
        private final AccountService accountService;

    		@PostMapping("/account/deposit") // 입금이기 때문에 인증이 필요없다.
        public ResponseEntity<?> depositAccount(@RequestBody @Valid AccountDepositReqDto accountDepositReqDto,
                                                BindingResult bindingResult){
            AccountDepositRespDto accountDepositRespDto = accountService.계좌입금(accountDepositReqDto);

            return new ResponseEntity<>(new ResponseDto<>(1,"계좌 입금 완료",accountDepositRespDto),HttpStatus.CREATED);

        }
    }
    ```


## 계좌 출금 컨트롤러

---

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class AccountController {
    private final AccountService accountService;

		@PostMapping("/s/account/withdraw") // 입금이기 때문에 인증이 필요없다.
    public ResponseEntity<?> withdrawAccount(@RequestBody @Valid AccountWithdrawReqDto accountWithdrawReqDto,
                                             BindingResult bindingResult,
                                             @AuthenticationPrincipal LoginUser loginUser) {
        AccountWithdrawRespDto accountWithdrawRespDto =
                accountService.계좌출금(accountWithdrawReqDto, loginUser.getUser().getId());
        return new ResponseEntity<>(new ResponseDto<>(1, "계좌 출금 완료", accountWithdrawRespDto), HttpStatus.CREATED);
    }
}
```

## 계좌 이체 컨트롤러

---

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class AccountController {
    private final AccountService accountService;

		@PostMapping("/s/account/transfer") // 입금이기 때문에 인증이 필요없다.
    public ResponseEntity<?> transferAccount(@RequestBody @Valid AccountTransferReqDto accountTransferReqDto,
                                             BindingResult bindingResult,
                                             @AuthenticationPrincipal LoginUser loginUser) {
        AccountTransferRespDto accountTransferRespDto =
                accountService.계좌이체(accountTransferReqDto, loginUser.getUser().getId());
        return new ResponseEntity<>(new ResponseDto<>(1, "계좌 이체 완료", accountTransferRespDto), HttpStatus.CREATED);
    }
}
```

## 계좌 상세 보기 컨트롤러

---

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class AccountController {
    private final AccountService accountService;

		@GetMapping("/s/account/{accountNumber}")
    public ResponseEntity<?> findDetailAccount(@PathVariable Long accountNumber,
                                               @RequestParam(value = "page", defaultValue = "0") Integer page,
                                               @AuthenticationPrincipal LoginUser loginUser){
        AccountDetailRespDto accountDetailRespDto =
                accountService.계좌상세보기(accountNumber, loginUser.getUser().getId(), page);

        return ResponseEntity.ok(new ResponseDto<>(1,"계좌상세보기 성공",accountDetailRespDto));
    }
}
```

## 관련 문서

- 상위 목차: [[어플리케이션 구현]]
- 다음 문서: [[AccountController 테스트]]
