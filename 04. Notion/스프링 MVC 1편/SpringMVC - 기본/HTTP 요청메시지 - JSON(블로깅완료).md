  

JSON 객체를 자바 Object로 변환하기 위해서 클래스 상단에 코드한줄을 적어주자  
`private ObjectMapper objectMapper = new ObjectMapper();`

  

## HttpServletRequest,Response를 이용

- servlet를 이용한 방법 코드가 상당히 더럽고 길다
    
    ```java
    @PostMapping("/request-body-json-v1")
        public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
            ServletInputStream inputStream = request.getInputStream();
    
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    
            log.info("messageBody={}",messageBody);
    
            HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
            log.info("username={}, age={}",helloData.getUsername(),helloData.getAge());
    
            response.getWriter().write("ok");
    
        }
    ```
    

  

## @ResponseBody, @RequestBody를 이용한 방법

- @ResponseBody를 이용하기 때문에 데이터를 view에 응답할 수 있고
- @RequestBody를 이용하기 때문에 메세지바디 데이터를 가져와서 사용이 가능하다
    
    ```java
    @ResponseBody
        @PostMapping("/request-body-json-v2")
        public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
    
    
            log.info("messageBody={}",messageBody);
    
            HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
            log.info("username={}, age={}",helloData.getUsername(),helloData.getAge());
    
            return "ok";
    
        }
    ```
    
- @RequestBody 객체 파라미터 방법
- Http메세지 컨버터를 이용해서 HttpEntity<객체> 로 사용가능
- objectMapper를 사용하지 않아도 자동으로 json을 객체로 변환해 준다.
    
    ```java
    @ResponseBody
        @PostMapping("/request-body-json-v3")
        public String requestBodyJsonV3(@RequestBody HelloData helloData) {
    
            log.info("username={}, age={}",helloData.getUsername(),helloData.getAge());
    
            return "ok";
    
        } 
    ```
    

==_**@RequestBody를 생략하면 자동으로 @ModelAttribute가 붙기 때문에 생략하면 안된다!!**_==

Spring은 @ModelAttrbute , @RequestParam 생략 시 다음과 같이 자동으로 적용한다  
String, int, Integer 과 같은 단순타입은 @RequestParam 자동적용  
그 외 @ModelAttribute 자동 적용

  

## 객체 파라미터를 직접 반환했을때

```java
@ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(HttpEntity<HelloData> helloData) {

        HelloData body = helloData.getBody();
        log.info("username={}, age={}",body.getUsername(),body.getAge());

        return body;
    }
```

![[Untitled 82.png|Untitled 82.png]]