# JOIN 심화

---

## EQUI JOIN

- 동일한 컬럼을 사용하여 두 릴레이션을 결함

## non-EQUI JOIN

- 정확하게 일치하지 않는 컬럼들을 사용하여 두 릴레이션을 결합
- = 을 사용하지 않음.

## CROSS JOIN

- **==key 없이==** JOIN 하면 2개의 테이블에 대해 ==**카테시안 곱(X) 발생**==
    
    ![[Untitled 56.png|Untitled 56.png]]
    
    ```sql
    SELECT * FROM T1 CROSS JOIN T2;
    -- 5 * 3 = 15 개의 행이 조회된다.
    ```
    

# SubQuery

---

### 메인쿼리 vs 서브쿼리

```sql
SELECT COUNT(*)
FROM 고객목록
WHERE 고객번호 NOT IN (SELECT 고객번호 FROM 연체자목록) -- where 절 안에 있는게 서브쿼리
```

### 이름있는 서브쿼리

- FROM 구에 SELECT 문이 있으면 → ==**인라인뷰(Inline View)**==
    
    ```sql
    FROM (SELECT * FROM 고객목록 WHERE 거주지='서울') A JOIN 연락처 B 
    						ON A.고객번호=B.고객번호 ;
    ```
    
- SELECT 문에 들어가고, ==**한 행과 한 컬럼만 반환**==하는 서브쿼리 → ==**스칼라 서브쿼리(Scalar Subquery)**==

### 단일행/다중행 서브쿼리

```sql
SELECT COUNT(*)
FROM 고객목록
WHERE 고객번호 NOT IN (SELECT 고객번호 FROM 연체자목록) -- 다중행 서브쿼리

SELECT (SELECT SUM(salary) FROM 급여 WHERE EXTRACT(YEAR FROM 급여지급일) = 2021)... 
-- 단일행 서브쿼리
```

# 계층형 조회

---

==**트리형태의 데이터에 대해 조회를 수행하는 것**==

![[Untitled 1 42.png|Untitled 1 42.png]]

```sql
SELECT col3
FROM 조직구조
START WITH col2 IS NULL
CONNECT BY PRIOR col1 = col2
ORDER SIBLINGS BY col3;
```

- `START WITH` : 계층 구조가 시작되는 지점(ROOT NODE)를 알려줌
- `CONNECT BY`, `PRIOR` : 다음 레코드가 어떤 것이 올지 알려줌
    - `CONNECT BY PRIOR a = b`
    - `CONNECT BY a = PRIOR b`
- `ORDER SIBLINGS BY` : 계층 구조는 그대로 유지하되, 동일 부모를 가진 자식들끼리 정렬 기준을 주는것

### 문제) 아래의 SQL을 수행한 결과 두 번째 행의 값을 기입하시오.

```sql
SELECT col3
FROM 조직구조
START WITH col1 IS NULL
CONNECT BY col1 = PRIOR col2;
```

![[Untitled 2 28.png|Untitled 2 28.png]]

|col1|col2|col3|
|---|---|---|
||13|15|
|13|12|13|
|12|11|12|
|11|10|10|

==**답 : 13/12/13**==