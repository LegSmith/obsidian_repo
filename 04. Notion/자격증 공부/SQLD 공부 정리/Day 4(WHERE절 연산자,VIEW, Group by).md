# SELECT 문 기본 구조 - WHERE 조건문

---

## 연산자 종류

|   |   |
|---|---|
|`IN(x,y,z,….)`|x,y,z 등으로 구성된 ==**목록 내 값 중 어느 하나라도 일치하면 된다.**==|
|`NOT IN(x,y,z,….)`|x,y,z 등으로 구성된 **==목록 내 값 중 어느 하나라도 일치하면 안된다.==**|
|`IS NULL`|NULL 인지 판단. ==**NULL 일 경우 TRUE**==|
|`IN NOT NULL`|NULL이 아닌지 판단. ==**NULL이 아닐 경우 TRUE**==|
|`BETWEEN a AND b`|a 와 b 사이에 값이 있는지|
|기타|비교연산자(=, > , > = , < , < = ) 등|

## 문자열 조건문 관련 연산자

|   |   |
|---|---|
|`A LIKE B`|A 에 대하여 B와 유사한 문자열을 찾아줌|
|`%`|문자 1개 이상이 존재한다는 의미|
|`__`|문자 한 개|

### WITH 구문

1. 서브쿼리를 사용해서 ==**임시테이블이나 뷰처럼 사용 가능함**==
2. **별칭 지정 가능함**
3. **인라인뷰**나 **임시테이블**로 판단한다.

    ```sql
    WITH table__name AS(
    	SELECT *
    	FROM C_INFO
    	WHERE NAME LIKE '%a%'
    );

    INSERT INTO SQLD_34_23 VALUES (1, 'A' );
    INSERT INTO SQLD_34_23 VALUES (2, 'A' );
    INSERT INTO SQLD_34_23 VALUES (1, 'C' );
    INSERT INTO SQLD_34_23 VALUES (1, 'D' );
    INSERT INTO SQLD_34_23 VALUES (2, 'D' );
    COMMIT;
    ```


### 서브쿼리와 인라인뷰

- **서브쿼리** : ==**SELECT 문 내에 SELEC문**==이 또 쓰여있는 쿼리
- **인라인뷰** : ==**서브쿼리가 FROM 절 내**==에 쓰여진 것.

    `SELECT * FROM (SELECT * FROM C_INFO WHERE name LIKE '%a%');`


### VIEW 테이블

- 일종의 ==**가상테이블**==로서 실제 데이터가 하드웨어에 저장되는 것이 아니다.
- 실제 데이터를 가지고 있지 않다.
- ==**테이블 구조가 변경되더라도 독립적으로 존재**==한다.

# SELECT 문 기본 구조 - NULL 관련 함수

---

|   |   |
|---|---|
|`NVL(col1, 대체값)`|NULL이면 다른 값으로 바꿔주는 함수  <br>`NVL(col1, 100)` → col1 이 NULL 이면, 100으로 바꿔줌|
|`NVL2(col1, 결과1, 결과2)`|col1 이 NULL일 때,  <br>`NVL(col1, 'F', 'T')` → 'T' 출력  <br>col1 이 NOT NULL 일때 'F'출력  <br>즉, ==**col1이 NULL 이면 결과2, NOT NULL 이면 결과 1 출력**==|
|`NULLIF(v1, v2)`|v1 == v2 이면 NULL==**v1 ! = v2 이면 v1 출력**==|
|`COALESCE(v1, v2, v3, …vn)`|==**NULL이 아닌 최초의 값을 반환**==|

# DML - SELECT 문 기본 구조 - 집계 함수

---

|   |   |
|---|---|
|`COUNT(*), COUNT(exp)`|`COUNT(*`) : NULL 포함, `COUNT(exp)` : NULL 제외|
|`SUM([DISTINCT \| ALL] exp)`|합계|
|`AVG([DISTINCT \| ALL] exp)`|평균|
|`MAX([DISTINCT \| ALL] exp)`|최대값|
|`MIN([DISTINCT \| ALL] exp)`|최소값|
|`STDDEV([DISTINCT \| ALL] exp)`|표준편차|
|`VARIAN([DISTINCT \| ALL] exp)`|분산|

```sql
SELECT 성별, AVG(연령) as 평균연령
FROM C_INFO
GROUP BY 성별 HAVING AVG(연령) BETWEEN 30 AND 40;
---
SELECT AS -> FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```

## SELECT 문 기본 구조 - 조회되는 행 수를 제한할 때

---

**만약, 테이블의 한 행만 조회하고 싶다면 ???**

### ORACLE 기준

```sql
SELECT *
FROM C_INFO
WHERE ROWNUM = 1;
```

- `ROWNUM` : 오라클에서 ==**조회된 행이 몇번째 행인지 부여해주는 것**==
- **SQL Server**

    `SELECT TOP(1) FROM C_INFO;`

- **MySQL**

    `SELECT * FROM C_INFO LIMIT 1;`

## 관련 문서

- 상위 목차: [[SQLD 공부 정리]]
- 이전 문서: [[Day 3(SELECT , 함수)]]
- 다음 문서: [[Day 9(1과목 - 데이터모델링)]]
