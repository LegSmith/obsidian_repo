# Group 함수들

---

## ROLLUP

- ==**부분합계와 전체합계값**==을 보여준다
- 인수의 순서에 영향을 받는다.
- ROLLUP(성별, 연령대) 명령어는 성별과 연령대를 그룹화한 다음 부분집계와 전체합계합을 구하는 명령어다.

    ```sql
    SELECT 성별, 연령, SUM(결제금액)
    FROM 결제
    GROUP BY ROLLUP(성별, 연령대)
    ORDER BY 성별, 연령
    ```

- 노란색 행이 성별이 F와 M 의 결제금액의 합이고, 빨간색 행이 전체합계의 합이다.

    1. (성별, 연령) 별 합계
    2. 성별 별 합계
    3. 전체 합계

    |성별|연령대|SUM(결제금액)|
    |---|---|---|
    |F|10대|1000|
    |F|20대|3000|
    |F||4000|
    |M|30대|1500|
    |M|40대|5500|
    |M||7000|
    |||11000|


## CUBE

- ROLLUP과 다르게 CUBE는 ==**그룹화될 수 있는 모든 경우에 대해 생성**==

    ```sql
    SELECT 성별, 연령, SUM(결제금액)
    FROM 결제
    GROUP BY CUBE(성별, 연령대);
    ```

- 노란색은 성별 별 합계,파랑색은 연령 별 합계,빨간색은 전체합계

    1. (성별, 연령) 별 합계
    2. 성별 별 합계
    3. 연령 별 합계
    4. 전체 합계

    |성별|연령대|SUM(결제금액)|
    |---|---|---|
    |F|10대|1000|
    |F|20대|3000|
    |F||4000|
    |M|30대|1500|
    |M|40대|5500|
    |M||7000|
    |||11000|
    ||10대|1000|
    ||20대|3000|
    ||30대|1500|
    ||40대|55000|


## GROUPING SETS

- 괄호 묶은 집합별 집계가 가능하다.

    ```sql
    SELECT 성별, 연령, SUM(결제금액)
    FROM 결제
    GROUP BY GROUPING SETS(성별, 연령대);
    ```

- 성별 별 합계와 연령 별 합계를 구할 수 있다.

    |성별|연령대|SUM(결제금액)|
    |---|---|---|
    |F||4000|
    |M||7000|
    ||10대|1000|
    ||20대|3000|
    ||30대|1500|
    ||40대|5500|


## GROUPING

- 소계, 합계 등이 계산되면 1을 반환하고, 아니면 0을 반환한다.

    ```sql
    SELECT 성별, GROUPING(성별) g1, 연령, GROUPING(연령) g2, SUM(결제금액)
    FROM 결제
    GROUP BY ROLLUP(  성별, 연령대)
    ORDER BY 성별, 연령;
    ```

- 3번째행에서 g2가 1이 나오는 이유는 성별이 F이고 연령대가 10대,20대의 결제금액의 합이기 때문에 합계가 계산되서 1이 반환됐다.
- 마지막행에서 g1 컬럼이 1이 반환된 이유는 성별 F, M의 전체합계가 계산이 됐기 때문이다.

    |성별|g1|연령대|g2|SUM(결제금액)|
    |---|---|---|---|---|
    |F|0|10대|0|1000|
    |F|0|20대|0|3000|
    |F|0||1|4000|
    |M|0|30대|0|1500|
    |M|0|40대|0|5500|
    |M|0||1|7000|
    ||1||1|11000|


# JOIN

---

==**JOIN : 테이블 간의 결합, 집합과 유사함**==

- **교집합**

    `INNER JOIN`

    `LEFT JOIN`

    `RIGHT JOIN`

    `OUTER JOIN`

- **합집합**

    `UNION(ALL)`

- **차집합**

    `MINUS`(orcale) = `EXCEPT`(SQL Server)

- **결합되는 대상간의 일치 정도**

    `EQUI JOIN` <> `non-EQUI JOIN`

- **조건구 없는** `CROSS JOIN`

## INNER JOIN

- INNER JOIN은 교집합이 있는 컬럼만 보여준다.

    ![[Untitled 55.png|Untitled 55.png]]

    ```sql
    -- ANSI 표준쿼리
    SELECT A.*, B.연령
    FROM GENDER A INNER JOIN AGE B 
    			ON A.회원코드 = B.회원코드;
    ```

- 3개 이상의 테이블을 JOIN 할 때는 JOIN을 두번 기입하면된다.

    ![[Untitled 1 41.png|Untitled 1 41.png]]

    ```sql
    -- ANSI 표준쿼리
    SELECT A.*, B.연령, C.생년
    FROM GENDER A JOIN AGE B ON A.회원코드=B.회원코드
    							JOIN BIRTH C ON B.연령=C.연령;
    ```


## LEFT JOIN(LEFT OUTER JOIN)

- JOIN 할 때 왼쪽 테이블을 기준으로 ==**왼쪽테이블의 교집합 컬럼이 오른쪽테이블에 없어도 전부 출력한다.**==

    ![[Untitled 2 27.png|Untitled 2 27.png]]

    ```sql
    SELECT A.*,B.연령
    FROM GENDER A LEFT JOIN AGE B ON A.회원코드=B.회원코드;

    SELECT A.*,B.연령
    FROM GENDER A, AGE B
    WHERE A.회원코드=B.회원코드(+); --NULL을 허용하는 곳에 (+)를 적어주면된다.
    ```


## OUTER JOIN

- FULL OUTER JOIN은 그냥 모두 출력한다.

    ![[Untitled 3 18.png|Untitled 3 18.png]]

    ```sql
    -- ANSI 표준쿼리
    SELECT A.*, B.연령
    FROM GENDER A FULL OUTER JOIN AGE B ON A.회원코드=B.회원코드;

    SELECT A.*, B.연령
    FROM GENDER A, AGE B
    WHERE A.회원코드(+) = B.회원코드(+);
    -- Oracle 9 버전 이후부터는 양쪽의 (+) 가능 
    ```


## UNION

- 동일한 컬럼 개수와 데이터 타입을 가진 두 테이블을 합쳐줌
- 세로로 합쳐지는것
- ==**중복된 레코드가 제거된다.**==

    ![[Untitled 4 16.png|Untitled 4 16.png]]

    ```sql
    SELECT * FROM T1
    UNION
    SELECT * FROM T2;
    ```

    |회원코드|성별|
    |---|---|
    |101|F|
    |102|M|
    |103|F|
    |104|M|
    |105|F|
    |107|F|
    |108|M|


## UNION ALL

- UNION ALL은 UNION과 정반대이다.

## MINUS( SQL Server에서는 EXCEPT를 사용)

- 차집합과 같다.

## 관련 문서

- 상위 목차: [[SQLD 공부 정리]]
- 다음 문서: [[Day6(JOIN심화,서브쿼리,계층형조회)]]
