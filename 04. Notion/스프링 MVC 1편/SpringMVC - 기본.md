[[로깅 레벨(포스팅 완료)]]

[[요청 매핑(포스팅 완료)]]

[[URL 매핑종류(블로깅 안해도댐)]]

[[HTTP 요청 - 헤더정보(블로깅안해도됨)]]

[[HTTP 요청 파라미터 - @RequestParam(블로깅완료)]]

[[HTTP 요청메시지 - 단순텍스트(블로깅 완료)]]

[[HTTP 요청메시지 - JSON(블로깅완료)]]

[[응답 - 정적리소스, 뷰 템플릿(블로깅 완료)]]

[[HTTP 메시지 컨버터(블로깅완료)]]

---

==**JSP 와 별도의 Tomcat을 사용할때는 War 를 쓰는게 좋다 !**==

## consumes 과 produces 차이

---

- Spring 프레임워크에서 Mapping을 할 때 받고싶은 데이터에 조건을 걸어줄 수 있다.
- 예를 들어 HTTP 헤더에 ==**Media Types**==을 ==**jsno**==인지 ==**text**==인지 정하는것과 같다.
- 이때 들어오는 데이터와 나가는 데이터를 정할수 있다.
- 들어오는 데이터 타입을 정할때 consumes을 사용한다.

### consumes

- 예를 들어 PostMapping 으로 json타입을 받는다하면 아래와 같이 코딩하면 된다.

    ```java
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }
    ```

- Media Types을 표현하는 방법은 여러가지 이다.

    `consumes = "text/plain" consumes = {"text/plain", "application/*"} consumes = MediaType.TEXT_PLAIN_VALUE`

- 만약 Media Types이 맞지 않으면 HTTP 415 상태코드를 반환한다.

## Json 으로 HTTP 요청 메세지를 보낼때

---

> json 타입으로 `{"username":"name", "age":20}` 을 통신한다고 가정하자

- 파라미터앞에 어노테이션을 생략한다면 데이터가 null 로 들어온다.
- ==**왜냐하면 Spring에서는 int, String 등과 같은 단순 타입은 @RequestParam으로 인식하고, 그 외에 사용자가 만든 객체같은 것들은 @ModelAttribute로 인식하기 때문이다.**==

    ```java
    // HelloData 클래스는 username과 age를 변수로 가지고있는 클래스이다.
    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {

        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

        return "ok";
    }
    ```


## HTTP 메시지 컨버터

---

스프링 MVC는 2가지 경우에 HTTP 메시지 컨버터를 적용한다  
1. HTTP 요청 : `@RequestBdoy` , `HttpEntity(RequestEntity)`  
2. HTTP 응답 : `@ResponseBody` , `HttpEntity(ResponseEntity)`

## 관련 문서

- 상위 목차: [[스프링 MVC 1편]]

## 하위 문서

- [[HTTP 메시지 컨버터(블로깅완료)]]
- [[HTTP 요청 - 헤더정보(블로깅안해도됨)]]
- [[HTTP 요청 파라미터 - @RequestParam(블로깅완료)]]
- [[HTTP 요청메시지 - JSON(블로깅완료)]]
- [[HTTP 요청메시지 - 단순텍스트(블로깅 완료)]]
- [[URL 매핑종류(블로깅 안해도댐)]]
- [[로깅 레벨(포스팅 완료)]]
- [[요청 매핑(포스팅 완료)]]
- [[응답 - 정적리소스, 뷰 템플릿(블로깅 완료)]]
