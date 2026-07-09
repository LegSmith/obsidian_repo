  

HTTP API 처럼 JSON 데이터를 HTTP메시지 바디에 넣어서 보내거나 읽어들일때 HTTP 메시지 컨버터를 사용하면 편리하다  
  
HttpMessageConverter 는 인터페이스이다. String, Json 등등의 타입으로 변환하기 떄문에 인터페이스로 구현되어 있다.

  

### 스프링 MVC에서 HTTP 메시지 컨버터 적용시

- HTTP 요청 : @RequestBody , HttpEntity(RequestEntity)
- HTTP 응답 : @ResponseBody , HTtpEntity(ResponseEntity)

  

  

### HTTP 메시지컨버터는 HTTP 요청,응답 둘 다 사용된다

- canRead(), canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크하는 메서드
- read(), write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

  

## 스프링 부트 기본 메시지 컨버터

- 대상 클래스 타입과 미디어 타입 둘 다 체크해서 사용여부를 결정한다.
    
    ````java
    ```
    // 숫자는 우선순위
    0 = ByteArrayHttpMessageConverter
    1 = StringHttpMessageConverter
    2 = MappingJackson2HttpMessageConverter
    ```
    ````
    

### 주요 메시지 컨버터

- ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다
    - 클래스타입 : btye[] , 미디어타입: * / *
    - 요청 예) @RequestBody byte[] data
    - 응답 예) @ResponseBody return byte[] , 쓰기 미디어타입 : application/octet-stream
- StringHttpMessageConverter : String 문자로 데이터를 처리한다
    - 클래스타입 : String , 미디어타입 : * / *
    - 요청 예) @RequestBody String data
    - 응답 예) @ResponseBody return “ok” 쓰기 미디어타입 : text/plain
- MappingJackson2HttpMessageConverter : application/json
    
    - 클래스타입 : 객체 또는 ‘HashMap’ , 미디어타입 : application/json 관련
    - 요청 예) @RequestBody HelloData data
    - 응답 예) @ResponseBody return helloData , 쓰기미디어타입 : application/json 관련