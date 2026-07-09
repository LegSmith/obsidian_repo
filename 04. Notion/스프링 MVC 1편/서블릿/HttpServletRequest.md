==_**서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP요청 메시지를 파싱한다!! 파싱한 결과를 HttpServletRequest 객체에 담아서 제공한다!!**_==

  

### 임시저장소 기능

- 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
- 저장 & 조회
    
    ```java
    request.setAttribute(name, value); //저장
    request.getAttribute(name); //조회
    ```
    

### 세션 관리 기능

```java
request.getSession(create:true);
```

  

==**HTTP 스펙이 제공하는 요청, 응답 메시지 자체를 이해해야 한다!**==