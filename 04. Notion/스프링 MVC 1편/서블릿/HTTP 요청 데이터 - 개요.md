[[HTTP 요청 데이터 - GET 쿼리 파라미터]]

[[HTTP 요청 데이터 - POST HTML Form]]

[[HTTP 요청 데이터 - API 메시지 바디]]


HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법은 아래 3가지와 같다.

## GET- 쿼리 파라미터

- http://www.naver.com==**?username=hello&age=20**==
- 메시지 바디 없이, URL 의 데이터를 포함해서 전달
- 예시) 검색, 필터, 페이징 등등

## POST - HTML Form

- content-type : application/x-www-form-urlencoded
- 메시지 바디에 쿼리 파라미터 형식으로 전달 ==**username=hello&age=20 (application/x-www-form-urlencoded 방식이다)**==

![[Untitled 75.png|Untitled 75.png]]

## HTTP message body 에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용, JSON, XML, TEXT
- 요새는 JSON형식을 사용
- POST, PUT , PATCH

## 관련 문서

- 상위 목차: [[서블릿]]

## 하위 문서

- [[HTTP 요청 데이터 - API 메시지 바디]]
- [[HTTP 요청 데이터 - GET 쿼리 파라미터]]
- [[HTTP 요청 데이터 - POST HTML Form]]
