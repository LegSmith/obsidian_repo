  

# JDBC 이해

---

JDBC가 등장한 이유는 ==_**애플리케이션을 개발하면서 중요한 데이터를 대부분 데이터베이스의 보관하기 때문에**_== JDBC가 나왔다.

### 애플리케이션 서버와 DB - 일반적인 사용법

1. **커넥션 연결** : 주로 _==**TCP/IP 를 사용하여 커넥션을 연결**==_한다.
2. **SQL 전달** : 서버는 _==**DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달**==_
3. **결과 응답** : DB는 전달된 ==_**SQL을 수행하고 그 결과를 응답**_==한다.
    
    ![[Untitled 61.png|Untitled 61.png]]
    
    > 문제는 ==_**데이터베이스의 종류마다 커넥션을 연결하는 방법, SQL을 전달하는 방법등이 모두 다르기 때문에**_== 표준이 없었다 !!  
    > 이런 문제를 해결하기 위해 `JDBC` 라는 **자바 표준**이 등장한다!
    

### JDBC 표준 인터페이스

`JDBC(Java Database Connectivity)`는 ==_**자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API**_== 다.

- JDBC 는 다음 3가지 기능을 표준 인터페이스로 정의해서 제공한다.
    
    - `java.sql.Connection` : 연결
    - `java.sql.Statement` : SQL을 담은 내용
    - `java.sql.ResultSet` : SQL 요청 응답
    
    ![[Untitled 1 45.png|Untitled 1 45.png]]
    
- 즉, ==_**개발자는 이 표준 인터페이스만 사용해서 개발**_==을 하고, 각각의 ==**DB 벤더(회사)가 JDBC 인터페이스를 자신의 DB에 맞도록 구현해서 라이브러리로 제공**==하는데 이것은 **JDBC 드라이버**라 한다.

> ==**표준화의 한계**==  
> **JDBC의 등장으로 많은 것이 편리해졌지만, DB마다 SQL, 데이터타입등 일부 사용법은 다르다. ANSI SQL 이라는 표준이 있지만 한계가 있다. 참고로 JPA(Java Persistence API)를 사용하면 JPQL을 통해 SQL이 통일되기 때문에 해결이 가능하다.**

# JDBC와 최신 데이터 접근 기술

---

JDBC는 1997년에 출시된 만큼 오래된 기술이고, 사용하는 방법도 복잡하다. 최근에는 JDBC를 직접 사용하지않고 편리하게 사용하는 기술을 쓴다. 대표적으로 `SQL Mapper` 와 `ORM` 기술로 나눌 수 있다.

### SQL Mapper

- **장점** : JDBC를 편리하게 사용하도록 도와준다.
    - **SQL 응답 결과를 객체로 변환**
    - **JDBC의 반복 코드를 제거**
- **단점** : _**개발자가 SQL을 직접 작성**_ ← ORM 기술에 비교하면 단점이다.
- 대표 기술 : `스프링 Jdbc Template` , `MyBatis`
    
    ![[Untitled 2 31.png|Untitled 2 31.png]]
    

### ORM 기술

- `ORM(Object Relational Mapping)` 은 ==_**객체를 관계형 DB 테이블과 매핑**_==해주는 기술이다.
- ORM 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고 ==**ORM 기술이 대신 SQL을 동적으로 만들어 실행**==해준다.
- 대표기술 : `JPA` , `Hibernate` , `Eclipes Link`
    
    ![[Untitled 3 21.png|Untitled 3 21.png]]
    

  

> _**MyBatis 든 JPA든 결국에 내부흐름을 보면 JDBC를 사용**_하게 된다. 즉, JDBC가 어떻게 동작하는지 기본 원리는 알아두어야 한다.==**JDBC는 자바 개발자라면 꼭 알아두어야 하는 필수 기본 기술**==이다.

# 데이터 베이스 연결

---

- DB접속을 위한 `URL` , `USERNAME` , `PASSWORD` 를 상수로 지정하자.
    
    ```java
    public abstract class ConnectionConst {
    		public static final String URL = "jdbc:h2:tcp://localhost/~/test";
        public static final String USERNAME = "sa";
        public static final String PASSWORD = "";
    }
    ```
    
- 그리고 실제 상수를 이용하여 구현한다.
- `Connection` 객체는 `java.sql` 패지키에 있는 객체를 사용해야 한다.
    
    - 실제 **JDBC 에 연결하는 커넥션**이다.
    
    ```java
    import static hello.jdbc.connection.ConnectionConst.*;
    
    @Slf4j
    public class DBConnectionUtil {
    
        public static Connection getConnection(){
            try {
                Connection connection = DriverManager.getConnection(URL,USERNAME,PASSWORD);
                log.info("get connection={}, class={}",connection, connection.getClass());
                return connection;
            } catch (SQLException e) {
                throw new IllegalStateException(e);
            }
        }
    }
    ```
    
- 테스트 코드를 작성하여 제대로 커넥션이 연결됐는지 확인해보자
    
    ```java
    @Slf4j
    public class DBConnectionUtilTest {
    
        @Test
        public void connection() throws Exception {
            //given
    
            //when
            Connection connection = DBConnectionUtil.getConnection();
    
            //then
            assertThat(connection).isNotNull();
        }
    }
    ```
    
    ![[Untitled 4 18.png|Untitled 4 18.png]]
    

### JDBC DriverManager 연결 이해

- JDBC가 제공하는 `DriverManager` 는 라이브러리에 등록된 DB ==**드라이버들을 관리하고, 커넥션을 획득하는 기능을 제공**==한다.
    
    ![[Untitled 5 15.png|Untitled 5 15.png]]
    
- DriverMnager 가 커넥션 요청을 하는 순서는 아래와 같다.
    1. 애플리케이션 로직에서 `DriverManger.getConnection();` 을 호출한다.
    2. `DriverManager` 는 **라이브러리에 등록된 드라이버 목록을 자동으로 인식**하고, ==**드라이버들에게 순서대로 다음 정보를 넘겨 커넥션을 획득할 수 있는지 확인**==한다.
        1. URL : 예시로 `jdbc:h2:tcp://localhost/~/test`
        2. `USERNAME` , `PASSWORD` 등 접속에 필요한 추가정보
        3. ==**정보를 받은 드라이버는 URL 정보를 체크하여 본인이 처리할 수 있는 요청인지 확인**==한다. 위에 URL을 보면 `jdbc:h2` 로 시작하기 때문에 **H2 드라이버가 확인하고 실제 DB에 연결하여 커넥션을 획득하고 클라이언트에 반환**한다.
    3. 이렇게 찾은 커넥션 구현체가 클라이언트에 반환된다.

  

> 즉, 위 애플리케이션은 H2 드라이버만 라이브러리에 등록했기 때문에 H2 드라이버가 제공하는 H2 커넥션을 제공받는다 !

# JDBC 개발 - 등록

---

**시나리오**

- JDBC 를 사용해서 회원(Member) 데이터를 데이터베이스에 관리하는 기능을 개발한다.

- 우선 회원테이블에 데이터를 저장하기 위해 `Member` 클래스를 만든다.
    
    ```java
    @Data
    public class Member {
    
        private String memberId;
        private int money;
    
        public Member() {
        }
    
        public Member(String memberId, int money) {
            this.memberId = memberId;
            this.money = money;
        }
    }
    ```
    
- 그리고 실제 Member 저장 같은 기능을 구현하는 Repository 를 만들었다.
    
    - `save()` 메서드는 파라미터로 Member 객체를 받아 Member 객체를 실제 DB 테이블에 저장하는 로직을 갖고 있다.
    - 순수한 JDBC 를 구현하기 위해 ==**데이터를 저장하는 INSERT 쿼리문을 String 에 담는다.**==
    - 그리고 `Connection` 객체와 `PreparedStatement`(SQL을 담는객체) 를 선언하고 **우선 null 로 초기화**한다.
    - 그리고 DB 커넥션을 가져오는 로직은 `getConnection()` 에 구현한 뒤 반환값인 커넥션을 위에서 선언한 Connection 객체에 담는다.
    - DB와 연결되어있는 커넥션객체에 `prepareStatement()` 호출하여 **SQL을 전달**하고, 위에서 선언한 ==**PreparedStatement 객체에 파라미터 바인딩**==을 한다.
    - 다음 실행하는 명령어인 `executeUpdate()` 를 호출한다.
    
    ```java
    @Slf4j
    public class MemberRepositoryV0 {
    
        public Member save(Member member) throws SQLException {
            String sql = "INSERT INTO member(member_id, money) values(?, ?)";
    
            Connection con = null;
            PreparedStatement pstmt = null; // 파라미터를 바인딩 할 수 있는 SQL 을 담을 수 있다.
    
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, member.getMemberId());
                pstmt.setInt(2, member.getMoney());
                pstmt.executeUpdate(); // 실제 실행시키는 명령어
                return member;
            } catch (SQLException e) {
                log.error("db error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
        }
    
        private void close(Connection con, Statement statement, ResultSet resultSet) { // Statement 는 일반 SQL 을 담는다.
            if (resultSet != null) {
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (con != null) {
                try {
                    con.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
        }
    
        private Connection getConnection() {
            return DBConnectionUtil.getConnection();
        }
    }
    ```
    

### 리소스 정리

- 쿼리를 실행하고 나면 _==**반드시 리소스를 정리해야 한다 !!**==_
- 리소스 정리는 `close()` 메서드를 사용하면 되는데 이 때 try-catch 로 묶어야 한다. **이중 try 문은 가독성이 안좋아 따로 메서드로 추출**했다.
- 이 때 본코드 finally 부분에 리소스정리 로직을 담은 이유는 ==**try 안에서 close() 를 호출하면 호출하기전 예외가 터졌을 때 close() 가 실행하지 않기 때문에**== 어떠한 경우에도 실행하는 finally 부분에 close()로직을 담았다.
- _==**리소스 정리 순서는 항상 역순**==_으로 한다.
    - `Connection` → `Statement(PreparedStatement)` → `ResultSet` 으로 생성했기 때문에 정리는 역순으로 하면 된다.

  

> `PreparedStatement` 는 `Statement`의 자식 타입인데, **SQL 작성시 ? 를 통한 파라미터 바인딩 기능을 제공**한다.  
> 그리고 ==**SQL Injection 공격을 예방**==할 수 있다.

  

### 테스트 코드

- 이후에 조회, 수정 등을 수행하기 위해 테스트 메서드 이름을 crud() 로 지정했다.
    
    ```java
    class MemberRepositoryV0Test {
    
        MemberRepositoryV0 repository = new MemberRepositoryV0();
    
        @Test
        void crud() throws SQLException {
            Member member = new Member("memberV0",10000);
            repository.save(member);
        }
    }
    ```
    

# JDBC 개발 - 조회

---

조회는 반환값이 존재하기 때문에 `ResultSet` 개념을 아는것이 중요하다 !!

- 데이터 등록 시에는 반환값으로 생성항 row 수가 Int 로 반환되었지만 **데이터 조회는 ResultSet 으로 반환**된다.
- 데이터 등록같은 **데이터의 변동이 일어나는 SQL 실행은 executeUpdate() 를 호출**하고 ==**데이터 조회는 executeQuery() 를 호출**==한다. 반환값으로 ResultSet 객체가 반환된다.
- ResultSet 객체는 `Cursor`가 ==**테이블 row 를 가리키면서 데이터에 접근**==한다.
    
    - `ResultSet.next()` 를 _==**맨 처음 실행하면 첫번쨰 row에 있는 데이터를 가리킨다.**==_
    - 필드명을 통해 해당 row에 있는 데이터를 가져올 수 있다.
    - Cursor 가 가리키는 row가 없으면 false 를 반환한다.
    - 참고로 ==**최초 Cursor 는 데이터를 가리키고 있지 않기 때문에**== `ResultSet.next()` 를 한 번 호출해야 한다.
    
    ```java
    @Slf4j
    public class MemberRepositoryV0 {
        public Member findById(String memberId) throws SQLException {
            String sql = "SELECT * FROM member WHERE member_id = ?";
    
            Connection con = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
    
                pstmt.setString(1, memberId);
                rs = pstmt.executeQuery(); // 데이터변경일 경우엔 executeUpdate() , 데이터를 조회할 때는 executeQuery()
    
                if (rs.next()) {
                    Member member = new Member();
                    member.setMemberId(rs.getString("member_id"));
                    member.setMoney(rs.getInt("money"));
                    return member;
                } else {
                    throw new NoSuchElementException("Member not found memberId = " + memberId);
                }
            } catch (SQLException e) {
                log.error("DB Error", e);
                throw e;
            }finally {
                close(con,pstmt,rs);
            }
        }
    
    
        private void close(Connection con, Statement statement, ResultSet resultSet) { // Statement 는 일반 SQL 을 담는다.
            if (resultSet != null) {
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (con != null) {
                try {
                    con.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
        }
    
        private Connection getConnection() {
            return DBConnectionUtil.getConnection();
        }
    }
    ```
    

# JDBC 개발 - 수정, 삭제

---

DB 저장,수정,삭제 등 과 같은 쿼리실행은 `executeUpdate()` 로 실행하면 된다. _==**반환값으로는 영향받은 row 수를 반환**==_한다.

- 데이터 수정과 삭제는 등록과 매우 유사하다 !!
- 이번에는 `executeUpdate()` 실행 후 영향받은 row수를 받아보자(해당 예제는 한건의 데이터만 수정하기 때문에 0 아니면 1이 반환한다)
    
    ```java
    @Slf4j
    public class MemberRepositoryV0 {
    		public void update(String memberId, int money) throws SQLException {
            String sql = "UPDATE member SET money=? WHERE member_id=?";
    
            Connection con = null;
            PreparedStatement pstmt = null;
    
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setInt(1, money);
                pstmt.setString(2, memberId);
                int resultSize = pstmt.executeUpdate(); // 쿼리를 실행하고 영향받은 row 수를 반환
                log.info("resultSize={}", resultSize);
            } catch (SQLException e) {
                log.error("DB Error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
        }
    
    		public void delete(String memberId) throws SQLException {
            String sql = "DELETE FROM member WHERE member_id=?";
    
            Connection con = null;
            PreparedStatement pstmt = null;
    
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, memberId);
                int resultSize = pstmt.executeUpdate(); // 쿼리를 실행하고 영향받은 row 수를 반환
                log.info("resultSize={}", resultSize);
            } catch (SQLException e) {
                log.error("DB Error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
    
        }
    
        private void close(Connection con, Statement statement, ResultSet resultSet) { // Statement 는 일반 SQL 을 담는다.
            if (resultSet != null) {
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
    
            if (con != null) {
                try {
                    con.close();
                } catch (SQLException e) {
                    log.error("error", e);
                }
            }
        }
    
        private Connection getConnection() {
            return DBConnectionUtil.getConnection();
        }
    }
    ```