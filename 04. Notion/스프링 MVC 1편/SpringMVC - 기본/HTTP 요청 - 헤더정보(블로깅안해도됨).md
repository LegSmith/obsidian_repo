## 헤더 정보 log로 확인하기

- request : 요청정보를 가져올 수 있다.
- response : 응답정보를 가져올 수 있다.
- httpMethod : HTTP 통신할때 HTTP 메서드를 알 수 있다
- locale : locale 정보를 알 수 있다 → 어떤 언어 기반인지 알 수 있다.
- headerMap : MultiValueMap<> 을 이용하면 하나의 키로 여러 값을 알 수 있다.
- host : 'host' 키값을 이용하여 host의 정보를 알 수 있다.
- cookie : 쿠키내용을 알 수 있다.

    ```java
    @RequestMapping("/headers")
        public String headers(HttpServletRequest request,
                              HttpServletResponse response,
                              HttpMethod httpMethod,
                              Locale locale,
                              @RequestHeader MultiValueMap<String, String> headerMap,
                              @RequestHeader("host") String host,
                              @CookieValue(value = "myCookie", required = false) String cookie){

            log.info("request={}", request);
            log.info("response={}", response);
            log.info("httpMethod={}", httpMethod);
            log.info("locale={}", locale);
            log.info("headerMap={}", headerMap);
            log.info("header host={}", host);
            log.info("myCookie={}", cookie);

            return "ok";

        }
    ```

    ![[Untitled 80.png|Untitled 80.png]]


### MultiValueMap

- MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
- HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.

    - **keyA=value1&keyA=value2**

## 관련 문서

- 상위 목차: [[SpringMVC - 기본]]
- 이전 문서: [[HTTP 메시지 컨버터(블로깅완료)]]
- 다음 문서: [[HTTP 요청 파라미터 - @RequestParam(블로깅완료)]]
