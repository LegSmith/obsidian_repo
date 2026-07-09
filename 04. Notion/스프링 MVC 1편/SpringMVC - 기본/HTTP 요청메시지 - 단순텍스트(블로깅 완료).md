## InputStream을 사용하여 데이터 읽어보기

1. HttpServletRequest, HttpServletResponse 를 이용하여 메세지 바디를 읽기

    ```java
    @PostMapping("/request-body-string-v1")
        public void requestBodyString(HttpServletRequest requset, HttpServletResponse response) throws IOException {
            ServletInputStream inputStream = requset.getInputStream();

            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

            log.info("messageBody={}",messageBody);

            response.getWriter().write("ok");
        }
    ```


스프링 MVC 는 다음 파라미터를 지원한다  
InputStream(Reader) : HTTP 요청 메세지 바디를 직접 조회  
OutputStream(Writer) : HTTP 응답 메세지의 바디를 직접 결과 출력

1. http message converter 를 이용한 메세지 바디 읽기

    ```java
    @PostMapping("/request-body-string-v3")
        public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {
            // 스프링은 HTTP Message Convertor 를 제공
            String messageBody = httpEntity.getBody();

            log.info("messageBody={}",messageBody);

            return new HttpEntity<>("ok");
        }
    ```


## HttpEntity : http header, body 정보를 편리하게 조회

- 메시지 바디 정보를 조회
- 요청파라미터(쿼리스트링, HTML Form 전송 등)를 조회하는 기능과 관계없다
- 응답에도 사용가능하다
    - 메서드 형식을 HttpEntity<type>으로 하고 반환시에 HttpEntity<>("응답내용"); 을 적어 OutputStream과 같은 역할로 사용 가능
- HttpEntity를 상속받는 RequestEntity<>, ResponseEntity<>사용 가능

    ```java
    @PostMapping("/http-message-converter")
        public HttpEntity<String> httpMessageConverter(RequestEntity<String> httpEntity) throws IOException{
            String messageBody = httpEntity.getBody();
            HttpMethod method = httpEntity.getMethod();
            URI url = httpEntity.getUrl();
            HttpHeaders headers = httpEntity.getHeaders();
            Type type = httpEntity.getType();
            log.info("method={}",method.toString());
            log.info("messageBody={}",messageBody);
            log.info("url={}",url.toString());
            log.info("headers={}",headers.toString());
            log.info("type={}",type.getTypeName());

            return new ResponseEntity<String>("ok", HttpStatus.CREATED);
        }
    ```


![[Untitled 81.png|Untitled 81.png]]


## HttpEntity를 대신할 @RequestBody를 이용

```java
@PostMapping("/request-body-string-v4")
    public HttpEntity<String> requestBodyStringV4(@RequestBody String messageBody) {

        log.info("messageBody={}",messageBody);

        return new HttpEntity<>("ok");
    }
```

## 관련 문서

- 상위 목차: [[SpringMVC - 기본]]
- 이전 문서: [[HTTP 요청메시지 - JSON(블로깅완료)]]
- 다음 문서: [[URL 매핑종류(블로깅 안해도댐)]]
