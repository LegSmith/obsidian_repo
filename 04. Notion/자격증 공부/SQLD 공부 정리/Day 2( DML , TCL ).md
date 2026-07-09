# DML : Data Manipulation Language

---

## INSERT : 데이터 입력

```sql
INSERT INTO MENU(NAME) VALUES('연어스시');
```

## UPDATE : 데이터 수정

```sql
UPDATE MENU SET discount_rate = 10 WHERE name = '연어스시';
```

## DELETE : 데이터 삭제

- `TRUNCATE` 와 `DROP` 과 다르게 `DELETE` 는 로그를 남긴다.
- 삭제된 데이터를 되돌릴 수 있다.(DB에 반영되기 전까지는)
- `TRUNCATE` , `DROP`은 되돌릴 수 없다.

    ```sql
    DELETE FROM MENU WHERE name = '연어스시';

    DELETE MENU; -- FROM 생략가능
    ```


# TCL : Transaction Control Language

---

`Transaction` **: DB의 상태를 변화시키기 위해 수행하는 ==작업의 단위==**

- `COMMIT` : ==**데이터에 대한 변화를 DB에 반영**==하기 위한 명령어
- `SAVEPOINT` : ==**코드를 분할하기 위한 저장 포인트 지정**==
- `ROLLBACK`

    - ==**트랜잭션이 시작되기 이전의 상태로 되돌리기 위한 언어**==
    - 최신 COMMIT 이나 특수한 SAVEPOINT로 되돌릴 수 있는 명령어


> ==**DML은 자동 commit(반영) 되지 않음.**==  
> ROLLBAK, DROP은 되돌릴 수 없다 → ==**SQL server기준 DDL 은 auto-commit**==


### COMMIT과 ROLLBACK 효과

1. ==**데이터 무결성**==을 보장할 수 있다.
2. 영구적인 변경 전 데이터에 대한 변동사항을 확인할 수 있다.
3. 논리적 연관성 있는 작업을 그룹화하여 처리할 수 있다.


## ==트랜잭션 ACID==

- **Atomicity** → **==원자성==**
    - 트랜잭션에서 정의된 연산은 모두 성공적으로 실행되던지 아니면 전혀 실행되지 않은 상태로 있어야 한다.
- **Consistency** → ==**일관성**==
    - 트랜잭션 **발생 전** 데이터베이스 내용에 잘못된 점이 없다면 트랜잭션 **수행 후**에도 데이터베이스에 내용이 잘못이 있으면 안된다.
- **Isolation** → ==**독립성**==
    - 트랜잭션이 **실행되는 동안** 다른 트랜잭션에 영향을 받아 잘못된 결과를 만들어선 안된다.
- **Durability** → ==**지속성**==
    - 트랜잭션이 성공적으로 완료되면 해당 트랜잭션이 갱신한 데이터베이스의 내용은 **영구적으로 저장된다**

## 관련 문서

- 상위 목차: [[SQLD 공부 정리]]
- 이전 문서: [[Day 1( DDL~DCL)]]
- 다음 문서: [[Day 3(SELECT , 함수)]]
