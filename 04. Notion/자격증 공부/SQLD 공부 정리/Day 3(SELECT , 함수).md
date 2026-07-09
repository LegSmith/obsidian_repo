# **SELECT**

---

**SELECT 는 데이터를 조회할 때 쓰는 명령어이다.**

## SELECT 문 기본구조

```sql
SELECT 컬럼명 등
FROM 테이블명
WHERE 조건문
GROUP BY 집계기준 컬럼명
HAVING grouping된 후 상태 기반의 조건문
ORDER BY 컬럼명
```

- DISTINCT 명령어는 중복을 제거해주는 명령어이다.

    - ==**DINTINCT 는 NULL도 단일 항목으로 본다.**==

    ![[Untitled 54.png|Untitled 54.png]]


## SELECT 문 기본 구조 - 문자형 함수

**주로 SELECT 과 WHERE에 자주 쓰인다**

|   |   |
|---|---|
|`LOWER(문자열)`|영어 문자열 ==**소문자로 변환**==  <br>`LOWER('SQL')` → 'sql'|
|`UPPER(문자열)`|영어 문자열 ==**대문자로 변환**==  <br>`LOWER('sql')` → 'SQL'|
|`CONCAT(문자열1, 문자열2)`|문자열 1과 문자열2을 ==**결합**==  <br>`CONCAT('가','나')` → '가나'|
|`SUBSTR(문자열, m, n)`|문자열에서 ==**m 번째 자리값부터 n개를 자른다**==  <br>`SUBSTR('KATE',2,2)` → 'AT'|
|`LENGTH(문자열) = LEN(문자열)`|공백을 포함하여 ==**문자열의 길이값**==  <br>`LEN('가 나다')` → 4|
|`TRIM(문자열, 제거대상)`|`TRIM('aabbccaa', 'a')` → 'bbcc'  <br>`TRIM(' aabbccaa. ' ,)` → 'aabbccaa' : 지정된 문자가 없으면 공백을 제거|
|`LTRIM(문자열, 제거대상)`|`LTRIM('aabbccaa', 'a')` → 'bbccaa' : 왼쪽에서 지정된 문자 삭제|
|`RTRIM(문자열, 제거대상)`|`RTRIM('aabbccaa', 'a')` → 'aabbcc' : 오른쪽에서 지정된 문자 삭제|

```sql
SELECT 회원코드, 연령대, UPPER(이름)
FROM C_INFO

SELECT 회원코드, TRIM(연령대,'대'), UPPER(이름)
FROM C_INFO
```

## SELECT 문 기본 구조 - 숫자형 함수

|   |   |
|---|---|
|`ROUND(숫자, 소수점 자릿수)`|반올림  <br>`ROUND(25.3578, 2)` → 25.36|
|`TRUNC(숫자, 소수점 자릿수)`|버림  <br>`TRUNC(25.3578, 2)` → 25.35|
|`CEIL(숫자)`|크거나 같은 최소 정수 반환  <br>`CEIL(33.5)` → 34|
|`FLOOR(숫자)`|작거나 같은 최대 정수 반환  <br>`FLOOR(33.5)` → 33|
|`MOD(분자, 분모)`|분자를 분모로 나눈 나머지 반환  <br>`MOD(3, 2)` → 1|
|`SIGN(숫자)`|==**숫자가 양수면 1, 0이면 0, 음수면 -1 반환**==|
|`ABS(숫자)`|절댓값|
|`SYSDATE`|쿼리를 돌리는 현재 날짜&시각 출력  <br>ex) 2022/01/31 14:00:26 (datetime 형태)|
|`````` ````` ```` ``` `` `EXTRACT(` `` ``` ```` ````` ```````정보` `FROM 날짜)`  <br>정보: YEAR, MONTH, DAY, HOUR, MINUTE, SECOND|날짜형 데이터에서 원하는 값을 추출함  <br>`EXTRACT(` `YEAR` `FROM date '2022-01-31')` → 2022  <br>`EXTRACT(YEAR FROM sysdate)`==`TO_NUMBER(TO_CHAR(sysdate,'YYYY'))`|

## SELECT 문 기본 구조 - 명시적/암시적 형변환

|   |   |
|---|---|
|`TO_NUMBER(문자열)`|문자열을 ==**숫자로 변환**==  <br>`TO_NUMBER('2022')` → 2022|
|`TO_CHAR(숫자 or 날짜, 포맷)`|숫자 혹은 날짜형 데이터를 ==**포맷에 맞게 문자로 바꿈**==  <br>`TO_CHAR(date '2022-02-11', 'day')` → 금요일  <br>`TO_CHAR(200)` → 200|
|`TO_DATE(문자열, 포맷)`|`TO_DATE('2022013120' , 'YYYYMMDDHH24')`  <br>→ 2022/01/31 20:00:00|

- `SYSDATE` 는 SQL을 작업하는 당일의 날짜와 시각을 알려주며, ==**-1 을 할 경우 전날의 날짜값이 출력된다.**==

## SELECT 문 기본 구조 - DECODE , CASE WHEN

|   |   |
|---|---|
|DECODE|= ==**IF문**==  <br>`DECODE(값1, 값2, 참일때 출력값, 거짓일때 출력값)`  <br>ex) `DECODE(col1, KATE, '본인', '다른사람')`|
|CASE WHEN|= if-else문  <br>CASE WHEN 조건 THEN 조건이 참일때 결과 ELSE 거짓일때 결과 END  <br>  <br>CASE WHEN 조건문1 THEN 결과값1  <br>WHEN 조건문 2 THEN 결과값2  <br>…..  <br>WHEN 조건문 n THEN 결과값 n  <br>ELSE 결과값 n+1  <br>END  <br>  <br>ex)  <br>CASE WHEN col1 < 10 THEN '한 자리수'  <br>WHEN col1 BETWEEN 10 AND 99 THEN '두 자릿수'  <br>ELSE '세 자리수'  <br>END|

### 문제

- 2개의 정렬 기준이 있다면 첫번째로 오는 값을 먼저 정렬후 중복되는 값을 두번째 정렬 조건값에 따라한다.

![[Untitled 1 40.png|Untitled 1 40.png]]

- 첫번째 case when 부분을 정렬한 결과 id가 101과 104는 1로 정렬돼있고, 102,103,105는 2로 정렬되있어서 아래와같이 표가 나온다

    |CF_ID|AGE|NAME|
    |---|---|---|
    |101|20|kate|
    |104|24|paul|
    |102|40|max|
    |103|59|lily|
    |105|11|james|

- 두 번째 정렬기준인 연령 DESC는 연령을 내림차순으로 정렬하란 명령어이다.
- 현재 101, 104가 1로 묶여있기 때문에 101과 104를 연령별로 내림차순하고 102,103,105도 2로 묶여있기 때문에 연령별로 내림차순하면 아래와 같은 결과가 나온다.

    |CF_ID|AGE|NAME|
    |---|---|---|
    |104|24|paul|
    |101|20|kate|
    |103|59|lily|
    |102|40|max|
    |105|11|james|

## 관련 문서

- 상위 목차: [[SQLD 공부 정리]]
- 이전 문서: [[Day 2( DML , TCL )]]
- 다음 문서: [[Day 4(WHERE절 연산자,VIEW, Group by)]]
