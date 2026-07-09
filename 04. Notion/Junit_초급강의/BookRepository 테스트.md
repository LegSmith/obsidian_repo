  

테스트!!  
→ 스프링부트를 띄우고 실행(Controller + Service + Repository = 통합 테스트)  
→ ==_**테스트 환경에서 테스트(Repository 만 띄운다 = 단위 테스트)**_==

  

## Save 테스트

---

- 순수하게 Repository 만 띄어놓은 환경에서 테스트할려면 `@DataJpaTest` 어노테이션을 사용한다.
- Repository 까지 데이터가 넘어올 때는 DTO가 아닌 엔티티 이므로 엔티티를 바로 생성하여 데이터를 받아 바로 저장해서 엔티티를 비교하면 된다 !!
    
    ```java
    @DataJpaTest // DB 와 관련된 컴포넌트만 메모리에 로딩(단위 테스트)
    public class BookRepositoryTest {
    
        @Autowired
        private BookRepository bookRepository;
    
        // 1. 책 등록
        @Test
        public void save_test() throws Exception {
            // given
            String title = "Football";
            String author = "Cristiano";
    
            Book book = Book.builder().title(title).author(author).build();
            // when
            Book bookPS = bookRepository.save(book);
            // then
            assertThat(bookPS.getTitle()).isEqualTo(title);
            assertThat(bookPS.getAuthor()).isEqualTo(author);
        }
    }
    ```
    

## SELECT 테스트

---

==**Junit 지원 어노테이션**==

`@BeforeAll` : 테스트 시작 전에 ==**한번만**== 실행

`@BeforeEach` : 각 테스트 시작전에 ==**한번씩**== 실행

- 테스트마다 데이터를 만들고 `persist()` 하는게 반복되므로 `@BeforeEach` 로 ==**테스트 시작전에 데이터를 초기화**==한다.

```java
@DataJpaTest // DB 와 관련된 컴포넌트만 메모리에 로딩(단위 테스트)
public class BookRepositoryTest {

    @Autowired
    private BookRepository bookRepository;

    @BeforeEach
    public void dataInit() {
        String title = "This is Football";
        String author = "Cristiano Ronaldo";
        Book book = Book.builder().title(title).author(author).build();
        bookRepository.save(book);
    }

    // 2. 책 목록보기
    @Test
    public void 책목록보기_test() throws Exception {
        // given
        String title = "This is Football";
        String author = "Cristiano Ronaldo";
        // when
        List<Book> books = bookRepository.findAll();

        // then
        assertThat(books.size()).isEqualTo(1);
        assertThat(books.get(0).getAuthor()).isEqualTo(author);
        assertThat(books.get(0).getTitle()).isEqualTo(title);
    }

    // 3. 책 한건보기
    @Test
    public void 책한건보기_test() throws Exception {
        // given
        String title = "This is Football";
        String author = "Cristiano Ronaldo";

        // when
        Book findBook = bookRepository.findById(1L).get();

        // then
        assertThat(findBook.getAuthor()).isEqualTo(author);
        assertThat(findBook.getTitle()).isEqualTo(title);
    }
}
```

## DELETE 테스트

---

- findById() 로 반환되는 엔티티는 Java8 이 지원하는 Optional 로 감싸진다.
- isPresent() 메서드는 Optional로 감싸진 엔티티가 null 이면 false을 반환한다.
    
    ```java
    @DataJpaTest // DB 와 관련된 컴포넌트만 메모리에 로딩(단위 테스트)
    public class BookRepositoryTest {
    
        @Autowired
        private BookRepository bookRepository;
    
        @BeforeEach
        public void dataInit() {
            String title = "This is Football";
            String author = "Cristiano Ronaldo";
            Book book = Book.builder().title(title).author(author).build();
            bookRepository.save(book);
        }
    
        // 4. 책 삭제
        @Test
        public void 책삭제_test() throws Exception {
            // given
            Long id = 1L;
    
            // when
            bookRepository.deleteById(id);
    
            // then
            /*
             * List<Book> books = bookRepository.findAll();
             * assertThat(books.size()).isEqualTo(0);
             */
    
            Optional<Book> bookPS = bookRepository.findById(id);
    
            assertFalse(bookPS.isPresent());
    
        }
    
    }
    ```
    

  

> ==_**문제점은 책삭제_test() 메서드만 테스트를 하면 성공하지만 책저장,조회 테스트를 한꺼번에 테스트하게 되면 ID값이 없다는 에러가 뜨게 된다 !!**_==

## UPDATE 테스트

---

- 책 수정 테스트 메서드를 실행하기 전에 `@SQL` 로 ==_**PK 값 초기화하는 SQL을 실행해주게 되면 PK값이 1번인 데이터가 들어온 상태에서 테스트**_==를 할 수 있다. → `@BeforeEach` 메서드가 1건의 데이터를 저장해주기 때문
- PK값 1번으로 객체를 생성하면 JPA가 영속화를 시켜준다.
- 생성자에서 값을 받으므로 더티체킹으로 인해 데이터가 변경된다.
    
    ```java
    @Test
    @Sql("classpath:db/tableInit.sql")
    public void 책수정_test() throws Exception {
        // given
        Long id = 1L;
        String title = "Football King";
        String author = "Ronaldo";
        Book book = new Book(id, title, author);
    
        // when
        Book bookPS = bookRepository.save(book);
    
        // then
        assertThat(id).isEqualTo(bookPS.getId());
        assertThat(title).isEqualTo(bookPS.getTitle());
        assertThat(author).isEqualTo(bookPS.getAuthor());
    
    }
    ```
    

## Junit 테스트 특징

---

1. 테스트 메서드가 3개라면 ==_**순서 보장이 안된다**_==. → 해결 법은 `Order()` 어노테이션으로 순서를 정해준다.
2. 테스트 메서드 1개가 종료되면 데이터가 초기화 된다 → `Transactional()` 어노테이션에 의해서
    - ==_**문제는 PK값이 초기화가 안된다 !!**_==
    - 이러한 문제 때문에 여러 테스트 메서드가 실행 후 초기화가 될 때 PK 값이 초기화되지 않고 계속 증가해서 오류가 생긴다

## 여러 테스트 메서드 통합 테스트 시 PK값 에러 해결법

---

### JPQL 작성

- JPQL 문법으로 PK값을 초기화하는 쿼리가 담긴 메서드 작성
- `@AfterEach` 어노테이션을 이용하여 위 메서드에 적용시켜 테스트 메서드가 종료될 때마다 PK가 초기화 되게함

### @Sql 어노테이션

- pk 를 초기하는 쿼리를 작성한 파일을 resources 경로안에 지정해두고 `@Sql(경로)` 를 지정해서 테스트 메서드가 실행되기 전에 SQL 이 동작하게 한다.
- `resources/db` 경로안에 `tableInit.sql` 파일을 작성해준다.
    
    ```java
    drop table if exists Book;
    create table Book (
        id bigint generated by default as identity,
        author varchar(20) not null,
        title varchar(50) not null,
        primary key (id)
    );
    ```
    
- PK 값을 다루는 테스트 메서드에 `@Sql("classpath:db/tableInit.sql")` 를 작성해주면 된다.
    
    ```java
    @DataJpaTest // DB 와 관련된 컴포넌트만 메모리에 로딩(단위 테스트)
    public class BookRepositoryTest {
    
        @Autowired
        private BookRepository bookRepository;
    
        @BeforeEach
        public void dataInit() {
            String title = "This is Football";
            String author = "Cristiano Ronaldo";
            Book book = Book.builder().title(title).author(author).build();
            bookRepository.save(book);
        }
    
        // 3. 책 한건보기
        @Sql("classpath:db/tableInit.sql")
        @Test
        public void 책한건보기_test() throws Exception {
            // given
            String title = "This is Football";
            String author = "Cristiano Ronaldo";
    
            // when
            Book findBook = bookRepository.findById(1L).get();
    
            // then
            assertThat(findBook.getAuthor()).isEqualTo(author);
            assertThat(findBook.getTitle()).isEqualTo(title);
        }
    
        // 4. 책 삭제
        @Sql("classpath:db/tableInit.sql")
        @Test
        public void 책삭제_test() throws Exception {
            // given
            Long id = 1L;
    
            // when
            bookRepository.deleteById(id);
    
            // then
            /*
             * List<Book> books = bookRepository.findAll();
             * assertThat(books.size()).isEqualTo(0);
             */
    
            Optional<Book> bookPS = bookRepository.findById(id);
    
            assertFalse(bookPS.isPresent());
        }
    
        // 5. 책 수정
    }
    ```
    

## @Transactional 이란?

---

- Junit 에서 메서드가 실행할때 트렌젝션이 시작되고 메서드가 종료되면서 트랜잭션이 같이 종료된다. → 이 때, ==**트랜잭션이 종료되면 Rollback**== 이 된다(Default 값)
- data가 rollback 이 되도 DB의 PK값은 초기화되지 않는다, 만약 ID값을 검증하는 테스트라면 각 테스트 메서드마다 DB를 DROP하는 SQL을 실행시켜 PK값을 초기화 해야 한다.