  

테스트 코드 작성시 @Transacional 을 클래스 단에 걸어주면 테스트 종료 시 rollback 을 하여 테이블에 데이터가 들어가있지 않다 !! 하지만 PK 값은 초기화 되지 않기 때문에 id를 조회하는 테스트가 있다면 전체 테스트 실행 시 오류가 난다 !!

  

## SQL 파일 작성

- `resources/db/teardown.sql` 파일을 작성하여 table 을 지우는게 아닌 비우는 truncate 로 꺠끗하게 초기상태로 만들어주자.
    
    ```sql
    SET REFERENTIAL_INTEGRITY FALSE; -- 제약조건 무효화
    truncate table transaction_tb;
    truncate table account_tb;
    truncate table user_tb;
    SET REFERENTIAL_INTEGRITY TRUE; -- 제약조건 재설정
    ```
    
- 클래스 상단에는 `@Sql(”classpath:/db/teardown.sql)` 적어주면 적용 끝