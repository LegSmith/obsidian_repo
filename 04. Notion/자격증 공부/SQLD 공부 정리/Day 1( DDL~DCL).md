# DDL : Data Definition Language

---

**데이터를** ==**보관하고 관리**==**하기 위한 객체의 구조를 정의하기 위한 언어**

## CREATE : 테이블 구조 생성

```sql
CREATE TABLE C_INFO (
	이름 varchar2(10),
	생년 number(4),
	phone varchar2(15),
	첫방문일 date,
	고객번호 varchar2(10)
);
```

- 컬럼명 : 영문, 한글, 숫자 모두가능(==**시작만 문자로**==)
    - `h10` → 가능 , `10h` → 불가능
- 데이터 타입
    - `number` : 숫자형
    - `date` : 날짜형
    - `varchar2` : ==**가변길이 문자열**==
    - `char` : **==고정된 크기 문자열==**

```sql
CREATE TABLE C_INFO (
	이름 varchar2(10),
	생년 number(4) default 9999,
	phone varchar2(15) not null,
	첫방문일 date,
	고객번호 varchar2(10) primary key
);
```

- 제약조건(CONSTRAINT)
    - `default` : 기본값 지정
    - `not null` : null 입력 불가
    - `primary key` : 기본키 지정
        - PK 는 not null
        - PK 는 unique 한 값(==**테이블 내 중복 없음**==) → ==**고유한 값**==
    - `foreign key` : 외래키 지정
        - ==**테이블당 여러개 가능**==

### NULL 관련 문제 대비

1. NULL 은 모르는 값을 상징학고, ==**값이 없음(부재)을 의미함**==
2. **NULL IS NULL = TRUE**
3. ==**NULL 과의 모든 비교는 NULL을 반환함**==
4. ==**NULL 은 숫자 0이나 공백문자(' ')와 동일하지 않음**==

## ALTER : 테이블과 컬럼 이름 및 속성 변경, 추가/삭제 등 구조 수정

### 테이블명 변경 → RENAME TO

```sql
ALTER TABLE table_name RENAME TO table_hoho;

RENAME TABLE table_name TO table_hoho; -- ALTER를 생략할 수 있다.(다수테이블명 변경가능)
```

### 컬럼명 변경 → RENAME COLUMN

```sql
ALTER TABLE table_name RENAME COLUMN phone TO tel;
```

### 컬럼 속성 변경 → MODIFY

```sql
ALTER TABLE table_name MODIFY( 
	이름 varchar(20) not null)
;
```

### 컬럼 추가 → ADD

```sql
ALTER TABLE table_name ADD(
	거주지역 varchar(10)
);
```

### 컬럼 삭제 → DROP COLUMN

```sql
ALTER TABLE table_name DROP COLUMN 이름;
```

### 제약 조건 추가/삭제 → ADD/DROP

```sql
ALTER TABLE table_name ADD CONSTRAINT;

ALTER TABLE table_name DROP CONSTRAINT;

ALTER TABLE RIDING MODIFY(
	phone varchar(15) not null
);
```

## DROP : 테이블 및 컬럼 삭제

### 테이블 삭제

```sql
DROP TABLE table_name;
```

### 테이블 삭제 (유의점)

```sql
DROP TABLE table_name CASCADE CONSTRAINT;
-- 해당 테이블의 데이터를 외래키(FK)로 참조한 제약사항도 모두 삭제
-- Oracle 에만 있는 옵션, SQL Server에는 존재하지 않음
-- FK 제약조건과 참조테이블 먼저 삭제하고, 해당 테이블을 삭제한다.
```

### TRUNCATE : 테이블 초기화

```sql
TRUNCATE TABLE table_name;
-- 테이블 데이터만 삭제되고 구조는 살아있다.
```

## DROP vs TRUNCATE

- `DROP`
    - ==**테이블 정의를 완전 삭제함**==
    - 테이블이 사용했던 모든 저장공간을 Release 됨
- `TRUNCATE`
    - ==**테이블을 초기상태**==로 만듦
    - 테이블 최초 형성시 사용했던 저장공간만 남기고 Release

# DCL : Data Control Language

---

**데이터베이스 ==사용자에게 권한을 부여/회수하는 언어==**

## GRANT → 권한 부여

```sql
GRANT 권한
ON 테이블
TO 유저;
```

- 권한의 종류
    - SELECT
    - INSERT
    - UPDATE
    - DELETE
    - REFERENCES
    - ALTER
    - INDEX
    - ALL
- GRANT 옵션

    ```sql
    TO 유저
    WITH GRANT OPTION;
    -- 특정 사용자에게 권한 부여가능한 권한을 부여함
    -- 단, 부모가 회수될때 자식도 회수됨
    ```

    - B의 권한을 삭제하면 C도 같이 삭제

        ![[Untitled 53.png|Untitled 53.png]]


    ```sql
    TO 유저
    WITH ADMIN OPTION;
    -- 테이블에 대한 모든 권한 부여
    -- 부모의 권환 회수는 나랑 상관없음
    ```

    - B의 권한을 삭제해도 다른 권한과는 관계업슴

        ![[Untitled 1 39.png|Untitled 1 39.png]]


## REVOKE → 권환 회수

```sql
REVOKE 권한
ON 테이블
FROM 유저;
```

## 관련 문서

- 상위 목차: [[SQLD 공부 정리]]
- 이전 문서: [[Day10(정규화, 반정규화)]]
- 다음 문서: [[Day 2( DML , TCL )]]
